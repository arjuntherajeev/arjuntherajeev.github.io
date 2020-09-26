---
full_title: "Entry 0x01 - My First Post"
description: "An introduction to Arjun's beginner pentesting setup and lab environment."
date: "2019-05-25"
coverImage: "screen-shot-2019-05-25-at-11.59.40-pm.png"
layout: post
tags: 
- Beginner
- CTF
- Kali
- Linux
- Vulnhub
---

Hello reader,

For the past few months or so, I have been playing with [VulnHub](https://www.vulnhub.com/).

Wondering what is VulnHub? Well, the [about page](https://www.vulnhub.com/about/) sums it up perfectly. In a nutshell, VulnHub serves as a repository of _vulnerable_ _virtual machines_ that allow curious minds to learn and practice in a safe environment.

By breaking into it, I refer to gaining administrative privileges on the machine. In most cases, this is followed by finding a _flag_. A flag refers to a unique string or number which was intentionally placed by the creator of the challenge in a specific location on the machine. Finding the flag is essential as it is a form of _evidence_ that we actually did solve the challenge. On most Linux VMs, the flag is usually found in `/root/flag.txt`. However, always be ready for more flags as this completely depends on the challenge.

There are basically 2 types of machines required to proceed with the challenges on VulnHub:

1. Attack Machine
2. Victim Machine

**Attack Machine** refers to a machine loaded with the tools and software ready to _attack_ the vulnerable machine. I use an image of **Kali Linux** imported into VirtualBox as my attack machine. You can download Kali Linux for VMware or VirtualBox from the [Offensive Security website](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/). Kali Linux comes with an excellent collection of tools (scripts, wordlists, frameworks, etc.) which can readily assist you in the various stages of attacking the vulnerable machine.

**Victim Machine** refers to the vulnerable VM downloaded from the VulnHub website. Most challenges provide both - VMWare and VirtualBox images for convenience.

Once both machines are ready to go, it is important to check the networking. I personally use **host-only** **networking** to ensure that both the VMs are isolated to my host machine.

With both the machines configured, it is **game on**. I lookup man pages, perform Google searches, and dive into source code with hopes of figuring out how to beat the challenge in front of me.

The past few months have been superb. I have learnt tremendously and owe most of my knowledge to well-written walkthroughs.

Here's to owning more VMs and finding more flags!
