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