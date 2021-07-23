# CanBus - Controller Area Network

Is a standardized communication protocol designed to allow microcontrollers and other devices to communicate with each other.
&nbsp;

## Installation:

pip install python-can

pip install cantools
&nbsp;
&nbsp;

## Activation on jetson (TX2):
sudo modprobe can

sudo modprobe can_raw

sudo modprobe mttcan

sudo ip link set can0 type can bitrate 250000 restart-ms 100 loopback

sudo ip link set can0 up
&nbsp;
&nbsp;

## Generate dbc file (canbus databases)
The dbc file can be generated as explained in [intro to dbc](https://www.csselectronics.com/screen/page/can-dbc-file-database-intro/language/en).
Using a program named Kvaser Database editor, you can create a simple dbc file according to your own messages protocol. 

An easy explanation can be found [here](https://www.youtube.com/watch?v=mAL1Xo2G9-4)

Simple example - my_cam.dbc:
```
VERSION ""

NS_ :
    CM_
    BA_DEF_
    BA_
    BA_DEF_DEF_

BS_:

BU_:

BO_ 50 TEST: 8 Vector_XXX
    SG_ ThrottleSpeed : 0|8@1+ (0.125,0) [0|0] "rpm_precent" Vector_XXX

BO_ 2566844926 CCVS1: 8 Vector_XXX
    SG_ WheelBasedVehicleSpeed : 8|16@1+ (0.00390625,0) [0|250.996] "km/h" Vector_XXX

CM_ BO_ 50 "Electronic Engine Controller 1";
CM_ SG_ 50 ThrottleSpeed "Actual engine speed"
```
&nbsp;
&nbsp;

## Communication - send and receive can messages
```
import sys
import os
import can
from can.bus import BusState
from threading import Thread, Event
import logging
import cantools
import time

log = logging.getLogger(__name__)

class CanNode:
    def __init__(self,db):
    self.db = db

    def send_encoded_message(self, bus, stop_event):
        """loop for sending encoded can messages"""
        log.warn("Start sending a message every 1s")
        start_time = time.time()
        while not stop_event.is_set():
            throttle_message = self.db.get_message_by_name('TEST')
            data = throttle_message.encode({'ThrottleSpeed': 10})
            msg = can.Message(arbitration_id = throttle_message.    frame_id, data = data, timestamp = time.time() -    start_time)
            bus.send(msg)
            log.warn("tx: {}".format(msg))
            time.sleep(1)
        log.warn("Stop sending messages)

    def receive_encoded_message(self, bus, stop_event)
        """loop for receiving"""
        log.warn("Start receiving can messages")
        while not stop_event.is_set():
            rx_msg = bus.recv(1)
            if rx_msg is not None:
                log.warn("rx: {}".format(rx_msg))
                log.warn("decoded: {}".format(self.db.decode_message(rx_msg.arbitration_id, rx_msg.data)))
        log.warn("Stop receiving messages")

def main():
    db = cantools.database.load_file('my_can.dbc')
    can_node = CanNode(db)
    with can.interface.Bus(bustype='socketcan', channel='can0', bitrate=250000) as bus:
        stop_event = Event()
        send_cyclic_thread = Thread(target = can_node.send_encoded_message, args=(bus, stop_event))
        receive_thread = Thread(target = can_node.receive_encoded_message, args=(bus, stop_event))

        send_cyclic_thread.start()
        receive_thread.start()

        try:
            while True:
                time.sleep(0)
        except KeyboardInterrupt:
            stop_event.set()
            time.sleep(0.5)
            exit(0)

if __name__=='__main__':
    sys.exit(main())
        
```



