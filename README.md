# Virt-scripts

This suite of "quick and dirty" scripts are intended to mount virtual sandboxes for labs during Linux training courses with KVM/libvirtd with Centos 7 or Debian Jessie hosts. 
The main goal is to have on hand the major distributions by exploiting the Linux environment as well as possible. But we can find other subsequents objectives as programmation and automation, scripting and virtual network and system management. Only for educational purposes.

## Contents

### Native installation and post-installation

Purposes : gold image auto-creation

1. `autoprep.sh` : prepare your system as virtualization host
* `get-iso.sh` : get iso distributions for fresh installations
* `auto-install.sh` : build a fresh Centos, Debian or Ubuntu system with http repos and kickstart files
* `auto-install-tui.sh` : auto-install.sh text user interface demo
* `sparsify.sh` : optimize space disk on the designated guest
* `clone.sh` : clone, sysprep and optimize builded guests
* `hosts-file` : print the running guests and their ipv4 address
* `nested-physical.sh` : nested installation

### Quickbuilder

Purposes : deploy quickly guests based on pre-builded with previous scripts. Some images are avaible on https://get.goffinet.org/kvm : centos7 debian7 debian8 ubuntu1604 kali metasploitable openwrt15.05.

1. `define-guest-image.sh` : deploy pre-builded images (quickbuilder)
* `get_and_install_openwrt.sh` : get and start openwrt with two interfaces

### Devices management

Purposes : change RAM and vcpus, add block devices and network facilities 

1. `add-memory.sh` : add RAM
* `add-vcpu.sh` : set vcpus count
* `add-bridge.sh` : add an isolated or ipv4 nat/ipv6 ula libvirt bridge
* `add-nic.sh` : a new NIC on live guest to a bridged interface
* `attach-nic.sh` : attach a live guest present NIC to a bridge
* `detach-nic.sh` : detach a live guest from a bridge
* `add-storage.sh` : attach an empty bit disk by GB size
* `start_all.sh` : start all the defined guests
* `destroy_and_undefine_all.sh` : destroy,  undefine all the guests with storage removing

## Native installation and post-installation

### Step 1 : Verify your installation

Script : autoprep.sh

Description : Setup KVM/Libvirtd/LibguestFS on RHEL7/Centos 7/Debian Jessie.

Usage : 

```
# ./autoprep.sh
This script will install all the necessary packages to use Libvirtd/KVM
Please reboot your host after this step
Are you sure? [y/N]
```

### Step 2 : Get iso images (optionnal)

Script : get-iso.sh

Description : Get latest iso of Centos 7, Debian Jessie and Ubuntu Xenial.

Usage :

```
# ./get-iso.sh
Usage : ./get-iso.sh [ centos | debian | ubuntu ]
```

### Step 3 : Build a guest automatically

Script : auto-install.sh 

Description :  Centos 7, Debian Jessie or Ubuntu Xenial fully automatic installation by HTTP Repo and response file via local HTTP.

Usage :

```
# ./auto-install.sh
Centos 7, Debian Jessie or Ubuntu Xenial fully automatic installation by HTTP Repos and response file via local HTTP.
Usage : ./auto-install.sh [ centos | debian | ubuntu ] nom_de_vm
Please provide one distribution centos, debian, ubuntu and one guest name: exit
```

Note : Escape character is `^]` (CTRL+ `]`)


### Step 4 : Sparse your native image

Script : sparsify.sh

Description : Sparse a disk. Great gain on disk space !

Usage :

```
./sparsify.sh 
This script sparses an attached disk
Please provide a the guest name of a destroyed guest: exit
Usage : ./sparsify.sh <guest name>
```

Check the disk usage : 2,0G

```
# du -h /var/lib/libvirt/images/ubuntu-gold-31122016.qcow2
2,0G    /var/lib/libvirt/images/ubuntu-gold-31122016.qcow2
```

Sparsify operation

```
# ./sparsify.sh ubuntu-gold-31122016

Sparse disk optimization
[   0,1] Create overlay file in /tmp to protect source disk
[   0,1] Examine source disk
[   4,3] Fill free space in /dev/sda1 with zero
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ --:--
[   6,9] Fill free space in /dev/u1-vg/root with zero
 100% ⟦▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒⟧ 00:00
[  70,6] Clearing Linux swap on /dev/u1-vg/swap_1
[  71,9] Copy to destination and make sparse
[ 191,4] Sparsify operation completed with no errors.
virt-sparsify: Before deleting the old disk, carefully check that the
target disk boots and works correctly.
```

