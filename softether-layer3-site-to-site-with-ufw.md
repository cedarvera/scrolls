2015-09-21

### Introduction

### Install SoftEther

### VPS Server

#### Setup ufw

SoftEther VPN and OpenVPN: TCP 443, TCP 992, TCP 5555

L2TP/IPSEC, L2TPv3 and EtherIP: UDP 500, UDP 4500

#### Set-up hub for each client site

#### Setup separate hub for remote access

#### Create virtual layer 3 switch

##### Option A: SecureNat

##### Option B: Setup local bridge

dnsmasq ports 53 and 67

### Client

This will be done for each client site.  Go to SoftEther to download the bridges package and install it with the same steps as `vpnserver` except replace with `vpnbridge`.
* Go to `/usr/local/vpnbridge` and run `./vpncmd`
* Press 1 to select "Management of VPN Server or VPN Bridge"
* Press Enter to connect to the localhost server
* Press Enter again to connect to server by server admin mode.
* Optional: setup the pass with `ServerPasswordSet` (NEVER TRIED)
* By default it already has one preset hub go to it by running `Hub Bridge`

#### Setup-up local bridge

Show the device list to connect the local bridge to by running:

    BridgeDeviceList

If the device name is `eth0` then create the local bridge without creating a tap by running:

    BridgeCreate Bridge /DEVICE:eth0 /TAP:no

#### Setup-up cascade connection

### References
##### Digital Ocean
* [How to Setup a Multi-Protocol VPN Server Using SoftEther](https://www.digitalocean.com/community/tutorials/how-to-setup-a-multi-protocol-vpn-server-using-softether)
* [How To Setup a Firewall with UFW on an Ubuntu and Debian Cloud Server](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server)

##### SoftEther
* [SoftEther Specifications and Capabilities](https://www.softether.org/3-spec)

##### ufw
* [ufw should support '_' in interface names](https://bugs.launchpad.net/ufw/+bug/1098472)

##### How-Tos

##### Other
* [Softether vpn ufw firewall config - is it correct?](https://serverfault.com/questions/626317/softether-vpn-ufw-firewall-config-is-it-correct)

