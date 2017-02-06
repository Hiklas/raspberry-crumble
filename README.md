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



