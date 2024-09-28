# 2024-07-20_Recovering

### What I achieved:
- Diagnosed and resolved a Linux VM failing to boot.
- Identified a flaw within automated processes causing the issue.

### I did this through:
- Understanding how to spot and research specific error codes.
- Booting into a live environment, taking ownership of the bloated directory with ``sudo chmod 777 -R /(path to folder)``, and deleting the errant files.

## Wait, why is nothing seeding?

My seeding VM had been pretty bulletproof for the past couple of months, and I've grown my upload total to almost six terabytes. Now that's a lot of Linux .isos. On rare occasions I'd notice that I had no active torrents on the tracker's site. Usually it was because of power loss (I don't have a UPS) and the NAS hadn't been manually decrypted yet. All I had to do in these occasions was decrypt it and reboot the VM.

However, this time, the VM wouldn't even boot. I was met with this:

![Error](/Entries/2024-07-20_Recovering/Welp.png)

My first thought here was that GNOME somehow got corrupted. One thing that's a little annoying about using a VM to host Ubuntu is that I can't get into GRUB and boot into recovery mode. What I did instead was attach an Ubuntu .iso into the virtual CD drive and boot off of that. I ran this as a live image so I could check out what was going on.

## Analyzing the failure

Once booted into the live environment, it became pretty clear what happened. From some basic troubleshooting with the ``Failed to start GNOME Display Manager`` error, I came across some threads that mentioned this could be due to a full disk. I knew that the virtual disk was pretty small since everything was actually being stored on the NAS, so I decided to check using [baobab](https://wiki.gnome.org/action/show/Apps/DiskUsageAnalyzer?action=show&redirect=Apps%2FBaobab), built into the live disk.

It was completely full. The disk was in fact so full, that it literally did not have space to load the desktop environment or GUI.

Luckily, it was actually pretty evident what had occurred here. This is a breakdown of what I imagine caused to happen issue:

1. Power loss caused NAS and VM server to reboot.
2. Seeding VM could not mount the NAS folder because it needed to be manually decrypted.
3. List of top 100 distros from Internet Linux .iso Database (ILDb) updated.
4. Automated processes and services kick off, automatically throwing those .torrent files into qBittorrent. *
5. qBittorrent downloaded to the same directory as always, but the NAS wasn't mounted.
6. qBittorrent created a new directory in place of where the mounted NAS should be.
7. Torrent files downloading completely filled up the local storage of the VM.

<sub>*This was actually a pretty big project, but I won't go into detail about it.</sub>


## Solving the problem

I had to take ownership over that directory since it was a live environment. I did this simply with ``sudo chmod 777 -R /(path to folder)``.

![Ownership](/Entries/2024-07-20_Recovering/Ownership.png)

At this point, I deleted the Linux .isos that had downloaded into local storage and rebooted. The VM proceeded to boot fine, start all of its services, and did what I configured it to.

The true solution to this would be some method to detect whether or not the NAS is actually mounted in the specified folder. I just need a way to block stuff from downloading where it shouldn't, perhaps using permissions. I will save this for another day. For now, I've disabled the automatic downloading since it also nearly filled up the 8TB NAS over the course of a couple months. I also cleaned out a ton of the automatically downloaded content since I had hit their minimum seeding time, and they were not performing as well as they were when they were first added to the list.

This also reminds me that I need to actually automate a backup process, that will back up snapshots of the VM every few days or so. I will also save this for another day. On a side note, last night I found a random group of guys on Discord that were all studying for their Network+/Security+/CCNA. I was up until about 1:00am reviewing content using practice tests with them. This also may explain my lack of motivation for immediately wanting solving this problem.