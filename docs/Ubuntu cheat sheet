# Ubuntu cheat sheet

man sed                                  # get manual of a command, for example sed command
ps -aux | grep <process_name>            # check all running processes
lspci                                    # check GPU version
lsb_release -a                           # check ubuntu version 
grep -r "word" /path/to/files            # find a word inside all files
uptime                                   # check computer running time
dmesg                                    # show usb connections
find . -iname <file_name>                # find a file
which <file_name>                        # find the file path
chown -Rfv user:user /tmp                # change user and group owner of /tmp (or any folder/file)
df -h                                    # show the free disk (-h= human readable, -a= all, even if the disk is 0)
sudo tcpdump -n -i lo <protocol_name>    # check connections (-n= no resolve, lo= local card, protocol_name= arp,icmp etc.)
                                           for example: sudo tcpdump -n -i any icmp = check connection for any card

## Installations

dpkg -l | grep <package_name>            # check for dpkg installations
dpkg -i <file_name>.deb                  # install deb file
cat /etc/apt/sources.list                # get link to all ubuntu installations and network deb files
cat /etc/hosts                           # file for changing IP and for adding external server machine

## Networks

ifconfig                                 # check my IP
ip addr                                  # see all IP addresses, as ifconfig
sudo ifconfig eth0:1 <new_ip>/24         # add virtual IP address. 24 is for subnet 255.255.255.0
netstat -nau                             # check udp processes
netstat -naup                            # check udp processes with details
netstat -natp                            # check tcp processes
arp -n                                   # check connection history


## Open a file

code <file_name>                         # open visual studio code IDE
vim <file_name>                          # terminal with edit option
less <file_name>                         # terminal - show content one page at a time
cat <file_name>                          # terminal short view

