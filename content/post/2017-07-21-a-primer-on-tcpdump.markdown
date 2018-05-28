---
author: "Cody Wilbourn"
categories:
- Command Line
comments: true
date: "2017-07-21T22:34:55Z"
slug: a-primer-on-tcpdump
tags:
- cli
- tools
title: A Primer on TCPDump
---

tcpdump is a great utility for debugging network applications and profiling them. It’s near universal - some tcpdump variant exists on every platform. You can even run tcpdump on some higher end switches and routers. Packet dumps can be saved for later analysis or to be sent off to vendors for debugging.
<!--more-->

For a GUI variant, [Wireshark ](https://www.wireshark.org/)adds a nice layer on top.


## Expressions


tcpdump has its own syntax language for filtering that’s important to understand in order to avoid the firehose of network packets. This expression is after any command line switches.

The full syntax can be found in the pcap-filter(7) manpage.


### Useful filters

  * `host foo` - traffic to and from host foo
  * `net[mask]` - Any traffic to or from network, with optionally provided netmask. Use IP notation for this, providing as many octets of the IP address as you need (less specific will get you more traffic). For example, 192.168.1.12 will get that one host, 192.168.1 will get anything from 192.168.1.0-255. You could also use something like “10” to get anything on 10.0.0.0/8.
  * `port` - Only traffic to/from this port
  * `portrange` - Only traffic to/from this range of ports (so you don’t have to type port 3240 or port 3241 or port 3242…)
  * `tcp` and `udp`  to filter for only packets of that specific protocol. More protocols are available but you probably won’t use them.


For the statements above, `dst` and `src` can be used as extra filters for packets to destination X or from source Y. They can also be used by themselves to capture all traffic
e.g.

	
  * `dst host foo` - Only packets with a destination of "foo"
  * `src net 10.1` - Only packets from network 10.1.0.0/16
  * `src 192.168.1.1` - Any traffic from 192.168.1.1


### Combining statements


You can use the keywords `and`, `or`, and `not` to create higher level expressions.

`&&`, `||`, and `!` can be used instead of `and`, `or`, and `not`, but I would avoid using them on the command line because they'll have to be escaped, which adds complexity. Where they are useful is if you write a tcpdump input file, which is the `-F` option on tcpdump. Similar to `sed` or `awk` scripts, you can pass a file containing the expression to be evaluated. This separate file is useful if you want to commit it to version control or have a number of different filters saved.


## Example usage


Try running a few of these on your system (you’ll need to be root or use sudo). All of packets matching the filter will be printed to stdout, which could then be redirected to a file with `> filename` or `| tee filename`. Quit the tcpdump with ctrl+c.

{{< highlight bash >}}
tcpdump tcp # Monitors any tcp traffic on your system
tcpdump dst port 80 # Monitors http requests
tcpdump udp port 53 # Monitors DNS queries
tcpdump host google.com # Go to google.com in your browser while this is running
tcpdump port 80 or port 443 # Listens to all traffic going to or coming from ports 80 and 443
{{< / highlight >}}


### Notable Options


{{< highlight bash >}}
-i eth0
{{< / highlight >}}

Listen only on eth0 for all packets. You can comma separate interfaces if there’s traffic on multiple interfaces (say a relay or firewall). In some platforms you can use the keyword `any`. Specifying an interface isn’t required.

{{< highlight bash >}}
-p, --no-promiscuous-mode
{{< / highlight >}}

Promiscuous mode allows tcpdump to receive packets not directed to your computer but are being broadcast over the same medium. This is generally called “sniffing” traffic, and tcpdump enables it by default. You have to explicitly turn it off.

On wireless networks, and on wired networks with special devices called [network hubs](http://en.wikipedia.org/wiki/Ethernet_hub), your computer can see packets intended for other computers because it’s all being transmitted on the same broadcast medium. Normally your computer ignores these packets, so in order to get this same behavior you need to disable promiscuous mode.

One situation where promiscuous mode would come in handy is if you could not run tcpdump on either of the two devices communicating. For example, if I wanted to see what data a smart home device, like the Nest thermostat, was sending to my router, I may start tcpdump with promiscuous mode and watch for traffic sent between the two devices.

{{< highlight bash >}}
-w # and corresponding -r
{{< / highlight >}}

The `-w` option writes raw packets out to a file instead of interpreting them. The `-r` option will read a file of these raw packets. Depending on your use case, you may want the raw packet stream saved.


## Example scenarios


A few scenarios I’ve used tcpdump for

  * Benchmarking application network usage between a client and server
  * Finding out whether an application was hitting a license server
  * Security auditing (this software isn't trying to contact something outside our network?)
  * Determining which files are opened via NFS, and what operations are performed against each file (read, write, stat, etc)
  * Finding the client making the most fileserver requests


