# 2024-09-15_VirtualNetwork

## Brainstorming and configuring virtual infrastructure

I wanted to build a fully fledged testing environment. I have been treating my "prod" home network as a testing environment, and what drives me insane is that every time I want to set a DHCP reservation on my [RT-AX86U](https://www.asus.com/us/networking-iot-servers/wifi-routers/asus-gaming-routers/rt-ax86u/) router, it takes down the entire network for like 90 seconds. I'd rather just have a completely isolated environment to do whatever I like in without worrying about bricking my home network.

### What I achieved:
- I stood up a virtual network from scratch using [OpenWrt](https://github.com/openwrt/openwrt) and virtual switches (``vmbr0`` & ``vmbr1``) in my Proxmox environment. It is not yet connected to the internet.
- I added another Ubuntu VM, ``host1``, to simulate an end host. This is connected to ``vmbr1``, therefore is not connected to the internet.

### I did this through:
- Creating and configuring a virtual router by hosting an OpenWrt virtual machine.
- Configuring and connecting virtual interfaces and switches in Proxmox (``vmbr``).
- Creating a new virtual machine to simulate an end host.

### A ten minute rudimentary diagram to help visualize: 

![Diagram](/Entries/2024-09-15_VirtualNetwork/rudimentary.png)

<sup><sub>This is not nearly my best work. It is simply to help visualize what I'm talking about here.</sub></sup>

## Configuring the virtual network

The first thing I needed to to was create a new virtual switch in Proxmox, and this was done easily enough through the web UI. I created ``vmbr1`` and set its IP to 192.168.5.2, with the plan being for the new network to be 192.168.5.0/24.

I followed along with [this guide](https://i12bretro.github.io/tutorials/0405.html) from i12bretro, and with some modification, I was able to get the OpenWrt VM spun up.

The download command was broken, so I had to manually find the correct OpenWrt version and entered:

``wget -O openwrt.img.gz https://archive.openwrt.org/releases/23.05.3/targets/x86/64/openwrt-23.05.3-x86-64-generic-ext4-combined.img.gz``

I repeated this process several times because I kept screwing up the config on the VM.

### Troubleshooting

First off, the new Ubuntu VM couldn't grab an IP because obviously the router (OpenWrt) wasn't set up yet. This should've been easy enough considering I have statically configured addresses in ``/etc/network/interfaces`` several times, but I learned there was another place this was done, and it was in **YAML**. 

I entered ``sudo nano /etc/netplan/01-network-manager-all.yaml`` and I configured it as followed:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens18:
      dhcp4: no
      addresses: 
        - 192.168.5.12/24
      gateway4: 192.168.5.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```
After misplacing multiple spaces and breaking the config several times (tested with ``sudo netplan try``), I was finally able to see the IP address configuration update for ``ens18`` after restarting the network service. Obviously ``nano`` is not the greatest for YAML.

After that, ``host1`` could ping ``vmbr1``, so at least I was able to confirm nothing was wrong with the switch config.

However, ``openwrt`` wasn't getting replies from anything besides its loopback.

OpenWrt only has ``vi`` installed as the default text editor. After fun times editing ``/etc/network/config`` and not getting anywhere, I recreated and remounted the virtual disk.

The original goal was for the virtual router to, you know, *route*. Including to the "WAN", which in this case would just be my local network. The problem is that OpenWrt is intended to replace your home router and run on physical hardware, and throwing virtual switches into the mix just made things more confusing.

After much troubleshooting, the issue was that something strange happens when booting the VM for the first time with both virtual switches connected to it. *I think* what's happening is that it just kept configuring the connection going to ``vmbr0`` as 192.168.5.1 instead of ``vmbr1``, and to be honest I didn't have the heart to dig through and find out how to fix this tonight. I also think this will be considerably easier to configure in the web UI and not the console.

I disconnected ``vmbr0`` from the VM, detached/removed/reconfigured the virtual disk, entered ``uci set network.lan.ipaddr='192.168.5.1'`` for the umpteenth time, restarted the network service, and I was able to ping ``vmbr1`` from ``openwrt``.

I confirmed by accessing the web UI from ``host1``. And this is where tonight ends, to be continued another day.

