# Entry Summaries

Select the dated folders located in this directory to view the contents of them. There will be readme files in each entry folder, with summaries located here.

## [2024-5-02_Prefacing](/Entries/2024-5-02_Prefacing/)

This is giving context to all future entries as to where my homelab sits as of 5/2/2024.

Around eight months ago, I embarked on upgrading my homelab setup, moving from a basic ISP router to a more configurable network and a modular workstation. I switched to a new router, got a Lenovo laptop, and set up a Synology NAS. After transitioning my old desktop into a virtualization server, I encountered issues with Windows on the new laptop, leading me to switch to Ubuntu. However, I later realized Ubuntu couldn't run Fusion 360 for my CAD projects. I decided to set up a dual boot system but faced networking issues with the hypervisor during the process. My plan now is to back up VMs, sell the current server, and build a purpose-built virtualization server.

## [2024-5-18_NewHost](/Entries/2024-5-18_NewHost/)

I began by prepping my old gaming computer for sale after a friend offered to buy it. The main task was exporting a crucial VM I used for 24/7 seeding to my NAS. I removed the 2TB SSD I wanted to keep and did a fresh Windows install on the remaining 500GB SSD. After selling the machine, I was left with the 2TB SSD and a VM backup.

With my laptop's multiple M.2 slots, I decided to set up a dual-boot system. I installed the 2TB SSD for Windows while keeping the 1TB SSD for Ubuntu. I ran into issues with Windows overwriting the bootloader and a failed attempt at using Veracrypt for encryption. Eventually, I successfully used Bitlocker, resulting in a working dual-boot setup.

I then bought a barebones Minisforum MS-01 to host Proxmox, attracted by its impressive I/O features. I planned to use the 1TB SSD from my laptop and partition the 2TB drive for dual OS use. After cloning the Ubuntu installation, I faced Bitlocker encryption issues but resolved them by exporting the recovery key.

Finally, I installed Proxmox on the MS-01 using PiKVM, configured the network interfaces, and recovered the VM from my NAS. With Proxmox up and running, I successfully set up the new system and achieved a functional dual-boot and Proxmox hosting environment.
