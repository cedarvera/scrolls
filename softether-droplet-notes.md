### Initial Debian Setup

* <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-8>

### ufw Firewall

Problems getting it to work with everything on complicated setups.

* <https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server>

Ports to allow:

* **HTTP**: TCP 80
* **HTTPS**: TCP 443
* **SoftEther VPN** and **OpenVPN**: TCP 443, TCP 992, TCP 5555
* **L2TP/IPSEC**, **L2TPv3** and **EtherIP**: UDP 500, UDP 4500
* **dnsmasq**: 53, 67 (for local bridge)

Don't forget to enable ssh port.

    ufw allow in on eth0 to any port 22

Annoyingly it is ordered by when it is added instead of by port number so for a quick copy-paste

    ufw allow in on eth0 to any port 22
    ufw allow in on {tap interface} to any port 53
    ufw allow in on {tap interface} to any port 67
    ufw allow 80/tcp
    ufw allow 443/tcp
    ufw allow 500/udp
    ufw allow 992/tcp
    ufw allow 1194/udp
    ufw allow 4500/udp
    ufw allow 5555/tcp

**TODO** Use iptable

### SoftEther

* <https://www.digitalocean.com/community/tutorials/how-to-setup-a-multi-protocol-vpn-server-using-softether>
* <https://www.softether.org/4-docs/1-manual/A._Examples_of_Building_VPN_Networks/10.6_Build_a_LAN-to-LAN_VPN_%28Using_L3_IP_Routing%29>

#### Other how-tos

* <http://bunic.si/2015/04/l3-bridge-with-two-raspberry-pi-2/>
* <http://blog.lincoln.hk/blog/2013/03/19/softether-on-vps/>
* <http://blog.lincoln.hk/blog/2013/05/17/softether-on-vps-using-local-bridge/>

### For local bridge with ufw and dnsmasq

#### Setup local bridge

In `vpncmd`:

    BridgeCreate [hubname] /DEVICE:soft /TAP:yes

Name of the tap was inconsistent with the one created in vpncmd so verify with ip addr or ifconfig (The name comes out as tap_soft).

#### The resulting sysvinit file

Create file `vpnserver` in `/etc/init.d/`

    #!/bin/sh
    # chkconfig: 2345 99 01
    # description: SoftEther VPN Server
    DAEMON=/usr/local/vpnserver/vpnserver
    LOCK=/var/lock/subsys/vpnserver
    TAP_ADDR=192.168.50.1
    
    test -x $DAEMON || exit 0
    case "$1" in
    start)
    $DAEMON start
    touch $LOCK
    sleep 1
    /sbin/ifconfig {tap interface} $TAP_ADDR
    ;;
    stop)
    $DAEMON stop
    rm $LOCK
    ;;
    restart)
    $DAEMON stop
    sleep 3
    $DAEMON start
    sleep 1
    /sbin/ifconfig {tap interface} $TAP_ADDR
    ;;
    *)
    echo "Usage: $0 {start|stop|restart}"
    exit 1
    esac
    exit 0

#### Setup dnsmasq
In `/etc/dnsmasq.conf` add:

    interface={tap interface}
    dhcp-range={tap interface},192.168.50.50,192.168.50.150,12h
    dhcp-option={tap interface},3,192.168.50.1

For static routing for bridge to bridge:

    dhcp-option=121,{IP_RANGE_01},192.168.50.254,{IP_RANGE_02},192.168.50.254

#### Setup ufw

In `/etc/default/ufw` change parameter `DEFAULT_FOWARD_POLICY`:

    DEFAULT_FORWARD_POLICY="ACCEPT"

In `/etc/ufw/before.rules` add before the filter rules:

Replace {LOCAL\_IP\_RANGE} with the bridge IP range such as 192.168.50.0/24

    # nat Table rules
    *nat
    :POSTROUTING ACCEPT [0:0]
    # Forward traffic from Softether through eth0
    -A POSTROUTING -s {LOCAL_IP_RANGE} -o eth0 -j MASQUERADE
    # tell ufw to process the lines
    COMMIT

Allow the ports to dnsmasq

    ufw allow in on {tap interface} to any port 53
    ufw allow in on {tap interface} to any port 67


