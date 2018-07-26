# raspberry-crumble

## Overview

Configuration and code for test Raspberry Pi Cluster.

I've seen lots of examples of people building test 'super computers' out of Raspberry Pi's and I wanted to have a go myself.  I got a free [Meraki](https://meraki.cisco.com) router after watching one of their webinars and thought this would make an excellent excuse, I mean, solution for building and powering the cluster.  The main reason being this particular router, [MS220-8P](https://meraki.cisco.com/products/switches/ms220-8), has PoE on all ports making it easy to hook up each Pi with a splitter.


## Hardware

### Infra-structure

As mentioned already, this is using one [MS220-8P](https://meraki.cisco.com/products/switches/ms220-8) switch.

Also I purchased several of these PoE to [Micro USB splitters](https://www.amazon.co.uk/gp/product/B01H37XQP8/ref=oh_aui_detailpage_o03_s00?ie=UTF8&psc=1) to power each of the Raspberry Pi's.


### Raspberry Pi

Currently I have five Raspberry Pi nodes, 4 are RPi 3, and one RPi 2.  This last one is rather odd as it's also connected to a [Logi-Pi](http://valentfx.com/logi-pi/) FPGA board.  That could prove useful/fun in terms of accelerating computation/processing later but I haven't got round to doing anything with it yet.

Each node has 16Gb MicroSSD though the 'master' has 32Gb.

I've also added the odd extra USB memory into the cluster and also planning to get an external drive for more mass storage.


## Setup

### Pre-requisites

Each Raspberry Pi is setup with the lastest Raspbian (Jessie) Image with the following tweaks

* Static IP for eth0
* SSH enabled
* Desktop disabled
* GPU memory set to 128Mb (planning on trying to run GPU-acclerated computations at some point
* File system expanded to full card
* Each node has a DNS name which is resolved by my local DNS server
* Copied an SSH public key to each node
* Set the password on each node to be the same (bad security I know)

### Network Setup

As per the notes below, using static addresses and DNS configuration across the cluster 
is incredibly complicated and ends up fighting against resolvconf.  Instead, the cluster relies on 
a router with DNS/DHCP that has reservations statically assigned.   


### Installing ansible

Since Ansible has been packaged for use with pip I've used virtualenv for python to allow a local installation that doesn't need admin access or muck up any of your system libraries.  I prefer this approach as it's cleaner and has less in the way of side-affects.  Feel free to install ansible from RPMs/.deb or other package managers onto your system natively if that is easier for you.

Create a virtual python environment and activate it

```
virtualenv ansible-env
. ansible-env/bin/activate
```

Install ansible

```
# Need to install/build OpenSSL first 
LDFLAGS="-L/usr/local/lib" pip install cryptography --no-use-wheel
pip install ansible
```


### Running ansible

To initialise brand new raspberry pi devices (from a clean Raspbian Image)

```
ansible-playbook -v -i inventory/hosts --extra-vars '{ "pi_password":"<unencrypted password>", "init_hosts":[ "dieter", "heimlich", "horst" ]}' --ask-become-pass --ask-pass initialise_pi_hosts.yml
```

If you have already run the above once and the password step has already been executed, then you must remember to
use the new password for subsequent attempt to run the same playbook

To build the entire cluster run the following command

```
ansible-playbook -i inventory/hosts --ask-become-pass build_cluster.yml
```

To just run part of the build use the tags, for example

```
ansible-playbook -i inventory/hosts -t master --ask-become-pass build_cluster.yml
```

### Ad hoc commands

#### Shutting down all machines

Shutting down all the machines can be done using the following

```
ansible -u pi --become --ask-become-pass allnodes -i inventory/hosts -a "shutdown -h now"
```

It drops the connection immediately


#### Stopping the Wifi

Since most (if not all of the machines now) in my cluster are RPi 3 there is a possibility that the WiFi on each is
pinging out traffic to look for access points (or maybe just using power anyway sitting there idle).  So this command 
was used to shut down the wlan0 interface until such time as I got a chance to integrate and test a new role in the 
ansible config that permanently shuts off the wifi devices (this is merged in as part of the update that includes this 
README change but not tested yet)

```
ansible -u pi --become --ask-become-pass allnodes -i inventory/hosts -a "ifconfig wlan0 down"
``` 


## Documentation

### Meraki

Firewall [ports](https://documentation.meraki.com/zGeneral_Administration/Other_Topics/Firewall_Rules_for_Cloud_Connectivity)


### Ansible

Managing [apt with ansible](http://docs.ansible.com/ansible/apt_module.html)

Managing [user creation with ansible](http://docs.ansible.com/ansible/user_module.html) and creating
[groups](http://docs.ansible.com/ansible/group_module.html)

Managing [SSH keys with ansible](http://blog.appliedinformaticsinc.com/how-to-manage-ssh-keys-using-ansible/)

Fetching [remote files](http://docs.ansible.com/ansible/fetch_module.html)


### MPICH

Installing [MPICH on Debian](http://www.cslu.ogi.edu/~zak/debianclusters/Installing_MPICH)


### Raspbian

Cleaning up ["untrusted versions of the following ..."](http://serverfault.com/questions/444798/debian-warning-untrusted-versions-of-the-following-packages-will-be-installe)
error from Debian when trying to upgrade.


### Running raspi-config noninteractive

Found post on [this](https://www.raspberrypi.org/forums/viewtopic.php?f=63&t=21632)

### Trying to use static addresses

Previously I've used static IP addresses for each node.  This has proved to be somewhat annoying as 
the Raspberry Pi is setup to default to DHCP and also makes use of 
[resolveconf](http://manpages.ubuntu.com/manpages/zesty/man8/resolvconf.8.html) 
which generates a resolv.conf file overwritting anything that is entered for a static configuration. 

According to the debian [instructions](https://wiki.debian.org/NetworkConfiguration#The_resolvconf_program)
and further details eluded to [here](https://wiki.debian.org/NetworkConfiguration#A.2Fetc.2Fnetwork.2Finterfaces)
the following lines in /etc/network/interfaces is enough to ensure that DNS servers will be set by resolveconf

```
dns-nameservers 1.2.3.4
dns-search some.domain.com
```

I think for a bigger cluster or if I build/rebuild this again I'll just rely on statically-assigned
DHCP leases and naming as mucking around with all of this is overly complicating matters :)


### Need to change password

Since the initial setup of the raspberry pi boards needs the pi password to be changed
(or it gets upset since it's using the default password) need to override the password initially passed 
into the ansible command.

Found a [post about this](https://stackoverflow.com/questions/34243229/in-ansible-is-it-possible-to-define-the-authentication-method-per-playbook)

The scripts are now using a method in the following link to generate the encrypted password, specifically
the [mkpasswd command mentioned here](http://docs.ansible.com/ansible/latest/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module) 
                      