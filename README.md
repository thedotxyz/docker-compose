#UNDER CONSTRUCTION WILL CHANGE
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
mkdir ~/docker_data
mkdir ~/docker_data/pihole-unbound
mkdir ~/docker_data/pihole-unbound/pihole
mkdir ~/docker_data/pihole-unbound/pihole/etc-dnsmasq.d
mkdir ~/docker_data/pihole-unbound/pihole/etc-pihole
mkdir ~/docker_data/pihole-unbound/unbound
```

<h2>Unbound Prerequisites</h2>
To prepare the configuration of unbound we start with creating two config files (<code>a-records.conf</code> (mandatory), <code>unbound.conf</code>) (optional) and downloading DNS <code>root.hints</code> from Internic.</br>
The next steps will cover the creation of the two configuraiton files. I've included the two seperate files and directory structure in this Github repo.

<h3>Configuration of local DNS records</h3>
At first we create and edit the <code>/docker_data/pihole-unbound/unbound/a-records.conf</code> file. This file is mandatory as we mapped the volume of the unbound docker image to another location. 

```console
touch ~/docker_data/pihole-unbound/unbound/a-records.conf
nano ~/docker_data/pihole-unbound/unbound/a-records.conf
```
You can leave it empty or put your local custom records if you need. Below is an example of the syntax for local rcords.

```conf
# A Record
#local-data: "somecomputer.local. A 192.168.1.1"
 
# PTR Record
#local-data-ptr: "192.168.1.1 somecomputer.local."
```
Save and exit with <code>CTRL+O</code>, <code>CTRL+X</code>.

<h3>Main configuration of Unbound (unbound.conf)</h3>
Next we can create and modify the main configuration of Unbound. Unbound will automatically generate a default one if you not create it. To create and edit the configuration file:

```console
touch ~/docker_data/pihole-unbound/unbound/unbound.conf
nano ~/docker_data/pihole-unbound/unbound/unbound.conf
```
If you want to define the config file yourself you can use the next example. Make modifications if needed.

```conf
server:
# If no logfile is specified, syslog is used
# logfile: "/var/log/unbound/unbound.log"
verbosity: 0
 
access-control: 172.16.0.0/12 allow
access-control: 127.0.0.0/8 allow
access-control: 10.0.0.0/8 allow
access-control: 192.168.0.0/16 allow
interface: 0.0.0.0
port: 5053
do-ip4: yes
do-udp: yes
do-tcp: yes
 
# May be set to yes if you have IPv6 connectivity
do-ip6: no
 
# You want to leave this to no unless you have *native* IPv6. With 6to4 and
# Terredo tunnels your web browser should favor IPv4 for the same reasons
prefer-ip6: no
 
# Use this only when you downloaded the list of primary root servers!
# If you use the default dns-root-data package, unbound will find it automatically
# I have to quote out this root-hints, as it causing container endless restarting for a new installation. You can add root-hints back after first run. 
#root-hints: “/opt/unbound/etc/unbound/root.hints”
 
# Trust glue only if it is within the server's authority
harden-glue: yes
 
# Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
harden-dnssec-stripped: yes
 
# Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
# see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
use-caps-for-id: no
 
# Reduce EDNS reassembly buffer size.
# Suggested by the unbound man page to reduce fragmentation reassembly problems
edns-buffer-size: 1472
 
# Perform prefetching of close to expired message cache entries
# This only applies to domains that have been frequently queried
prefetch: yes
 
# One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
num-threads: 1
 
# Ensure kernel buffer is large enough to not lose messages in traffic spikes
so-rcvbuf: 1m
 
# Ensure privacy of local IP ranges
private-address: 192.168.0.0/16
private-address: 169.254.0.0/16
private-address: 172.16.0.0/12
private-address: 10.0.0.0/8
private-address: fd00::/8
private-address: fe80::/10 
```
Save and exit with <code>CTRL+O</code>, <code>CTRL+X</code>.

<h3>DNS Root Hints</h3>
If you don't want to use the default DNS 'root hints' in the unbound container (otherwise you should have unquoted the 'root-hints' part of the config file'), you should download the most recent '<code>root.hints</code>' file to your unbound data folder (<code>/docker_data/pihole-unbound/unbound</code>). You do this by:

```console
cd ~/pihole-unbound
wget -O root.hints https://www.internic.net/domain/named.root
cd
```

<h2>Installing</h2>
After installing Git you clone this repository to your local server by:

```console
git clone https://github.com/thedotxyz/pihole-unbound
```

Next edit the docker-compose file to modify specific settings to your liking. At least change the 'Webpassword' for pihole or any of the other variables like ip, ports or volumes.

```console
nano ~/pihole-unbound/docker-compose.yaml
```
Save and exit with <code>CTRL+O</code>, <code>CTRL+X</code>.

<h2>Running the containers</h2>

Let’s start unbound container first:
```console
cd ~/docker_data/pihole-unbound/unbound
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

https://nlnetlabs.nl/projects/unbound/about/</br>
https://www.xfelix.com/2020/09/pihole-unbound-docker-setup-on-raspberry-pi/</br>
