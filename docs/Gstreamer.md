# Gstreamer
Gstreamer - pipeline-based multimedia framework

## Installation
```
sudo apt-get install gstreamer1.0-tools \
  gstreamer1.0-plugins-base \
  gstreamer1.0-plugins-good \
  gstreamer1.0-plugins-bad \
  gstreamer1.0-plugins-ugly \
  gstreamer1.0-gtk3 \
  gstreamer1.0-doc \
  gstreamer1.0-tools

sudo apt-get install libcairo2-dev libjpeg-dev libgif-dev libgirepository1.0-dev gir1.2-gtk-3.0

pip3 install pycairo
pip3 install GObject
pip3 install pygobject
```

## Check video device

You can use v4l2-ctl application in order to control the linux4video devices.

Example for checking camera resolution:
``` 
sudo apt-get install v4l-utils
v4l2-ctl -d /dev/video0 --list-formats-ext
```

## Pipes for example

### stream MJPG camera with compositor
```
gst-launch-1.0 compositor name=comp \
sink_0::xpos=0    sink_0::ypos=0   sink_0::width=320   sink_0::height=480 \
sink_1::xpos=320    sink_1::ypos=0   sink_1::width=320   sink_1::height=480 ! autovideosink \
v4l2src device="/dev/video0" name=cam0 ! capsfilter name=capssrc0 \
caps=image/jpeg,width=640,height=480,framerate=30/1 ! jpegdec ! videoconvert ! \
video/x-raw,width=640,height=480 ! tee name=video_tee !  queue ! comp. \
video_tee. ! videoscale ! video/x-raw ! comp.
```

## Dynamic compositor streaming

Here is an example python code for adding video camera to a compositor and removing it.

