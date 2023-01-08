# Running Unbound + Pi-hole (2 Containers - Bridge Network)

<h2>Scenario description</h2>
In this scenario we will use docker-compose to install two docker containers in a bridge network. It is the combination of a DNS sinkhole (pi-hole) to block unwanted content on your devices on a netwerk level (DNS based). This can be unwanted adverts or trackers. You as user has influence on the used blocklists. Blocklist are covered in this readme file.</br>
Allthough there are several privacy aware upstream DNS servers (like Cloudflare's <a href="https://www.cloudflare.com/learning/dns/what-is-1.1.1.1/">1.1.1.1 </a>) we don't want to use an external iterative DNS server. In this scenario we use our own recursive dns server (Unbound) to resolve all of the DNS queries of our devices using our own controlled services.

<h2>Context: Unbound (Recursive DNS server</h2>
<img title="a title" alt="Alt text" src="Unbound_FC_Shaded_cropped.svg"></br>
Unbound is a validating, recursive, caching DNS resolver created by NLNET Labs. It is designed to be fast and lean and incorporates modern features based on open standards. Late 2019, Unbound has been rigorously audited, which means that the code base is more resilient than ever.</br>
To help increase online privacy, Unbound supports DNS-over-TLS and DNS-over-HTTPS which allows clients to encrypt their communication. In addition, it supports various modern standards that limit the amount of data exchanged with authoritative servers. These standards do not only improve privacy but also help making the DNS more robust. The most important are Query Name Minimisation, the Aggressive Use of DNSSEC-Validated Cache and support for authority zones, which can be used to load a copy of the root zone.</br>
Unbound is free, open source software under the BSD license.</br>

<h2>Context: Pi-Hole (DNS Sinkhole)</h2>
<img title="a title" alt="Alt text" src="pihole.png"></br>
The Pi-hole® is a DNS sinkhole that protects all of the connected devices (for instance in your own LAN) from unwanted content without installing any client-side software.</br>

<h2>Graphical overview</h2>
XX

<h2>Details</h2>
XX

<h2>Server Prerequisites</h2>
To run this in a home-lab scenario you've got to perform some prerequisite steps. This combination of services runs perfectly fine on a Raspberry Pi. Recommended steps to perform are:

<ol>
    <li>Install Raspbian / Raspberry Pi</li>
    <li>Make sure you're server has got a static IP address</li>
    <li>Configure Automatic Updates</li>
    <li>Configure Fail2Ban</li>
    <li>Install Git</li>
    <li>Install Docker</li>
    <li>Install Docker-Compose</li>
    <li>Configure Firewall</li>
</ol>

>It's best to enable the firewall (UFW) after installing and running Unbound & PiHole.

<h2>File and folder structere Prerequisites</h2>
Login on the server with the account you will use to install services (sudo permissions required). Make the folder structure for the docker-files.

```console
mkdir ~/docker_data/pihole-unbound
mkdir ~/docker_data/pihole-unbound/etc-dnsmasq.d
mkdir ~/docker_data/pihole-unbound/etc-pihole
mkdir ~/docker_data/pihole-unbound/unbound
```

<h2>Unbound Prerequisites</h2>
XX

<h2>Installing</h2>
After installing Git you clone this repository to your local server by:

```console
git clone https://github.com/thedotxyz/pihole-unbound
```

Next edit the docker-compose file to modify specific settings to your liking. At least change the 'Webpassword' for pihole or any of the other variables like ip, ports or volumes.

```console
sudo nano <PATH>\docker-compose.yaml
```
Save and exit with <code>CTRL+O</code>, <code>CTRL+X</code>.

<h2>Running the containers</h2>

Let’s start unbound container first:
```console
sudo docker-compose up -d unbound
```

To get access to testing utilities (nslookup and dig) install dnsutils:
```console
sudo apt-get install dnsutils
```

Test if unbound is working as expected:
```console
dig www.google.com @172.20.0.7 -p 5053
dig www.google.com @127.0.0.1 -p 5053
```
If all return normal results then unbound is up running.</br>

Then start pihole container.

```console
sudo docker-compose up -d pihole
```

<h2>Governance</h2>
After succesfully starting both of the containers we can try to access the Pi-hole management interface to configure the desired Pi-hole blocklist by accessing http://your-Raspberry-pi-ip/admin/index.php.



<h2>Populate Pi-hole</h2>
XX

<h2>Updating</h2>
Because we have seperated the configuration and data files for Unbound en Pi-hole in our volume configuration, updating docker containers is pretty easy. We perform this by downloading the new container versions, stopping the current containers and removing them and then create and start new updated containers. This can be done by:

```console
sudo docker pull pihole/pihole:latest
sudo docker pull mvance/unbound-rpi:latest
sudo docker stop pihole
sudo docker stop unbound
sudo docker rm pihole
sudo docker rm unbound
sudo docker-compose up -d unbound
sudo docker-compose up -d pihole
```

<h2>Sources</h2>

https://nlnetlabs.nl/projects/unbound/about/
https://www.xfelix.com/2020/09/pihole-unbound-docker-setup-on-raspberry-pi/
