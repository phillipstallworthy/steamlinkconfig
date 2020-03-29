# SteamLink
Raspberry Pi to Windows 10 SteamLink config notes.

## Wot
Raspberry pi connected to SteamLink to Windows 10 gaming PC (PC)
Wifi - just connecting over wifi is too slow.
Both connected to modem by network cable - still slow (10 Mbps)
Network cable runs between Pi and Win (wifi still works for browsing).Dunno.

## Both connected to the modem
Network cable from Pi and Windows to modem.
Make sure all config files on Pi are default, IE dhcpcd.config
Retore Windows ethernet configuration to auto, dhcp.
Turn off wifi on both
Ta-da! It just works. Not very well, only gets 10 Mbs, but does work.
Note. Modem was caching static IP that I set up earlier, fixed by modem reboot.

## Cable between them.

### Ethernet cable.
Connect the pi to the pc with a simple straight RG45 cable. No crossover required. (not sure why, modern network cards just switch modes I guess)
Make sure the green and orange LED's go on.

### Current state
* Windows wifi set to share internet connection which always used ip 192.168.137.1 - Keeps this ip even if sharing is turned off. I don't need sharing so turned off. 
* Windows firewall allows file and print services pub and private. (check me)
* Pi ip set to 192.168.137.2. This puts it on the same network segment 192.168.137.0
 * done by setting ip in dhcpcd.conf
>  Example static IP configuration:
interface eth0
static ip_address=192.168.137.2/24
* check Pi routes
 * `route -n`
* check Win routes
 * `route print`

90Mbps on SteamLink from Pi to Win.
Need to turn the wifi off on the Pi to get it to work. (maybe a metric solution to that.)

### Some facts
192.169.1.1 modem IP address. 

### Static IP on Pi and Win.
Is static IP the way to go? Seems like it should be. The modem runs a dhcp server to assign IP addresses to devices that connect. Pi runs dhcp client (dhcpcd), Win must do something similar. Sure that neither are running a DHCP server, so how else would they get an IP?
**Static IP leads to a world of pain**

### Static ip on Windows
Setting, network config, IP4 proporties.
IP 192.168.1.50 - same range as modem. It assigns last number 1 upwards.
Network Mask 255.255.255.0 - same 
Default Route - Configure so that Pi and Win have default route of each other. Does that make sense??

### Static ip on Pi - 
ifconfig on the PI show the WIFI connected to wlan0 with an IP of 192.168.1.11

### Pi eth0 no config
No changes on PI, just plugged in cable. Where did this come from???
 inet 169.254.22.144  netmask 255.255.0.0  broadcast 169.254.255.255

Possible pseudo random!
https://raspberrypi.stackexchange.com/questions/34132/ssh-into-pi-why-static-ip-169-254-149-192-always

Avahi sets up a link-local ip address. 


### Internet connection sharing
Comes up alot on posts as a way to do it. Don't want to as them the Pi would rely on Windows for internet. Hang on, maybe not. Pi would have wifi too.

Setting up wifi sharing on Windows leads to an IP of 192.168.137.1

.

### Brain dump

The lower the metric, the more likly to use a route, FYI.

### Route table changes.
Required? Don't think so, but i tryed anyway
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    301    0        0 wlan0
192.168.1.0     0.0.0.0         255.255.255.0   U     301    0        0 wlan0
All routes go the wlan0 - IE the wifi. Not what we want. Wifi is not fast enough for SteamLink.

ping 192.168.2.1 works, but it goes via the WIFI router. How do I know??

$ ip route get  192.168.2.1 
192.168.2.1 dev wlan0 src 192.168.2.2 uid 1000

so add a route:
sudo route add -host 192.168.2.1  dev eth0

route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    301    0        0 wlan0
192.168.1.0     0.0.0.0         255.255.255.0   U     301    0        0 wlan0
192.168.2.1     0.0.0.0         255.255.255.255 UH    0      0        0 **eth0**

route to the host 192.168.2.1 is not via the eth0 interface. 

Again, there may be anther way (metrics??), but this does work.

### Add a static IP to windows
Settings - Network - properites - IP4 - follow your nose.

### turn off the windows firewall
Maybe not required. Not sure yet.

### TODO
route from windows to PI?? tracert command
do we need a route on windows?? dunno how to do that?

### Pi commands

* `$ sudo service dhcpcd status` **"status of  dhcp, where did it get the ip from?"**
* `$ tail -n100 /var/log/daemon.log`
* `$ tail -n100 /var/log/syslog`
* `$ tail -n100 /var/log/kern.log`
* `$ tail -n100 /var/log/auth.log`
* `$ cat etc/os-release` **"version of pi OS"**
* `$ ip route get 192.168.1.1` **"route to a destination, ie check eth or wlan"**
* `$ cmd` **"doc"**

### Info
https://en.wikipedia.org/wiki/IP_address