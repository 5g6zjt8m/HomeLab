# Entry Summaries

Select the dated folders located in this directory to view the contents of them. There will be readme files in each entry folder, with summaries located here.

## [2024-05-02_Prefacing](/Entries/2024-05-02_Prefacing/)

This is giving context to all future entries as to where my homelab sits as of 5/2/2024.

Around eight months ago, I embarked on upgrading my homelab setup, moving from a basic ISP router to a more configurable network and a modular workstation. I switched to a new router, got a Lenovo laptop, and set up a Synology NAS. After transitioning my old desktop into a virtualization server, I encountered issues with Windows on the new laptop, leading me to switch to Ubuntu. However, I later realized Ubuntu couldn't run Fusion 360 for my CAD projects. I decided to set up a dual boot system but faced networking issues with the hypervisor during the process. My plan now is to back up VMs, sell the current server, and build a purpose-built virtualization server.

## [2024-05-18_NewHost](/Entries/2024-05-18_NewHost/)

I began by prepping my old gaming computer for sale after a friend offered to buy it. The main task was exporting a crucial VM I used for 24/7 seeding to my NAS. I removed the 2TB SSD I wanted to keep and did a fresh Windows install on the remaining 500GB SSD. After selling the machine, I was left with the 2TB SSD and a VM backup.

With my laptop's multiple M.2 slots, I decided to set up a dual-boot system. I installed the 2TB SSD for Windows while keeping the 1TB SSD for Ubuntu. I ran into issues with Windows overwriting the bootloader and a failed attempt at using Veracrypt for encryption. Eventually, I successfully used Bitlocker, resulting in a working dual-boot setup.

I then bought a barebones Minisforum MS-01 to host Proxmox, attracted by its impressive I/O features. I planned to use the 1TB SSD from my laptop and partition the 2TB drive for dual OS use. After cloning the Ubuntu installation, I faced Bitlocker encryption issues but resolved them by exporting the recovery key.

Finally, I installed Proxmox on the MS-01 using PiKVM, configured the network interfaces, and recovered the VM from my NAS. With Proxmox up and running, I successfully set up the new system and achieved a functional dual-boot and Proxmox hosting environment.

## [2024-06-01_DNS](/Entries/2024-06-01_DNS/)

I heard great things about the Pihole project, which involves setting up your own DNS server to block advertisements by associating ad domains with invalid IP addresses. I configured Pihole to forward legitimate DNS requests to Quad 9 and set it as the default DNS server on my DHCP server, effectively blocking most ads on my network.

To set up Pihole in Proxmox, I spun up a new VM using Ubuntu, allocated 1024MB of RAM, 8GB of storage, a single CPU core, and installed the OS. After reserving a DHCP address for the server, I installed curl and then Pihole, enabling the web GUI. However, running a full OS for this felt wasteful, especially since Ubuntu used 90% of the RAM.

I tried DietPi to save RAM but encountered issues. Frustrated, I decided to try containers (LXC) in Proxmox, which are lighter than VMs. After some trial and error, I successfully created a container with an Ubuntu template, allocated 512MB of RAM, one core, and 8GB of storage. Installing Pihole in the container was straightforward, and I managed to save 512MB of RAM without unnecessary OS bloat.

## [2024-07-20_Recovering](/Entries/2024-07-20_Recovering/)

I had an issue with my seeding virtual machine, which had been working well until it suddenly stopped seeding. When the VM wouldn't boot, I saw an error indicating that GNOME Display Manager couldn't start.

To troubleshoot, I booted from an Ubuntu live image and quickly realized the virtual disk was completely full, preventing the desktop environment from loading. I figured out that a power loss had caused the NAS to reboot and remain encrypted. Automated downloads from qBittorrent filled up the VM's local storage instead of the NAS because the NAS wasn’t mounted.

To fix this, I took ownership of the directory using ``sudo chmod 777 -R /path/to/folder``, deleted the downloaded files, and rebooted the VM. It then booted correctly and resumed its normal operations.

I need to find a way to detect if the NAS is mounted to prevent such issues in the future, possibly using permissions. For now, I’ve disabled automatic downloading since it nearly filled up the 8TB NAS too. I also plan to automate VM backups.

## [2024-09-15_VirtualNetwork](/Entries/2024-09-15_VirtualNetwork/)

I wanted to create a dedicated testing environment, as I've been using my home network for experiments, and every time I set a DHCP reservation on my RT-AX86U router, it disrupts the entire network for about 90 seconds. To avoid this, I decided to build a completely isolated virtual network in my Proxmox environment.

- I created a virtual router using an OpenWrt VM.
- Configured and connected virtual interfaces and switches.
- Created a new VM to act as an end host.

After much troubleshooting, I'm leaving off tonight by having an accessible web UI for the router.

## [2024-09-28_OpenWRT](/Entries/2024-09-28_OpenWrt/)

I connected my testing LAN (192.168.5.0/24) to my home LAN (10.2.50.0/24), allowing it to access the internet. To do this, I manually configured the br-lan bridge in OpenWrt from the command line to include only eth1, which let me access the web UI from host1. I also set the router interfaces with correct addresses, gateways, and broadcast addresses.

When I couldn't access the web GUI after rebooting with both virtual switches (vmbr0 and vmbr1) attached, I realized br-lan had reset itself to 192.168.0.1. I fixed it by running uci commands to restore the correct IP, 192.168.5.1. I then discovered br-lan was assigned to eth0 instead of eth1 and adjusted it accordingly, which solved the access issue.

Once I regained access to the web UI, I created a WAN interface using eth0 and set the correct address and gateway. I also reconfigured the LAN interface for eth1 and adjusted some firewall zone settings to ensure traffic could flow between 192.168.5.0/24 and 10.2.50.0/24. As a result, my testing LAN now has a proper route to the home LAN and internet connectivity.
