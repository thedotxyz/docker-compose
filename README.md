# Running Unbound + Pi-hole (2 Containers - Bridge Network)

<h2>Scenario description</h2>
In this scenario we will use docker-compose to install two docker containers in a bridge network. It is the combination of a DNS sinkhole (pi-hole) to block unwanted content on your devices on a netwerk level (DNS based). This can be unwanted adverts or trackers. You as user has influence on the used blocklists. Blocklist are covered in this readme file.</br>
Allthough there are several privacy aware upstream DNS servers (like Cloudflare's <a href="https://www.cloudflare.com/learning/dns/what-is-1.1.1.1/">1.1.1.1 </a>) we don't want to use an external iterative DNS server. Instead of 


<h2>Graphical overview</h2>
XX

<h2>Unbound (Recursive DNS server</h2>
<img title="a title" alt="Alt text" src="Unbound_FC_Shaded_cropped.svg"></br>
Unbound is a validating, recursive, caching DNS resolver created by NLNET Labs. It is designed to be fast and lean and incorporates modern features based on open standards. Late 2019, Unbound has been rigorously audited, which means that the code base is more resilient than ever.</br>
To help increase online privacy, Unbound supports DNS-over-TLS and DNS-over-HTTPS which allows clients to encrypt their communication. In addition, it supports various modern standards that limit the amount of data exchanged with authoritative servers. These standards do not only improve privacy but also help making the DNS more robust. The most important are Query Name Minimisation, the Aggressive Use of DNSSEC-Validated Cache and support for authority zones, which can be used to load a copy of the root zone.</br>
Unbound is free, open source software under the BSD license.</br>

<h2>Pi-Hole (DNS Sinkhole)</h2>
<img title="a title" alt="Alt text" src="pihole.png"></br>
The Pi-hole® is a DNS sinkhole that protects all of the connected devices (for instance in your own LAN) from unwanted content without installing any client-side software.</br>



<h2>Prerequisites</h2>
XX

<h2>Installing</h2>
XX

<h2>Governance</h2>
XX

<h2>Updating</h2>
XX
<h2>Sources</h2>

https://nlnetlabs.nl/projects/unbound/about/
https://www.xfelix.com/2020/09/pihole-unbound-docker-setup-on-raspberry-pi/
