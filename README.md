# Authenticate-devices-with-ForeScout
# Understand the technique
Before moving on to the configuration. We need to understand the technique that protect network. Basically, the system will authenticate through 2 layers, 802.1x and DNS Enforce.
## 802.1X
So let me explain the workflow of the technique more detail.
![image alt](https://github.com/ThinhNT272/Authenticate-devices-with-ForeScout/blob/46819f2c3c024b231615e66feeebd4d189301171/Assets/802.1x.png)

When endpoint connect to the network, it need to authenticate 802.1x firstly. At this point,  Forescout is like a authentication server. It self is not an authentication server, but its role is query to the real server to decide authentication of a device.

Forescout can detect new devices based on SPAN port in the switch. When any devices connect to the switch which is configured 802.1x, the port of the switch will intercept any packet except EAP packet. If the device support 802.1x, it will send a  EAP-Request/Identify packet to the switch, the switch will transmit it to the Forescout to authentication. 

And if devices do not support  802.1x, after timeout, switch will trasnmit MAC address of devices to the Forescout to authenticate through MAB (Mac Address Bypass). Then devices will be sent to a VLAN correspond to its authenticate method. That is how 802.1X work to authenticate devices before allow it accesses to the network. But 802.1X is just authenticate for corporate devices, we also need a solution for guest devices.
## DNS Enforce
For guest devices. We need to configure DNS Enforce plugin in Forescout and setup that DNS to the DHCP server. So if Forescout detect any guest devices based on 802.1x, that device will be sent to another VLAN that has special DNS. This special DNS redirect devices to a login page and you can sign up at there.
![image alt](https://github.com/ThinhNT272/Authenticate-devices-with-ForeScout/blob/46819f2c3c024b231615e66feeebd4d189301171/Assets/DNS%20Enforce.png)

After that, the devices will be sent to another VLAN to check policy. Because it need to make sure that every devices connect to the network is believable.
![image alt](https://github.com/ThinhNT272/Authenticate-devices-with-ForeScout/blob/46819f2c3c024b231615e66feeebd4d189301171/Assets/Policy%20Compliance.png)

The devices need to make sure that it runs Anti Virus application, turns on Firewall, and updates to the newest version (Window only). If not, it will be sent to VLAN Un-Compliance policy to quarantine to the network until it compliance all policies.
# Configuration
Based on the technique above, we need to configure these things:
- Configure 802.1x and MAB (MAC Address Bypass) on NAS devices (AP, Switch,...) to act as RADIUS clients.
- Configure RADIUS plugin in Forescout include: Add AD server and authentication policy.
- Configure DNS Enforce plugin in Forescout  and authentication policy for guest devices
- Configure DNS server for quarantine VLAN to redirect guest devices to login page.

## Step 1: Configure 802.1x and MAB on NAS devices
Firstly, we turn on 802.1X on NAS device. In my case, I used WLC (Wireless LAN Controller) C9800. 
```cli
! — turn on AAA (802.1x) —
# config t
# aaa new-model

! — Update RADIUS Server. In this case is the Forescout —
# radius server <radius-server-name>
# address ipv4 <radius-server-ip> auth-port 1812 acct-port 1813

! — Time that NAS waits for EAP respond from guest devices —
# timeout 300
# retransmit 3
# key <shared-key>
# exit

! — Group RADIUS server — 
# aaa group server radius <radius-grp-name>
# server name <radius-server-name>
# exit

! — Configure CoA (Change of Authorization) —
# aaa server radius dynamic-author
# client <radius-server-ip> server-key <shared-key>

! — Link group RADIUS server to authentication dot1x (802.1x) —
# aaa authentication dot1x <dot1x-list-name> group <radius-grp-name>
```

Then, configure WLAN authentication 802.1x
```cli
# config t

# wlan <profile-name> <wlan-id> <ssid-name>
# security dot1x authentication-list <dot1x-list-name>

! - Turn on MAB as a fallback technique -
# security mac-filtering mab

# no shutdown
```

Configure policy
```cli
# config t

! — Create policy  for wlan —
# wireless tag policy <policy-tag-name>
# wlan <profile-name> policy <policy-profile-name>

! — Assign policy to specific AP —
# ap <ethernet-mac-addr>
# policy-tag <policy-tag-name>
# end
```

## Step 2: Configure RADIUS plugin on Forescout
