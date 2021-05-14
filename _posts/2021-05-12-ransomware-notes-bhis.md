---
full_title: "Notes: BHIS Emergency Ransomware Webcast (11 May 2021)"
description: "A step-by-step guide on how to download, install, and set up IBM QRadar Community Edition V7.3.3 on VirtualBox."
date: "2021-05-13"
coverImage: "ran_pic.jpeg"
layout: post
tags:
- Ransomware
- Security
- DarkSide
- Colonial-Pipeline
- Notes
- Webcast 
- Raccine
- Deception
- Beaconing
- Honeypot
- C&C
- vssadmin
sitemap:
    lastmod: 2021-05-13
---

It's always interesting to read about the devastation that Ransomware brings about to organizations around the globe. Recently, on May 7, Colonial Pipeline fell victim to a Ransomware attack orchestrated by the [DarkSide group](https://www.independent.co.uk/news/world/americas/darkside-hacker-group-pipeline-ransomware-b1844972.html). Check out [this ZDNet article](https://www.zdnet.com/article/colonial-pipeline-ransomware-attack-everything-you-need-to-know/) and [this Security Intelligence post](https://securityintelligence.com/posts/darkside-oil-pipeline-ransomware-attack/) summarizing the incident.

While the investigations (and aftermaths) are ongoing, I came across this emergency webcast organized by [Black Hills Information Security](https://www.youtube.com/c/BlackHillsInformationSecurity/featured) presented by John Strand on YouTube.

<iframe width="100%" height="415" src="https://www.youtube.com/embed/wKAQB4Yp-k4?start=1765" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In this post, I wanted to list the key takeaways (trends, tools, and techniques) from the webcast. They are:

## Attack Motivation

Rather than targeting organizations limited to having IT environments, attackers have understood the value and impact of targeting critical infrastructure with OT environments. The reasoning is simple - more chaos and destruction - forcing organizations to pay the ransom to ensure that critical infrastructure and lives continue. 

As seen in the Tweets below from Patrick De Haan ([@GasBuddyGuy](https://twitter.com/GasBuddyGuy)), this is a classic example of chaos - limited resources leading to panic-buying of fuel.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">BREAKING: 71% of gas stations in metro Atlanta are without fuel.</p>&mdash; Patrick De Haan ⛽️📊 (@GasBuddyGuy) <a href="https://twitter.com/GasBuddyGuy/status/1392644633491124226?ref_src=twsrc%5Etfw">May 13, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">South Florida- you&#39;d be well advised to stop panic buying- you&#39;re creating something from nothing.</p>&mdash; Patrick De Haan ⛽️📊 (@GasBuddyGuy) <a href="https://twitter.com/GasBuddyGuy/status/1392815539785973763?ref_src=twsrc%5Etfw">May 13, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">NATIONAL AVERAGE REACHES $3/GAL, FIRST TIME IN NEARLY 7 YEARS<br><br>The national average price of gasoline has reached $3 per gallon for the first time October 30, 2014.</p>&mdash; Patrick De Haan ⛽️📊 (@GasBuddyGuy) <a href="https://twitter.com/GasBuddyGuy/status/1392446812716507136?ref_src=twsrc%5Etfw">May 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## Deception

John says that Deception is no longer a "nice to have". In fact, Deception is core and essential.

EDR in itself is not sufficient to protect. We spend too much effort and time securing the endpoint. However, companies are still getting compromised with EDR solutions deployed. 

Deception should be more than just Honeypots. We need to be looking at the attack pathways that attackers take post-exploitation to move laterally in an environment to take over that enivornment. The attackers are practicing tried-and-tested techniques which are the same techniques used by red teams/pen-testing professionals. 

Put Deception in the right places to detect when something has gone awry. 

### Techniques

Set bait for attackers. Word documents are great because we can put them on: 
1. Shares
2. Compromised systems
3. Websites
4. Email to spammers

Set in the right place to get triggered when an attacker accesses it. You can extract multiple valuable fields, such as:
1. IP address
2. Machine name
3. User ID

Bonus: Easy alternative to DLP solution

#### What to check out:

1. [jqreator/honeydoc](https://github.com/jqreator/honeydoc)
> HoneyDoc creates a "honey" document including things like fake names and social security numbers to look appealing to would be attackers. It also includes a 1x1 pixel png file called `hello.png` as the tracking image so that you can see the IPs of who opens the document in your web server logs. To install the image, place it in your web servers root directory (or any other directory you want to use) and specify your URL using the `--url` flag when generating the document.
>
> Once the document is generated it can be edited and personalized to make it look any way you want.
>
> Check out [HoneyDoc's GitHub repo](https://github.com/activecm/rita) for more information about installation and usage. 

2. [Canarytokens](https://canarytokens.org/generate)
> You'll be familiar with web bugs, the transparent images which track when someone opens an email. They work by embedding a unique URL in a page's image tag, and monitoring incoming `GET` requests.
> 
> Imagine doing that, but for file reads, database queries, process executions or patterns in log files. Canarytokens does all this and more, letting you implant traps in your production systems rather than setting up separate honeypots.
>
> How Canarytokens work (in 3 short steps):
> 1. Go to `canarytokens.org` and select your Canarytoken (supply an email to be notified at as well as a memo that reminds you which Canarytoken this is and where you put it).
> 2. Place the generated Canarytoken somewhere special (read the [examples](https://docs.canarytokens.org/guide/examples.html) for ideas on where).
> 3. If an attacker ever trips on the Canarytoken somehow, you'll get an email letting you know that it is happened.
> 
> Check out Canarytoken's excellent [documentation](https://docs.canarytokens.org/guide/getting-started.html) to view examples and more. 

3. Honey Accounts
> Useful to detect attackers trying to fly under-the-radar of SIEM and UEBA when they use a technique such as password spraying. As soon as they attempt to login to the honey account, a rule is triggered to shutdown the machine and alert the SOC/IR team.
>
> The idea is that when someone does breach your network perimeter, some of the first steps in performing recon is collecting information from Active Directory. In this recon, they stumble on a DA account called ‘helpdeskDA’. They even discover a password in the description! Well this looks like an easy win and a critical finding. In order to figure out how to leverage this new found user, the attacker attempts to RDP or use psexec to move to a higher value target. In doing so, AD checks the credentials and returns to the attacker that his newfound account is not allowed to login during this time. Meanwhile, this login attempt has triggered an alert and is being investigated.
> 
> Check out [this post](https://jordanpotti.com/2017/11/06/honey-accounts/) from Jordan Potti for steps and more information.

4. Kerberoasting Deception
> Threat actors can abuse the Kerberos protocol to recover passwords related to service accounts using a tactic called Kerberoasting. In a Windows domain, the authentication protocol Kerberos uses a Ticket Granting Ticket (TGT) to request access tokens from the Ticket Granting Service (TGS) for specific resources/systems joined to the domain.
>
> In Kerberoasting, threat actors abuse valid Kerberos TGTs to make a request for a TGS from any valid Service Principal Name (SPN) within your Microsoft Active Directory domain. These TGSs are vulnerable to offline password cracking, which can allow a threat actor to recover the plaintext password of the associated service account mapped by the SPN.
> 
> To avoid false positive detections, you can create a service account honeypot (honeycred) to detect Kerberoasting.
> 
> Check out Blumira's [Guide To Cybersecurity Deception Techniques](https://www.blumira.com/cybersecurity-deception-techniques/) for more information.

## Beacons 

### What is C&C Beaconing?
> Command-and-Control (C&C or C2) beaconing is a type of malicious communication between a C&C server and malware on an infected host. C&C servers can orchestrate a variety of nefarious acts, from denial of service (DoS) attacks to ransomware to data exfiltration.
> 
> Often, the infected host will periodically check in with the C&C server on a regular schedule, hence the term beaconing. This pattern can differentiate it from normal traffic because of the regularity of intervals. But beaconing on common ports and protocols (such as `HTTP:80` or `HTTPS:443`) often obscures malicious traffic within normal traffic and helps the attacker evade firewalls. Another evasion tactic, notably used by SUNBURST, involves waiting long, randomized periods of time before communicating.
> 
> Check out ExtraHop's [quick primer on C&C Beaconing](https://www.extrahop.com/resources/attacks/c-c-beaconing/) for more information.

#### What to check out:

1. [activecm/rita](https://github.com/activecm/rita)
> RITA (Real Intelligence Threat Analytics) is a framework for detecting command and control communication through network traffic analysis.
> 
> The framework ingests [Zeek](https://zeek.org/) logs in `TSV` format, and currently supports the following major features:
> 1. Beaconing Detection: Search for signs of beaconing behavior in and out of your network
> 2. DNS Tunneling Detection Search for signs of DNS based covert channels
> 3. Blacklist Checking: Query blacklists to search for suspicious domains and hosts
>
> Check out [RITA's GitHub repo](https://github.com/activecm/rita) for more information about installation and usage.

## Ransomware of the Third Kind

John says that Ransomware was typically seen in two categories:

1. Ransomware that encrypts your hard drive
2. Ransomware that encrypts your files

However, trends reveal a third kind of Ransomware - one using which attackers steal files and threaten to release them to the public. This is done for a couple of reasons:
1. Proof of Life: Attackers want to prove that they really have infiltrated the organization's network and stolen data
2. Insurance: Attackers will release all files to the public if the ransom is not paid threatening confidentiality 

## Raccine - A Simple Ransomware Vaccine

> We see Ransomware delete all shadow copies using `vssadmin` pretty often. What if we could just intercept that request and kill the invoking process? Let's try to create a simple vaccine.

[Neo23x0/Raccine](https://github.com/Neo23x0/Raccine) is a simple yet powerful tool created by Florian Roth which can prevent Ransomware disaster. It works as follows:

> We register a debugger for `vssadmin.exe` (and `wmic.exe`), which is our compiled `raccine.exe`. Raccine is a binary, that first collects all PIDs of the parent processes and then tries to kill all parent processes.

### What is `vssadmin`?

> The Volume Shadow Service Administration Tool (`vssadmin.exe`) is a default Windows process that manipulates volume shadow copies of the files on a given computer. These shadow copies are often used as backups, and they can be used to restore or revert files back to a previous state if they are corrupted or lost for some reason. `vssadmin` is commonly used by backup utilities and systems administrators.
> 
> As such, the people responsible for Ransomware campaigns often attempt to delete them so that their victims can’t restore file access by reverting to the shadow copies. As a note, interacting with `vssadmin` should require administrative privileges.
>
> Check out [this article](https://redcanary.com/blog/its-all-fun-and-games-until-ransomware-deletes-the-shadow-copies/) from Red Canary explaining more about `vssadmin` and how to detect malicious usage. 

### Avantages:
> 1. The method is rather generic
> 2. We don't have to replace a system file (`vssadmin.exe` or `wmic.exe`), which could lead to integrity problems and could break our raccination on each patch day
> 3. Flexible YARA rule scanning of command line params for malicious activity
> 4. The changes are easy to undo
> 5. Runs on Windows 7 / Windows 2008 R2 or higher
> 6. No running executable or additional service required (agent-less)

### Disadvantages / Blind Spots:
> 1. The legitimate use of `vssadmin.exe delete shadows` (or any other blacklisted combination) isn't possible anymore
> 2. It even kills the processes that tried to invoke `vssadmin.exe delete shadows`, which could be a backup process
> 3. This won't catch methods in which the malicious process isn't one of the processes in the tree that has invoked `vssadmin.exe` (e.g. `via schtasks`)
>
> Check out [Raccine's GitHub repo](https://github.com/Neo23x0/Raccine) for more information about installation and usage.

## Ransomware Protection in Windows

### What is Controlled Folder Access?
> Controlled folder access helps protect your valuable data from malicious apps and threats, such as Ransomware. Controlled folder access protects your data by checking apps against a list of known, trusted apps. Supported on `Windows Server 2019` and `Windows 10 version 1709 and later` clients, controlled folder access can be turned on using the Windows Security App, Microsoft Endpoint Configuration Manager, or Intune (for managed devices).
> 
> With controlled folder access in place, a notification appears on the computer where an app attempted to make changes to a file in a protected folder. You can customize the notification with your company details and contact information. You can also enable the rules individually to customize what techniques the feature monitors.
>
> The protected folders include common system folders (including boot sectors), and you can add more folders. You can also allow apps to give them access to the protected folders.
> 
> Windows system folders are protected by default, along with several other folders:
>
> - `c:\Users\<username>\Documents`
> - `c:\Users\Public\Documents`
> - `c:\Users\<username>\Pictures`
> - `c:\Users\Public\Pictures`
> - `c:\Users\Public\Videos`
> - `c:\Users\<username>\Videos`
> - `c:\Users\<username>\Music`
> - `c:\Users\Public\Music`
> - `c:\Users\<username>\Favorites`
> 
> You can configure additional folders as protected, but you cannot remove the Windows system folders that are protected by default.
>
> Check out [this page](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/controlled-folders?view=o365-worldwide) on Microsoft docs to learn more about Controlled Folder Access and its features.

## IOCs vs Hardening & Generic Rules 

John referred to this Tweet from Florian below and commented about the over-reliance on IOCs to detect Ransomware. 

Instead, organizations should focus on hardening using generic rules to detect known malicious techniques. This is due to the fact that IOCs (file hashes, IP addresses, etc) can vary in instances of targeted attacks due to customization and obfuscation by the attacker.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">The typical approach <a href="https://twitter.com/hashtag/Ransomware?src=hash&amp;ref_src=twsrc%5Etfw">#Ransomware</a> <a href="https://t.co/0RXF1RE4fd">pic.twitter.com/0RXF1RE4fd</a></p>&mdash; Florian Roth (@cyb3rops) <a href="https://twitter.com/cyb3rops/status/1392016608365797376?ref_src=twsrc%5Etfw">May 11, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>