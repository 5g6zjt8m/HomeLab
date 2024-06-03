# 2024-05-02-Prefacing
## The past
### Prefacing
About eight months ago, I wanted to dive a little deeper into homelabbing than what I had going on at the time. My setup
consisted of the standard ISP issued router, with all devices on the network connected wirelessly via DHCP. My main workstation
was a Windows desktop that I had built about a year and a half prior, spec sheet and photos attached here.

![ComputerSpecs](/Entries/2024-05-02_Prefacing/ComputerSpecs.png)

![ComputerBuild](/Entries/2024-05-02_Prefacing/ComputerBuild.png)

This was not much of a complicated setup. Networking settings were untouched from stock except maybe a couple of forwarded ports. My workstation hosted a VM for seeding, but often this VM was powered off due to how loud the computer was, and the lack of sleep I was getting due to some of the RGB components not easily being disabled, mainly on the motherboard.

It was time for a change. I wanted a more configurable networking setup and a more modular workstation setup. I bought a new router ([RT-AX86U](https://www.asus.com/us/networking-iot-servers/wifi-routers/asus-gaming-routers/rt-ax86u/)) and managed to snag a very good deal on an open-box Lenovo laptop ([16IRX8H](https://psref.lenovo.com/Product/Legion/Legion_Pro_7_16IRX8)) locally. I even swapped in 32GB of DDR5 RAM. I found another great deal on an open-box Dell docking station  ([WD22TB4](https://www.dell.com/en-us/shop/dell-thunderbolt-dock-wd22tb4/apd/210-bdqh/docks)). This setup would be very modular, with  the option to plug in any laptop via thunderbolt and have a functioning workstation.

I also finally purchased a Synology NAS ([DS224+](https://www.synology.com/en-us/products/DS224+)). I threw in two 8TB hard drives and have them running in a RAID 1.

### Converting the old desktop
I copied data that I did not want to lose from the 2TB SSD to the NAS, and then created a virtual disk with the 500GB SSD being
used as my boot drive. I moved the desktop next to the router in the other room and could finally plug it in with an ethernet
cable, and could now get a full night's sleep. I also plugged in my custom built [PiKVM](https://docs.pikvm.org/). More details on this coming in a future entry, but basically imagine a lights-out/pre-OS server management solution like HP iLO/Dell iDRAC.

With this newfound desk space, I set up the laptop and docking station. I downloaded the .iso for [Proxmox](https://www.proxmox.com/en/), uploaded it to the PiKVM, and finished the entire installation process from the PiKVM web GUI. Good practice for remote OS installs. Once the Proxmox web server was running, I was home free to start virtualizing.

I created an Ubuntu VM to run as a seedbox. This was essentially trial by fire in forcing me to get better in the Linux terminal.
As I figured out how to permanently mount a folder from the NAS, I learned that a slight misconfig in ``/etc/fstab`` meant that you no longer boot into a desktop environment. In the end, the seedbox was up and running and worked well.

### The issues began
The seedbox VM worked great, with occasional hiccups caused by QBittorrent that were remedied through updates. The host system itself also ran great. Proxmox as the hypervisor never gave me issues, and was incredibly reliable.

Of course, the first thing to act up was Windows, docking station drivers, and some laptop-specific stuff.

The laptop came preinstalled with Windows 11, and that's what Nvidia 40 series GPUs are designed to run on. I found the telemetry phoning home to Microsoft to be particularly egregious, especially as pointed out by [this video](https://www.youtube.com/watch?v=VU9L0udNV9M). With this knowledge, I plugged in a bootable Windows 10 drive and downgraded. Networking drivers trackpad drivers, GPU drivers and many others did not work on a fresh install and needed to be downloaded manually. This ran fine for a while, but was not problem free.

I learned that my laptop plays a balancing act between using integrated graphics and the GPU. If there was no real graphical load being processed, the laptop would shut off the GPU to save power. This presents itself as flickering on external displays. Imagine the google-fu necessary in narrowing down this symptom, to where this is a specific BIOS setting in Lenovo laptops.

The docking station drivers themselves were also somewhat unreliable, and sometimes would refuse to update or cause other issues like failing to pass through USB devices plugged into the docking station. Sometimes the external display would not power on along with the laptop.

The final nail in the coffin was when I booted up the laptop and the external display did not power on. The monitor would only read "no input". I had tried every combination of rebooting the monitor and laptop, plugging directly into the laptop vs the docking station, displayport and HDMI, with nothing fixing it.

This was it. I was done using Windows. I figured it was an issue with the hack job required to get these Windows 11 drivers working in Windows 10. If I couldn't run Windows 10, then it was time to try something new.

### A new frontier
I installed Ubuntu and did not look back. Better yet, the network and trackpad drivers worked natively.

I was very careful using Ubuntu since I still was not very comfortable in Linux. I was able to get most of the programs I need except for obvious ones like Adobe stuff. I figured at some point in the future I'd get a GPU passthrough set up and get that working. Otherwise, pretty much any software that I needed worked  perfectly in Ubuntu.

This worked great for months.

## The present
### Time for change, again
Fusion 360 does not run on Linux. I never realized this until very recently. I still had yet to get a dual boot running, with that 2TB SSD still just sitting in the hypervisor host literally unused. I had planned on making some new VMs that might use up some of that storage, but was not a fan of that idea in the end. The 500GB SSD that the hypervisor was installed onto was already plenty, and realistically I did not have plans for a VM that would eat up a ton of local storage. I did consider
making a virtualized gaming server at one point and passing through the RTX 3080 in the host machine. However, anti-cheats can detect when a game is being run in a VM.

It was finally time to get the dual boot working. I haven't edited any photography projects or designed anything in CAD for about 8 months at this point, which are two fairly big hobbies of mine. I gracefully shut down the seedbox VM, and ran ``poweroff`` to gracefully shutdown the hypervisor host. I took off the side panel and removed the GPU to get to the 2TB SSD.

As I had the computer disassembled, side panel and GPU removed, I looked at how dusty the screen on the all in one liquid cooler was. I took in the size of the massive GPU that has sat literally entirely unused for 6 months. I remembered the process of installing the six 140mm fans and programming their lighting sequences. The disabled RGB modules on the RAM as well.

It reminded me that this was built as a gaming computer, and truly was never meant to act as a VM server. Sure it has enough CPU cores and RAM to be one, but that's not what I built it for. I decided that I am going to sell this computer and build a purpose-made virtualization server in a much better chassis. I just need to back up my seedbox VM, and an IRC bot VM that had been powered off for a while. I correctly reassembled the machine and put it back in place.

### The hypervisor's last gift
I booted up the machine, but I was unable to access the web GUI. No wonder, I had forgotten to plug in the ethernet cable. Rebooted and tried again, same issue.

I connected to the PiKVM to see what was going on, and was met with the regular boot screen for Proxmox stating that the web server was running on this IP address and port, and prompting me for a local login. Everything looked fine, but I still could not connect to that web UI.

I checked the web GUI for the router, and it stated that the connection over that switchport was up. Interestingly though, I could not see the device online. I remoted back into the PiKVM and did some investigating by trying to ping that default gateway, but was met with ``connect: Network is unreachable``. This made me more suspicious of something being wrong with the networking config locally on the machine. But how could that be? All I had done was remove the unused drive.

I ran ``ip address`` and sure enough, the only ethernet interface ``enp3s0`` was marked with ``qdisc noop state DOWN``. I googled how to enable the interface and tried ``ip link set enp3s0 up``. I restarted the networking service with ``service networking restart`` still to no avail, the web server was unreachable. I tried rebooting as well.

Not having configured ``/etc/network/interfaces`` before, I opened the file with ``nano`` and was met with an output that I did not quite recognize.

```
auto lo
iface lo inet loopback

iface enp5s0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 10.2.50.201/24
        gateway 10.2.50.1
        bridge-ports enp5s0
        bridge-stp off
        bridge-fd 0

iface wlo1 inet manual
```
<sub>(Writing this now, wow I wish I had noticed the name of the ethernet interfaces at this point. In fairness, I had never even seen ``systemd/udev`` interface names before.)</sub>

In hindsight, all I needed to do was adjust those interface names. But of course, I nuked this file and rewrote it to just this:

```
iface eth0 inet static
    address 10.2.50.201
    netmask 255.255.255.0
    network 10.2.50.0
    broadcast 10.2.50.255
    gateway 10.2.50.1
```

I then ran ``service networking restart`` again, ran ``ip address`` again, but still, ``enp3s0`` was marked with ``qdisc noop state DOWN``. I tried ``ip address add 10.2.50.201/24 dev enp3s0`` followed by ``ip link set enp3s0 up`` and ``ip route add default via 10.2.50.1``.

This finally worked. I was finally able to get into the web GUI, but now the VM will not start. I suspected that it was because I removed the ``vmbr0`` interface. VPN'd in from my phone, remoted into PiKVM, I rewrote ``/etc/network/interfaces`` to the following:

```
        auto lo
iface lo inet loopback

iface enp3s0 inet manual

auto vmbr0
iface vmbr0 inet static
address 10.2.50.201/24
gateway 10.2.50.1
bridge-ports enp3s0
bridge-stp off
bridge-fd 0

iface wlo1 inet manual
```

[Here is what that looks like on a 6 inch phone screen](Entries/2024-05-02_Prefacing/MobileTroubleshooting.png). What joy.

The future plan now is to back up the VMs, sell the current "server", and build out a new one.