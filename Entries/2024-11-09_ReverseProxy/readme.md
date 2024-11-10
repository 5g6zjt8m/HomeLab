# 2024-11-09_ReverseProxy

### What I achieved:
- I configured a secure and scalable reverse proxy solution for my home network. This is intended for remote use of my [GNS3](https://www.gns3.com/) server without a VPN client, but I do anticipate scaling for use with other services.

### I did this through:
- Importing a GNS3 VM and configuring it for use.
- Creating a [Nginx](https://nginx.org/) LXC container in my Proxmox environment and reserving its IP.
- Registering a subdomain with a [dynamic DNS provider](https://www.duckdns.org/why.jsp) and creating a [cron](https://en.wikipedia.org/wiki/Cron) job to keep the DNS entry up to date.
- Forwarding ports 80 and 443.
- Configuring Nginx to require an SSL certificate for incoming connections over the internet.

## Creating an Nginx LXC container

User [ttek](https://github.com/tteck) has an excellent repository of scripts for Proxmox. I used their [Nginx Proxy Manager LXC script](https://github.com/tteck/Proxmox/raw/main/ct/nginxproxymanager.sh) to very easily create and launch the container. I ran this in the shell of my PVE:

``bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/nginxproxymanager.sh)"``

I loaded the web GUI, set the credentials, and it was smooth sailing from there. Importantly, I reserved the IP address that was leased to the VM.

## Dynamic DNS

DuckDNS describes the concept of dynamic DNS very well:

```
most connections to the internet are through a dynamic external IP address which changes quite often (weekly or even daily).
this can make it very difficult to connect to home services from an external computer.

to get around this, Duck DNS is a provider of what is known as a DDNS (Dynamic DNS) service
we provide a public DNS server that anyone can get a subdomain and use one of our provided scripts to update their record(s).
so instead of trying to remember an IP address, you can use a domain name that is kept up-to-date by a computer at home

once this is done, periodically (usually every 5 minutes), the computer running the client, tells our central system (via a HTTPS post), to update the record with its latest external IP
```

Essentially, all I needed to do was register a subdomain with them and then use a script to make HTTP POST requests that update the DNS entry. When the POST request is sent from my home network, DuckDNS just updates the DNS entry for the subdomain with the source address of the request.

## Configuring Nginx

The GUI is well designed and intuitive. I filled in the domain name field with my registered subdomain, and then set the forward IP to my GNS3 server. I left the protocol as HTTP as that is what the GNS3 server supports by default. I had attempted to configure GNS3 to use an SSL certificate for accessing its web GUI, but was unsuccessful.

![ProxyHost](/Entries/2024-11-09_ReverseProxy/ProxyHost.png)

Initially, I had neglected to configure an SSL certificate for this proxy setup. I forwarded ports 80 and 443 on my router, and when going to log in, I was graciously met with the following message:

![Unencrypted](/Entries/2024-11-09_ReverseProxy/Unencrypted.png)

This immediately alerted me that I needed to set up the SSL certificate. However, it also was a good demonstration as for why HTTPS is so important here. I loaded up Wireshark and captured this:

![HTTP](/Entries/2024-11-09_ReverseProxy/HTTP.png)

Very easy to spot the ``Authorization`` portion of the HTTP header with the ``Credentials`` plaintext.

Fortunately, Nginx makes it incredibly easy to configure and force an SSL certificate for incoming connections. It's literally a matter of checking a few boxes, and it is complete.

## Security implications

Opening up more ports on the network is never a good feeling, but I feel confident in this configuration and that everything will be fine. The only ports I have forwarded are for HTTP, HTTPS, and OpenVPN.

All traffic hitting the reverse proxy is forced to use HTTPS. There is no sensitive data being passed in the clear in this hop. Traffic on my LAN is in the clear, and I'm ok with that for now for the purposes of this project. There is nothing on my network being passed plaintext that is worth anything or will lead anywhere. The only way someone is going to capture those plaintext packets is by cracking OpenVPN or physically connecting to my LAN.

![Topology](/Entries/2024-11-09_ReverseProxy/Topology.png)

After a couple hours of troubleshooting, I was not able to configure logging to work correctly. The shell script for creating the container works great, but I just don't know enough about how it works and how to solve some key issues. It isn't just a simple install of Nginx, it's also using OpenResty that screws with where conf and log files are located and I believe there are permissions issues

Ideally, I'd like to have a centralized logging server where syslog messages are forwarded. I'm particularly interested in the access logs for the reverse proxy. With this now being public, I'd be interesting in seeing what IPs are attempting to access it and brute force the credentials (you can't). I will likely need to build a new container or VM to get these logs working right.