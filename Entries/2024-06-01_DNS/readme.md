# 2024-06-01_DNS

## Home DNS server

I had heard great things about the [Pihole project](https://pi-hole.net/). Essentially, you're just standing up your own DNS server and then Pihole will make those DNS requests. The Pihole DNS server uses a configurable internal list of domains used for advertisements. It checks your DNS requests, and if the domain name matches on the blocklist, it will serve back an empty page. Pihole does this by just having those name records be associated with invalid IP addresses. The DNS record would look something like this:

``advertising.domain IN A 0.0.0.0``

Any DNS request for that domain will get nothing back, since the associated IP is invalid.

For valid domains that are not for advertising, Pihole is configured to forward those requests to the chosen name server. In my case, I arbitrarily chose quad 9 and it has worked fine.

The final step would then be setting Pihole as the default DNS server on your DHCP server, which then in effect will block most advertisements from your entire network.

### Setting up Pihole in Proxmox

This is very easy, or should I say, _can_ be very easy. I spun up a new VM using Ubuntu as my host OS, dedicated 1024MB of RAM, 8GB of storage, a single CPU core, and installed the OS. I created a DHCP reservation for the server, and installed Pihole.

I installed curl with ``sudo apt install curl``, and then installed Pihole with ``curl -sSL https://install.pi-hole.net | bash``.

That was it. Followed the prompts, selected Quad 9 as the DNS server for legitimate requests, and enabled the web GUI. I now had a functioning Pihole DNS server, but something didn't feel quite right. It felt pretty wasteful to have an entire desktop environment, along with running a whole host OS that is kind of known for being bloaty. Even then, I noticed that dedicating 1GB of RAM is less than the recommended minimum for Ubuntu, which I figured was _probably_ fine, but it was sitting at a solid 90% utilization.

To make a long story short, I spent way too much time screwing with other operating systems in an effort to save a measly 512MB of RAM. DietPi is a very lightweight flavor of Raspbian, and it forced me to learn how to import ``.qcow2`` disks into Proxmox and stand up a VM with it. That didn't work out though. Next I tried using an ``.iso`` version of a DietPi install, which still did not work. I was met with a black screen in the console every time the disk mounted and attempted to install the OS. I didn't have the heart to troubleshoot it any further, and I really hate going into the terminal of the hypervisor itself and playing with stuff that I don't understand yet.

It was time to try something even lighter.

### Diving into containers (LXC)

I had yet to utilize containers in any fashion and preferred using what I already knew, virtual machines. I knew that running Pihole in a container was much more preferable than standing up an entire VM. I've heard of [Docker](https://www.docker.com/) and [Portainer](https://www.portainer.io/) and will definitely give those a try at some point in the future, but Proxmox has really nice LXC integration and I figured that might be a little easier. In hindsight, setting up a Portainer VM for all of my docker needs would've probably been simpler for me.

First I had to figure out how to even create an LXC container. This involved downloading a template. Every place I read just suggested using a Debian template since that's what Pihole likes and was made for. However, every container that I tried to create just threw errors that again, I didn't have the heart to troubleshoot at the time and couldn't comprehend the solutions. Finally I ended up with a container successfully starting that used an Ubuntu template, allocating 512MB of RAM, one core, and 8GB of storage.

Once the container was stood up, I created a new DHCP reservation, installed curl, installed Pihole, and that was it. Very easy, and now there isn't a ton of unnecessary OS bloat. I had successfully saved... 512MB of RAM.