Check the disk usage : 432M


```
# du -h /var/lib/libvirt/images/ubuntu-gold-31122016.qcow2
432M    /var/lib/libvirt/images/ubuntu-gold-31122016.qcow2
```

### Step 5 : Clone your guest

Script : clone.sh

Description : Cloning a domain disk with sparsifying and Linux sysprep.

Usage :

```
./clone.sh
This script clones, sparsifies and syspreps linux guest
Usage : './clone.sh <original guest> <destination guest>'
Please provide a the guest name of a destroyed guest: exit
```

### Step 6 : Quickbuilder

Assume that you have installed three guests with `auto-install.sh` :

```
~/virt-scripts# ./auto-install.sh centos7 centos
~/virt-scripts# ./auto-install.sh debian8 debian
~/virt-scripts# ./auto-install.sh ubuntu1604 ubuntu
```

And you can verify it :

```
~/virt-scripts# virsh list --all
 Id    Name                           State
----------------------------------------------------
 -     centos7                        shut off
 -     debian8                        shut off
 -     ubuntu1604                     shut off

```

Move those images disk into the `virt-scripts` directory and undefine original guests :

```
cd ~/virt-scripts
for x in centos7 debian8 ubuntu1604
do
mv /var/lib/libvirt/images/$x.qcow2 ./
virsh undefine $x
done

```

And you can deploy quicky builded and optimized guests based on those images :

```
~/virt-scripts# ./define-guest-image.sh c1 centos7

Début d'installation...
Création du domaine...                                                   |    0 B     00:00
Création du domaine terminée.  Vous pouvez redémarrer votre domaine en lançant :
  virsh --connect qemu:///system start c1
```

```
~/virt-scripts# ./define-guest-image.sh d1 debian8

Début d'installation...
Création du domaine...                                                   |    0 B     00:00
Création du domaine terminée.  Vous pouvez redémarrer votre domaine en lançant :
  virsh --connect qemu:///system start d1
```

```
~/virt-scripts# ./define-guest-image.sh u1 ubuntu1604

Début d'installation...
Création du domaine...                                                   |    0 B     00:00
Création du domaine terminée.  Vous pouvez redémarrer votre domaine en lançant :
  virsh --connect qemu:///system start u1
```

### Step 7 : Manage guests

Verify your running guests

### Step 8 : Add the guest hostname resolution

Script : 

Description : Print a new `/etc/resolv.conf` with the ip address and the hostname of running guests.

Usage :

```
./hosts-file.sh
```

For example :

```

# ./hosts-file.sh
192.168.122.152 d1
192.168.122.236 d2
192.168.122.190 d3
192.168.122.155 c1
192.168.122.100 c2
192.168.122.40 c3
```

To update your `/etc/hosts` :

```
./hosts-file.sh >> /etc/hosts
```


### Manage devices

Script : add-bridge.sh 

Description : add an isolated or ipv4 nat/ipv6 ula libvirt bridge 

Usage :

```
./add-bridge.sh
Description : This script create an isolated or a nat/ipv6 bridge
Usage       : ./add-bridge.sh <name> <interface> <type, isolated or nat>
Example     : './add-bridge.sh net1 virbr100 isolated' or './add-bridge.sh lan101 virbr101 nat'
```

Script : add-storage.sh 

Description : attach an empty bit disk by GB size

Usage :

```
./add-storage.sh
Description : This script attach a disk to a live guest
Usage       : ./add-storage.sh <guest name> <block device name> <size in GB>
Example     : './add-storage.sh guest1 vdb 4' add a vdb 4GB disk to guest1
```

To be continued ...

### Next steps ...

* Install ansible, add ssh hosts keys, create an ansible inventory and test your managed nodes.
* Exploit snapshots and virtual storage
* Exploit free-ipa, pacemaker, ovirt, openstack, gns3

## Todo

* `auto-install.sh` 
  * Fedora
* `create_repo.sh` : create local repo
