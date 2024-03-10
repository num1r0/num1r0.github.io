---
title: How to block Internet access for programs on Linux
description: How to block programs from accessing the Internet on Linux with these simple steps
layout: post
permalink: how-to-block-internet-access-for-programs-linux
image: "img/block-internet-access-programs/firewall.jpeg"
tags: HowTo
categories: HowTo
---
There are times when we may need to block a specific program from accessing the Internet. I'm not saying while analysing some malicious executable, for that you should use a virtual environment (for starters). What if you want to block a text editor like Visual Studio Code? It is SCARY to look at the Wireshark/tcpdump output while executing it! Before even loading my file, it already made several DNS requests and established 4 TCP connections; and that's with disabled analytics/metrics reports.

And yes, [Codium](https://vscodium.com/ "VSCodium - Open Source Binaries of VSCode") is also chatty!

{% include adsense.html %}

## Steps to block Internet access

1. Create a new local group that we'll use as a Internet contained group
```bash
sudo addgroup no-internet
```
2. Get the ID of the newly created group and store it in `GID` variable
```bash
GID=$(getent group no-internet | cut -d ':' -f3)
```
3. Add a nftables rule to DROP outgoing traffic from the new group
```bash
nft add rule ip filter OUTPUT skgid $GID counter drop
```
4. Execute commands/programs from inside the new group
```bash
sudo -g no-internet ping 8.8.8.8 # No Internet access for ping
sudo -g no-internet code # No Internet access for VS Code
```

That's it! Expect some errors from VS Code/VS Codium about not being able to connect to the extensions repositories, at this stage you probably have all the necessary extensions installed.