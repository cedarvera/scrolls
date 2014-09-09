# Current Virtualization Setup
* * *
## Debian OS
###### (as of 2014-03-06)
* * *
### Links
* <http://wiki.debian.org/LXC>
* <http://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/Virtualization_Administration_Guide/sec-lvm-storage-pools-virt-mgr.html>

##### Host VM Server Packages
* lxc
* qemu-kvm
* libvirt-bin
* bridge-utils
* lvm2
* __OPTIONAL__: xvfb
* __OPTIONAL__: x11vnc

##### Client Packages
* virt-manager

* * *
## Initial Setup
### Host VM Server
##### Partitioning Scheme
Due to the issues of booting with grub2 and lvm, I recommend only having the home or swap partition be a lvm logical volume if desired.

* ~50GB root partition
* Rest in a lvm volume group
##### Network Bridge
* In `/etc/network/interfaces` remove eth0 (and/or other hardware ports) and add
        auto br0
        iface br0 inet dhcp
          bridge_ports eth0
          bridge_stp off
          bridge_maxwait 0
          bridge_fd 0
* Add any other interface to `bridge_ports` if desired
* __TODO__: NAT

##### Linux Containers
* __OPTIONAL__: Add user to lxc group (probably can be bypassed via libvirt)
        usermod -a -G lxc <user>
* Add to `/etc/fstab`
        cgroup    /sys/fs/cgroup    cgroup    defaults    0    0
* Mount or reboot
        mount /sys/fs/cgroup
* The Debian wiki says _optional_ but virt-manager complains if not set
  * In `/etc/default/grub`
    * Modify `GRUB_CMDLINE_LINUX=""`
            GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
    * Run
            update-grub2
    * Reboot
* To verify kernel can run LXC, run
        lxc-checkconfig

##### KVM
* __OPTIONAL__: Add user to kvm or qemu group
        usermod -a -G kvm <user>
* Check if computer can run kvm
        egrep '(vmx|svm)' --color=always /proc/cpuinfo
* Check if needs to enabled in BIOS

##### libvirt
* Add user to libvirt group (or `unix_sock_group` in `libvirtd.conf`)
        usermod -a -G libvirt <user>
* To allow remote connection (virt-manager), in `/etc/libvirt/libvirtd.conf` add/uncomment
        listen_tcp = 1
* __TODO__: Authentication based connection

* * *
## Client
##### virt-manager
* __OPTIONAL__: Install in the Host VM server
* To bypass setting up the TLS authentication, I used X forwarding over SSH after installing in host
  * I used a headless X server `xvfb`
* __TODO__: Another idea is to use x11vnc with xvfb
###### LVM
* Create a volume group to hold the VMs
* In virt-manager goto `Edit->Connection Details`
  * Go to the `Storage` tab
  * Press the `+` button to add a new storage pool
  * Add name and change the type to `LVM` and got to the next page
  * The `Target Path` is the location of the volume group
  * __OPTIONAL__: If `Target Path` is not a preexisting volume then the `Source Path` should be provided and `Build Pool` checked.
* Based on what I gather, a new logical volume would have to be created and selected when creating new VM.

* * *
### Usage
##### Qemu (OSX)
* Install qemu
        brew install qemu
* Create a image
        qemu-img create <image>.qcow <size>
* Start the vm with the installation ISO
        qemu-system-x86_64 -cdrom <iso image> -boot order=d -smp <cpu count> -m <ram size> -vga std <image>.qcow
* Start the vm without the installation ISO with SDL window
        qemu-system-x86_64 -boot order=c -smp <cpu count> -m <ram size> -vga std <image>/qcow
__TODO__