### Setup ipv4 forwarding

Add/uncomment in file /etc/sysctl.conf

    net.ipv4.ip_forward = 1

Apply

    sysctl --system

### Restart the services

    /etc/init.d/vpnserver restart
    /etc/init.d/dnsmasq restart
    ufw disable && ufw enable

## Create a layer 3 site-to-site vpn

### Softether Server

#### Create a hub for each network
In `vpncmd`:

    HubCreate [hubname0]
    HubCreate [hubname1]

**NOTE:** We don't need to setup SecureNat or local bridge for these hubs as the networks already have their own dhcp servers.

#### Create layer 3 switch
Uses router commands in vpncmd:

    RouterAdd       - Define New Virtual Layer 3 Switch
    RouterDelete    - Delete Virtual Layer 3 Switch
    RouterIfAdd     - Add Virtual Interface to Virtual Layer 3 Switch
    RouterIfDel     - Delete Virtual Interface of Virtual Layer 3 Switch
    RouterIfList    - Get List of Interfaces Registered on the Virtual Layer 3 Switch
    RouterList      - Get List of Virtual Layer 3 Switches
    RouterStart     - Start Virtual Layer 3 Switch Operation
    RouterStop      - Stop Virtual Layer 3 Switch Operation
    RouterTableAdd  - Add Routing Table Entry for Virtual Layer 3 Switch
    RouterTableDel  - Delete Routing Table Entry of Virtual Layer 3 Switch
    RouterTableList - Get List of Routing Tables of Virtual Layer 3 Switch

Add a virtual layer 3 switch:

    RouterAdd [name]
    
Set the IPs of the switch on for each of the networks:

    RouterIfAdd [name] [/HUB:hub] [/IP:ip of switch/mask]

Check if the interfaces are setup:

    RouterIfList [name]

Start the switch:

    RouterStart [name]

### Bridge to the SoftEther Server

This is for each site.  After installing the vpnbridge software and starting `vpncmd`:

    Hub Bridge
    
Create a local bridge to the LAN (no for tap):

    BridgeCreate [hubname] [/Device:device_name] [/TAP:yes|no]

Find the device names:

    BridgeDeviceList

Uses cascade commands in `vpncmd`:

    CascadeAnonymousSet      - Set User Authentication Type of Cascade Connection to Anonymous Authentication
    CascadeCertGet           - Get Client Certificate to Use for Cascade Connection
    CascadeCertSet           - Set User Authentication Type of Cascade Connection to Client Certificate Authentication
    CascadeCompressDisable   - Disable Data Compression when Communicating by Cascade Connection
    CascadeCompressEnable    - Enable Data Compression when Communicating by Cascade Connection
    CascadeCreate            - Create New Cascade Connection
    CascadeDelete            - Delete Cascade Connection Setting
    CascadeDetailSet         - Set Advanced Settings for Cascade Connection
    CascadeEncryptDisable    - Disable Encryption when Communicating by Cascade Connection
    CascadeEncryptEnable     - Enable Encryption when Communicating by Cascade Connection
    CascadeGet               - Get the Cascade Connection Setting
    CascadeList              - Get List of Cascade Connections
    CascadeOffline           - Switch Cascade Connection to Offline Status
    CascadeOnline            - Switch Cascade Connection to Online Status
    CascadePasswordSet       - Set User Authentication Type of Cascade Connection to Password Authentication
    CascadePolicySet         - Set Cascade Connection Session Security Policy
    CascadeProxyHttp         - Set Connection Method of Cascade Connection to be via an HTTP Proxy Server
    CascadeProxyNone         - Specify Direct TCP/IP Connection as the Connection Method of Cascade Connection
    CascadeProxySocks        - Set Connection Method of Cascade Connection to be via an SOCKS Proxy Server
    CascadeRename            - Change Name of Cascade Connection
    CascadeServerCertDelete  - Delete the Server Individual Certificate for Cascade Connection
    CascadeServerCertDisable - Disable Cascade Connection Server Certificate Verification Option
    CascadeServerCertEnable  - Enable Cascade Connection Server Certificate Verification Option
    CascadeServerCertGet     - Get the Server Individual Certificate for Cascade Connection
    CascadeServerCertSet     - Set the Server Individual Certificate for Cascade Connection
    CascadeSet               - Set the Destination for Cascade Connection
    CascadeStatusGet         - Get Current Cascade Connection Status
    CascadeUsernameSet       - Set User Name to Use Connection of Cascade Connection
    
