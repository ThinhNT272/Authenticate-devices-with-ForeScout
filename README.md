# Authenticate-devices-with-ForeScout
# Understand the technique
Before moving on to the configuration. We need to understand the technique that protect network. Basically, the system will authenticate through 2 layers, 802.1x and DNS Enforce.
## 802.1X
So let me explain the workflow of the technique more detail.
![image alt]()

When endpoint connect to the network, it need to authenticate 802.1x firstly. At this point,  Forescout is like a authentication server. It self is not an authentication server, but its role is query to the real server to decide authentication of a device.

Forescout can detect new devices based on SPAN port in the switch. When any devices connect to the switch which is configured 802.1x, the port of the switch will intercept any packet except EAP packet. If the device support 802.1x, it will send a  EAP-Request/Identify packet to the switch, the switch will transmit it to the Forescout to authentication. 

And if devices do not support  802.1x, after timeout, switch will trasnmit MAC address of devices to the Forescout to authenticate through MAB (Mac Address Bypass). Then devices will be sent to a VLAN correspond to its authenticate method. That is how 802.1X work to authenticate devices before allow it accesses to the network. But 802.1X is just authenticate for corporate devices, we also need a solution for guest devices.
## DNS Enforce
For guest devices. We need to configure DNS Enforce plugin in Forescout and setup that DNS to the DHCP server. So if Forescout detect any guest devices based on 802.1x, that device will be sent to another VLAN that has special DNS. This special DNS redirect devices to a login page and you can sign up at there.
![image alt]()

After that, the devices will be sent to another VLAN to check policy. Because it need to make sure that every devices connect to the network is believable.
![image alt]()

The devices need to make sure that it runs Anti Virus application, turns on Firewall, and updates to the newest version (Window only). If not, it will be sent to VLAN Un-Compliance policy to quarantine to the network until it compliance all policies.
# Configuration
