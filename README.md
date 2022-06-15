# virt-manager Ubuntu20
Notes on creating a vms and a bridged network in ubuntu20.

# The methosd in this link feels wrong, cobbled together.
https://linuxize.com/post/how-to-install-kvm-on-ub

sudo useradd virtkvm
sudo passwd virtkvm
sudo usermod -aG kvm virtkvm
sudo usermod -aG libvirt virtkvm
su virtkvm

# Create new bridge
brctl show
https://levelup.gitconnected.com/how-to-setup-bridge-networking-with-kvm-on-ubuntu-20-04-9c560b3e3991

what is netfilter?
https://www.netfilter.org/

# edit netplan config
gateway4 is depreciated
https://www.techrepublic.com/article/set-default-gateway-with-netplan-now-gateway4-been-deprecated/

# THIS IS BETTER
https://linuxconfig.org/how-to-use-bridged-networking-with-libvirt-and-kvm
https://help.ubuntu.com/community/KVM/Networking#bridgednetworking

#THIS IS GREAT
https://jamielinux.com/docs/libvirt-networking-handbook/

https://wiki.archlinux.org/title/network_bridge
http://www.policyrouting.org/iproute2.doc.html

https://wiki.libvirt.org/page/Networking
learn iptables usage
https://libvirt.org/formatnetwork.html
https://www.libvirt.org/manpages/virsh.html #this is important


# Start Here
https://wiki.libvirt.org/page/Networking
Bridged networking (aka "shared physical device")
Host configuration

The NAT based connectivity is useful for quick & easy deployments, or on machines with dynamic/sporadic networking connectivity. More advanced users will want to use full bridging, where the guest is connected directly to the LAN. The instructions for setting this up vary by distribution, and even by release.

Important Note: Unfortunately, wireless interfaces cannot be attached to a Linux host bridge, so if your connection to the external network is via a wireless interface ("wlanX"), you will not be able to use this mode of networking for your guests.

Important Note: If, after trying to use the bridge interface, you find your network link becomes dead and refuses to work again, it might be that the router/switch upstream is blocking "unauthorized switches" in the network (for example, by detecting BPDU packets). You'll have to change its configuration to explicitly allow the host machine/network port as a "switch".

Debian/Ubuntu Bridging

See the debian wiki for up to date instructions of bridging: https://wiki.debian.org/BridgeNetworkConnections

# Libvirt and bridging

Libvirt is a virtualization API that supports KVM (and various other virtualization technologies). It's often desirable to share a physical network interface with guests by creating a bridge. This usually offers excellent performance and doesn't require NAT. This operation is composed of two parts:

    Setup the bridge interface on host as described in this article, 
      [here](https://wiki.libvirt.org/page/Networking#Altering_the_interface_config) 
    or 
      [here](https://jamielinux.com/docs/libvirt-networking-handbook/bridged-network.html)
    
    Configure guest to use the newly-created bridge 

The libvirt Networking Handbook provides thorough instructions.
https://jamielinux.com/docs/libvirt-networking-handbook/

You can verify if bridging is working properly by looking at brctl output:

root@server:/etc/libvirt/qemu# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.001ec952d26b       yes             eth0
                                                        vnet0
                                                        vnet1
                                                        vnet2
virbr0          8000.000000000000       yes

As can be seen, guest network interfaces vnet0, vnet1 and vnet2 are bound with the physical interface eth0 in the bridge br0. The virbr0 interface is only used by libvirt to give guests NAT connectivity. 


# Guest configuration

In order to let your virtual machines use this bridge, their configuration should include the interface definition as described in Bridge to LAN. In essence you are specifying the bridge name to connect to. Assuming a shared physical device where the bridge is called "br0", the following guest XML would be used:

 <interface type='bridge'>
    <source bridge='br0'/>
    <mac address='00:16:3e:1a:b3:4a'/>
    <model type='virtio'/>   # try this if you experience problems with VLANs
 </interface>

NB, the mac address is optional and will be automatically generated if omitted.

To edit the virtual machine's configuration, use:

virsh edit <VM name>

For more information, see the FAQ entry at:

http://wiki.libvirt.org/page/FAQ#Where_are_VM_config_files_stored.3F_How_do_I_edit_a_VM.27s_XML_config.3F 
  
 # virsh instead of virt-manager
  https://www.libvirt.org/manpages/virsh.html 
  