Create a cascade connection:

    CascadeCreate [name] [/SERVER:IP:port] [/HUB:hubname] [/USERNAME:username]
    
Set the password for the connection (use standard):

    CascadePasswordSet [name] [/PASSWORD:password] [/TYPE:standard|radius]
    
Available policies (not sure what they all do yet):

    DHCPFilter              - Filter DHCP Packets (IPv4)                      [yes|no]
    DHCPNoServer            - Disallow DHCP Server Operation (IPv4)           [yes|no]
    DHCPForce               - Enforce DHCP Allocated IP Addresses (IPv4)      [yes|no]
    CheckMac                - Deny MAC Addresses Duplication                  [yes|no]
    CheckIP                 - Deny IP Address Duplication (IPv4)              [yes|no]
    ArpDhcpOnly             - Deny Non-ARP / Non-DHCP / Non-ICMPv6 broadcasts [yes|no]
    PrivacyFilter           - Privacy Filter Mode                             [yes|no]
    NoServer                - Deny Operation as TCP/IP Server (IPv4)          [yes|no]
    NoBroadcastLimiter      - Unlimited Number of Broadcasts                  [yes|no]
    MaxMac                  - Maximum Number of MAC Addresses                 [num]
    MaxIP                   - Maximum Number of IP Addresses (IPv4)           [num]
    MaxUpload               - Upload Bandwidth                                [num]
    MaxDownload             - Download Bandwidth                              [num]
    RSandRAFilter           - Filter RS / RA Packets (IPv6)                   [yes|no]
    RAFilter                - Filter RA Packets (IPv6)                        [yes|no]
    DHCPv6Filter            - Filter DHCP Packets (IPv6)                      [yes|no]
    DHCPv6NoServer          - Disallow DHCP Server Operation (IPv6)           [yes|no]
    CheckIPv6               - Deny IP Address Duplication (IPv6)              [yes|no]
    NoServerV6              - Deny Operation as TCP/IP Server (IPv6)          [yes|no]
    MaxIPv6                 - Maximum Number of IP Addresses (IPv6)           [num]
    FilterIPv4              - Filter All IPv4 Packets                         [yes|no]
    FilterIPv6              - Filter All IPv6 Packets                         [yes|no]
    FilterNonIP             - Filter All Non-IP Packets                       [yes|no]
    NoIPv6DefaultRouterInRA - No Default-Router on IPv6 RA                    [yes|no]
    VLanId                  - VLAN ID (IEEE802.1Q)                            [num]

* See <https://www.softether.org/4-docs/1-manual/3._SoftEther_VPN_Server_Manual/3.5_Virtual_Hub_Security_Features>

Set `DHCPFilter` and `DHCPv6Filter` to `yes` with:

    CascadePolicySet [name] [/NAME:policy_name] [/VALUE:num|yes|no]
    
If concerned with bandwith you can set policy `ArpDhcpOnly` to `yes`.

See the policy setup:

    CascadeGet [name]

**TODO:** What else I need to change for site-to-site?

Bring cascade online:

    CascadeOnline [name]
    
Check status:

    CascadeStatusGet [name]
    
If settings are changed then restart the cascade connection by:

    CascadeOffline [name]
    CascadeOnline [name]
    
#### Setup static routing on the local routers

Remember to add static routing so that packets that are bound for other subnets go to the IP of the switch.  In addition make sure the IP of the switch would not be auto-assigned to another device.

In pfsense, go to System->Routing and add a gateway for the switch's IP address.  Add static routes such that any requests to other subnets are then directed to the newly added gateway.

If using SecureNAT, then add static routes using:

    DhcpSet [/START:start_ip] [/END:end_ip] [/MASK:subnetmask] [/EXPIRE:sec] [/GW:gwip] [/DNS:dns] [/DNS2:dns2][/DOMAIN:domain] [/LOG:yes|no] [/PUSHROUTE:"routing_table"]
