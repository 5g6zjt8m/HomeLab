# 2024-09-28_OpenWrt

This is a continuation of where I left off on this from last time, [2024-09-15_VirtualNetwork](https://github.com/5g6zjt8m/HomeLab/tree/main/Entries/2024-09-15_VirtualNetwork).

### What I achieved:
- I connected the testing LAN (192.168.5.0/24) to my home LAN (10.2.50.0/24), therefore the internet.

### I did this through:
- Manually configuring the OpenWrt bridge, ``br-lan``, from the command line to only include ``eth1``. This is the interface that the testing LAN switch ``vmbr1`` (192.168.5.2) is connected to. **In essence, this means that I was able to configure the rest of the settings via the web UI of ``openwrt`` from ``host1`` (192.168.5.12)**
- Manually configuring the router interfaces with addresses, default gateways, and broadcast addresses.

## Configuring and troubleshooting from the command line
After reaching the point where my testing LAN was completely segregated, I quickly realized that I needed a route to the internet to download updates, or anything for that matter. This meant that it was time to tackle the issue as to why I couldn't access the web GUI with both virtual switches, ``vmbr0`` and ``vmbr1`` attached to ``openwrt``. I attached ``vmbr1`` and rebooted the VM, and subsequently I could no longer access the web GUI.

I ran ``ip addr`` and saw that the ``br-lan`` interface had just reset itself to 192.168.0.1. I set this back to 192.168.5.1 by running:

```
uci set network.lan.ipaddr=192.168.5.1 
uci commit network 
/etc/init.d/network restart
```

This was only part of the problem. I was intrigued by the ``uci`` command, and found the following result very helpful:

![uci](/Entries/2024-09-28_OpenWrt/uci.png)

Using this info, next I ran ``uci show network`` and was met with the following:

```
network.@device[0]=device
network.@device[0].name='br-lan'
network.@device[0].type='bridge'
network.@device[0].ports='eth0'
```

From cobbling together the knowledge I gathered from the previous commands, this identified the issue. The following diagram makes this simple to comprehend, look at the red arrow.

![Incorrect](/Entries/2024-09-28_OpenWrt/Incorrect.png)

``br-lan`` is a virtual switch built into OpenWrt. Notice how ``eth1`` isn't connected to it.

All I needed to do was change the ``network.@device[0].ports=`` value to ``eth1``, and then ``eth1`` would be connected to that internal ``br-lan`` interface.

I issued the following commands, and I was able to access the web GUI from ``host1``.

```
uci set network.@device[0].ports='eth1'
uci commit network 
/etc/init.d/network restart
```

## Configuring and troubleshooting from the GUI

Now that I finally had access to the web UI for ``openwrt``, I could much more easily see what I was configuring.

I went into the interfaces tab and created a new interface named ``WAN``, and associated it with the "physical" ``eth0`` interface that is attached to ``vmbr0``. I addressed it as 10.2.50.215 (this should be changed later) and set the default gateway to my home router DG, 10.2.50.1.

I then finished out the configuration for the ``lan`` interface associated with ``br-lan`` and ``eth1``. I mistakenly set the DG to 10.2.50.1, and later corrected it to 10.2.50.215.

Once configured, ``host1`` (192.168.5.12) could ping the ``WAN`` interface (10.2.50.215). However, traffic was not getting out to 10.2.50.0/24. This was due to a firewall zone setting from within OpenWrt.

![Zone](/Entries/2024-09-28_OpenWrt/Zone.png)

**As a result, 192.168.5.0/24 has a route to 10.2.50.0/24 via 10.2.50.215 (eth0), and vice versa.**
