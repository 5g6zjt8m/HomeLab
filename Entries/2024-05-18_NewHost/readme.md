# 2024-05-18_NewHost

## Prepping for sale

A friend of mine offered to buy the old gaming computer from me. There was a little bit of preparation I needed to do to get it ready for a new owner. There is really only one VM I cared about, which was the one I used for 24/7 seeding. There was quite a bit of configuration setting up that VM. Stuff like making it only connect to external networks using a VPN, getting some folders mounted through configuring ``/etc/fstab`` (and breaking it multiple times), ufw rules, QBittorrent settings, having everything autolaunch properly. I'd rather not do it all over again if I didn't need to.

I exported the VM to my NAS, and powered off the machine.

The system had two SSDs in it, a 2TB M.2 storage drive and a 500GB M.2 boot drive. I removed the 2TB drive as I knew that I would probably be needing it for whatever came in the future. I gave the remaining 500GB SSD a fresh install of Windows, and preinstalled the drivers and RGB software.

I sold the machine, and all I was left with was a 2TB SSD and a cold VM backup.

## Dual booting

Knowing that my laptop had multiple M.2 slots, I decided it was finally time to get a dual boot going. It had been such a long time since I was able to edit stuff in CAD or use Adobe stuff. GPU passthrough to a locally hosted VM is not a realistic solution, and has a ton of issues. In my brilliance, I decided to just stick the 2TB SSD into the laptop and run Windows off of it. So at this point inside of the laptop, there's a 1TB SSD running Ubuntu, and a 2TB SSD getting ready to have Windows installed onto it. In hindsight, not a great setup and pretty wasteful. But also, the 2TB SSD would have just been sitting around had I not installed it.

I figured the rest of this would be fairly simple. Plug in a Windows install USB, install it to the new drive, and it'd be all done. This was pretty much the case until realizing that Windows will overwrite any bootloader as it assumes it is the only OS installed. This means GRUB was now broken, and I couldn't boot into Ubuntu.

Not too bad of a fix. Plugged in an Ubuntu install USB, ran it live and repaired GRUB using ``Boot-Repair``.

### Encryption

I had attempted to try full disk encryption with Veracrypt instead of Bitlocker, which was a disaster. Turns out Veracrypt also has its own bootloader, and stuff just got so messy and broken that I had to reinstall Windows (and repair GRUB again) since I literally could not decrypt the volume. Thankfully my Ubuntu install was spared since it was on a separate drive.

I encrypted using Bitlocker instead and this was a much better solution. I now had a good dual boot running so that I can occasionally boot into Windows for certain projects.

## The new host

I was searching for a new system to host Proxmox. I wanted to be able to host a decent streaming server using something like Plex or Jellyfin. I stumbled upon the [Minisforum MS-01](https://store.minisforum.com/products/minisforum-ms-01) and bought it barebones. An i9-13900H is a beast of a processor. More importantly, I thought the I/O was pretty crazy.

![MS01](/Entries/2024-05-18_NewHost/MS01.png)

Dual 2.5G LAN ports and dual 10Gbps SFP+ ports is huge. It opens the door for so many projects in the future, maybe speeding up data transfer between the NAS and VM hosts.

It came in rather quickly considering it shipped from Hong Kong, less than a week.

### Repurposing hardware

Since it was barebones, I planned to use the 1TB SSD in my laptop that was running Ubuntu. I was going to partition the 2TB drive so that each OS would have 1TB of storage using that single drive. I also ordered a 32GB kit of SODIMM DDR5 RAM to install in it.

Onto cloning the Ubuntu drive. I booted into Windows and shrunk the C:\ volume to 1TB using the ``diskmgmt`` GUI. I then booted into Ubuntu and shrunk that volume to 250GB using ``GParted``. There is some rounding and funky stuff that happens when shrinking/cloning volumes, so I'd just figured to undershoot and extend the volume later. Cloned the two Ubuntu partitions to the new unallocated space on the 2TB drive using [Macrium Reflect](https://www.macrium.com/). Easy stuff.

I took apart the laptop and removed the 1TB SSD. Booted it back up and GRUB loaded fine, so I booted into Ubuntu. Everything looked good there. This looked like a success until I booted into the Windows partition.

I was met with this screen.

![Bitlocker](/Entries/2024-05-18_NewHost/Bitlocker.png)

Not a good sign. I didn't know that this was a function of Bitlocker. I figured it was like any other encryption where your password is the key and that's it. I assumed the "recovery key" was something that's used for when someone forgot their key, and it was just some huge long string that would decrypt the drive. Of course I didn't want that file laying around somewhere, so I never saved it.

Knowing now, that is not how Bitlocker works.

I stuck the 1TB drive back in and thankfully was able to boot into Windows using the bootloader on that drive. I'm not sure why Bitlocker freaked out when it was basically untouched besides a shrunken volume and a couple new partitions. I think it's related to the bootloader changing, but I'm not positive. I was able to export the key, remove the drive again, boot back into Windows and enter the recovery key.

Now the dual boot was complete, using a single drive. Now the 1TB drive was free to go into the MS01. My RAM also had not come in yet, so I used the old 16GB DDR5 kit that came in the laptop and stuck those in the MS01 too. It's an easy upgrade when the RAM comes in, and I just wanted to get the system up and running.

### Configuring the new system

I plugged the PiKVM into it, chose the Proxmox .iso to mount from the KVM, and remotely installed Proxmox. Simple setup, and I made sure to make a new DHCP reservation for the new MAC address. The install was easy.

When Proxmox boots, it launches a web GUI that can be accessed at the configured IP address. This was not working after the install. Running into this before, I knew to check ``/etc/network/interfaces`` and see what was going on. I was met with this:

```
auto lo
iface lo inet loopback

iface enp87s0 inet manual

auto vmbr0
iface vmbr0 inet static
    address 10.2.50.201/24
    gateway 10.2.50.1
    bridge-ports enp87s0
    bridge-stp off
    bridge-fd 0

iface enp88s0 inet manual

iface enp2s0f0 inet manual

iface enp2s0f1 inet manual

iface wlp89s0 inet manual
```

I figured the best option was to just substitute different interfaces into the ``bridge-ports`` section. I didn't know which was which. I subsituted all of them, and on the last attempt, ``enp88s0`` was the correct one. I was now able to load into the web GUI and get out of the KVM.

With Proxmox now loaded up and working, I was able to mount the storage folder on my NAS and recover the VM that was stored in there. It was so easy. I booted up the VM and everything worked perfectly.