```
import sys
import gi
gi.require_version('Gst', '1.0')
from gi.repository import Gst, GLib

class ProbeTest:

    def __init__(self):
        # Initialize gstreamer
        Gst.init(sys.argv)

        # Define pipeline
        gst_str = """
        compositor name=comp 
        sink_0::xpos=0    sink_0::ypos=0   sink_0::width=320   sink_0::height=480
        sink_1::xpos=320    sink_1::ypos=0   sink_1::width=320   sink_1::height=480 ! videorate !
        video/x-raw,framerate=30/1 ! autovideosink 
        videotestsrc pattern=ball is-live=True ! video/x-raw,framerate=30/1 !  queue ! comp. 
        videotestsrc pattern=snow is-live=True ! video/x-raw,framerate=30/1 ! comp. 
        """
        
        # Create the pipeline
        self.loop = GLib.MainLoop()
        self.pipeline = Gst.parse_launch(gst_str)

        self.bus = self.pipeline.get_bus()
        self.bus.add_signal_watch()
        self.bus.connect("message", self._on_message)

        # prepare source objects
        self.new_source = Gst.ElementFactory.make("v4l2src", "video0")
        self.new_source.set_property('device', '/dev/video0')
        self.new_sourcepad = self.new_source.get_static_pad("src")
        self.new_source_probe = self.new_sourcepad.add_probe(Gst.PadProbeType.BLOCK_DOWNSTREAM, self.modify_pipeline)

        # add new source to the pipeline
        self.pipeline.add(self.new_source)
        print("added new source to pipeline")

        # prepare source caps objects
        self.new_capssrc = Gst.ElementFactory.make("capsfilter", "new_capssrc")
        self.new_capssrc.set_property('caps', Gst.Caps.from_string("image/jpeg,width=640,height=480,framerate=30/1"))
        self.new_jpegdec = Gst.ElementFactory.make("jpegdec", "new_jpegdec")
        self.new_videoconvert = Gst.ElementFactory.make("videoconvert", "new_videoconvert")
        self.new_capssrc2 = Gst.ElementFactory.make("capsfilter", "new_capssrc2")
        self.new_capssrc2.set_property('caps', Gst.Caps.from_string("video/x-raw,width=640,height=480"))        
        self.new_queue = Gst.ElementFactory.make("queue", "new_queue")
        self.new_queue_pad = self.new_queue.get_static_pad("src")

        self.comp = self.pipeline.get_by_name('comp') # get compositor element from pipeline

        # Start the pipeline
        self.pipeline.set_state(Gst.State.READY)
        self.pipeline.set_state(Gst.State.PLAYING)

    def change_source(self, line):
        if line=="1\n":
            self.new_comp_sink_pad = self.comp.get_request_pad("sink_2") # create new compositor pad
            print(self.new_comp_sink_pad.name)

            # add new source elements to the pipeline
            self.pipeline.add(self.new_capssrc)
            self.pipeline.add(self.new_jpegdec)
            self.pipeline.add(self.new_videoconvert)
            self.pipeline.add(self.new_capssrc2)
            self.pipeline.add(self.new_queue)

            # set the source caps to ready and playing
            self.new_capssrc.set_state(Gst.State.READY)
            self.new_jpegdec.set_state(Gst.State.READY)
            self.new_videoconvert.set_state(Gst.State.READY)
            self.new_capssrc2.set_state(Gst.State.READY)
            self.new_queue.set_state(Gst.State.READY)
            self.new_capssrc.set_state(Gst.State.PLAYING)
            self.new_jpegdec.set_state(Gst.State.PLAYING)
            self.new_videoconvert.set_state(Gst.State.PLAYING)
            self.new_capssrc2.set_state(Gst.State.PLAYING)
            self.new_queue.set_state(Gst.State.PLAYING)

            # change the old compositor sink pads according to new properties
            self.comp_sink_pad0 = self.comp.get_static_pad("sink_0")
            self.comp_sink_pad0.set_property("height",240)
            self.comp_sink_pad1 = self.comp.get_static_pad("sink_1")
            self.comp_sink_pad1.set_property("height",240)

            # set the properties for the new compositor pad
            self.new_comp_sink_pad.set_property("xpos",0)
            self.new_comp_sink_pad.set_property("ypos",240)
            self.new_comp_sink_pad.set_property("width",640)
            self.new_comp_sink_pad.set_property("height",240)


            # link the new source : source--> caps(mjpg)--> jpegdec--> videoconvert--> caps(x/raw)--> queue
            self.new_source.link(self.new_capssrc)
            self.new_capssrc.link(self.new_jpegdec)
            self.new_jpegdec.link(self.new_videoconvert)
            self.new_videoconvert.link(self.new_capssrc2)
            self.new_capssrc2.link(self.new_queue)

            self.new_queue_pad.link(self.new_comp_sink_pad)
            self.new_sourcepad.remove_probe(self.new_source_probe)

        if line=="2\n":
            if self.new_queue_pad.is_linked(): # check if the source pad is already exist. if not, there is nothing to remove
                self.new_source_probe = self.new_sourcepad.add_probe(Gst.PadProbeType.BLOCK_DOWNSTREAM, self.modify_pipeline)
                print("added source probe")

                self.new_comp_sink_pad = self.new_queue_pad.get_peer()
                self.new_queue_pad.unlink(self.new_comp_sink_pad) # unlink the new source pad from the compositor pad

                # stop the caps elements of the source
                self.new_capssrc.set_state(Gst.State.NULL)
                self.new_jpegdec.set_state(Gst.State.NULL)
                self.new_videoconvert.set_state(Gst.State.NULL)
                self.new_capssrc2.set_state(Gst.State.NULL)
                self.new_queue.set_state(Gst.State.NULL)

                # remove the caps elements of the source from pipeline
                self.new_capssrc.get_parent().remove(self.new_capssrc)
                self.new_jpegdec.get_parent().remove(self.new_jpegdec)
                self.new_videoconvert.get_parent().remove(self.new_videoconvert)
                self.new_capssrc2.get_parent().remove(self.new_capssrc2)
                self.new_queue.get_parent().remove(self.new_queue)

                # remove the new pad from the compositor
                self.comp.remove_pad(self.new_comp_sink_pad)

    def modify_pipeline(self, pad, info):
        print("pad_blocked")
        return Gst.PadProbeReturn.OK

    def _on_message(self, bus, message):
        mtype = message.type
        print("ErrorType: " + str(mtype))

    def stdin_cb(self, source, condition):
        print("stdin")
        line = source.readline()
        print("line is: {}".format(line))
        self.change_source(line)
        return True

def main():
    probeTest = ProbeTest()

    try:
        GLib.io_add_watch(sys.stdin, GLib.IO_IN, probeTest.stdin_cb)
        probeTest.loop.run()
    
    except KeyboardInterrupt:
        probeTest.pipeline.set_state(Gst.State.NULL)
        probeTest.loop.quit()
        exit(0)

if __name__ =='__main__':
    sys.exit(main())
```