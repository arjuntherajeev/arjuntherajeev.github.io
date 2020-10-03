---
full_title: "Entry 0x04 - VulnHub: DC: 3 Walkthrough"
description: "A step-by-step walkthrough of solving the DC: 3 pentesting challenge from VulnHub."
date: "2019-06-03"
coverImage: "dc-3-00.png"
layout: post
tags:
- Beginner
- CTF
- Kali
- Linux
- Vulnhub
- Walkthrough
- Writeup
- Security
- DC-Series
---

**DC: 3** is a challenge posted on [VulnHub](https://www.vulnhub.com/entry/dc-3,312/) created by [DCAU](https://www.vulnhub.com/author/dcau,610/). This is a write-up of my experience solving this awesome CTF challenge.

With my Attack Machine (**Kali Linux**) and Victim Machine (**DC: 3**) set up and running, I decided to get down to solving this challenge.

> Read more about my set up and environment [here](/blog/my-first-post)

I decided to start my journey with `netdiscover` to complete the _host discovery_ phase as follows: `netdiscover -r 192.168.56.0/24`

![Screen Shot 2019-06-03 at 1.00.07 PM.png](/assets/images/screen-shot-2019-06-03-at-1.00.07-pm.png)

Great, the Victim Machine is `192.168.56.101`. Let’s continue!

For the next step, I decided to run an `nmap` scan to determine open ports and services using the command: `nmap -sV -F 192.168.56.101`

![Screen Shot 2019-06-03 at 1.02.48 PM.png](/assets/images/screen-shot-2019-06-03-at-1.02.48-pm.png)

The `nmap` scan revealed that port `80` was open. Time to hit the browser!

> Pro-tip: if you want to stay in the _flow_ of using the terminal, type in `firefox http://192.168.56.101`

![Screen Shot 2019-06-03 at 1.05.11 PM.png](/assets/images/screen-shot-2019-06-03-at-1.05.11-pm.png)

Cool, a webpage with a message from @DCAU!

I decided to go ahead with `dirb` to get a feel of the website and its directories using the command: `dirb http://192.168.56.101`

![Screen Shot 2019-06-03 at 1.09.17 PM.png](/assets/images/screen-shot-2019-06-03-at-1.09.17-pm.png)

Wow! That's a lot of URLs to check. However, the `/administrator` path caught my attention. I decided to check it out - `http://192.168.56.101/administrator`

![Screen Shot 2019-06-03 at 1.11.38 PM.png](/assets/images/screen-shot-2019-06-03-at-1.11.38-pm.png)

Joomla!

After I ran `dirb`, I decided to run `nikto` just to be sure that I did not miss anything important. The command is: `nikto -h http://192.168.56.101`

![Screen Shot 2019-06-03 at 1.36.51 PM.png](/assets/images/screen-shot-2019-06-03-at-1.36.51-pm.png)

Okay. Now, I realized that I was working with a Joomla-based web application. Are there any specific tools to enumerate and find vulnerabilities? Is there something like `wpscan` for WordPress available for Joomla?

I needed to do my research.

I came across an interesting tool called [JoomScan](https://tools.kali.org/web-applications/joomscan) - a Joomla Vulnerability Scanner. It is available on Kali Linux so I decided to give it a test run with our Joomla website using the command: `joomscan -url http://192.168.56.101 -enumerate-components`

![Screen Shot 2019-06-03 at 2.15.03 PM.png](/assets/images/screen-shot-2019-06-03-at-2.15.03-pm.png)

Hmm, nothing out of the blue. The useful information is the Joomla version - `3.7.0`.

I decided to fire up `searchsploit` to check for any exploits for this version of Joomla using the command: `searchsploit joomla 3.7.0`

![Screen Shot 2019-06-03 at 2.17.47 PM](/assets/images/screen-shot-2019-06-03-at-2.17.47-pm.png)

Woohoo! I was excited to give it a shot. First, I decided to _copy_ the `.TXT` file to my current directory using the command: `searchsploit -m 42033`

![Screen Shot 2019-06-03 at 2.19.31 PM.png](/assets/images/screen-shot-2019-06-03-at-2.19.31-pm.png)

Nice! Let's have a look using the trusty `cat` command: `cat 42033.txt`

![Screen Shot 2019-06-03 at 2.21.16 PM.png](/assets/images/screen-shot-2019-06-03-at-2.21.16-pm.png)

Interesting. Basically, we need to run the `sqlmap` command to proceed with the _injection_. Obviously, in our case, I needed to change `localhost` to `192.168.56.101` before running the command:

![Screen Shot 2019-06-03 at 2.24.19 PM.png](/assets/images/screen-shot-2019-06-03-at-2.24.19-pm.png)

![Screen Shot 2019-06-03 at 2.25.20 PM.png](/assets/images/screen-shot-2019-06-03-at-2.25.20-pm.png)

Super! `sqlmap` got us the database names. The most interesting one is `joomladb`. I decided to get table names as follows: `sqlmap -u "http://192.168.56.101/index.php option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] --tables -D joomladb`

The only change from the previous command to this is that we mention: `--tables -D joomladb` which is a way of saying that we want a list of _tables_ from the database `joomladb`

![Screen Shot 2019-06-03 at 2.31.04 PM.png](/assets/images/screen-shot-2019-06-03-at-2.31.04-pm.png)

![Screen Shot 2019-06-03 at 2.31.59 PM.png](/assets/images/screen-shot-2019-06-03-at-2.31.59-pm.png)

The interesting thing is that we got a list of **76** tables in the `joomladb` database. For me, the crucial table is `#__users`. I decided to get the columns in that table using the command: `sqlmap -u "http://192.168.56.101/index.php option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] --columns -D joomladb -T "#__users"`

![Screen Shot 2019-06-03 at 3.44.11 PM.png](/assets/images/screen-shot-2019-06-03-at-3.44.11-pm.png)

For some reason, `sqlmap` gave a **warning** saying `unable to retrieve column names for table '#__users' in database 'joomladb'` For this reason, I gave `y[es]` to the question `Do you want to use common column existence check? [y/N/q]`

It got names of **6** columns in the table `#__users` in the database `joomladb`. I was super interested in the `email`, `password`, and `username` columns. Time to see the data!

I modified the command as follows: `sqlmap -u "http://192.168.56.101/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomladb -T "#__users" -C email,password,username --dump`

![Screen Shot 2019-06-03 at 3.55.47 PM.png](/assets/images/screen-shot-2019-06-03-at-3.55.47-pm.png)

Woah! `admin` credentials? Gold!

The data dumped includes:

- email: `freddy@norealaddress.net`
- password: `$2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu`
- username: `admin`

What's next? We need to break the _hash_. But first, we need to identify the hash type. I decided to proceed with [this](https://www.tunnelsup.com/hash-analyzer/) trusty website:

![Screen Shot 2019-06-03 at 4.00.10 PM](/assets/images/screen-shot-2019-06-03-at-4.00.10-pm.png)

Cool - `bcrypt`

I decided to use `hashcat` to get the password as follows: `hashcat -m 3200 hash.txt /usr/share/wordlists/rockyou.txt --force`

![Screen Shot 2019-06-03 at 4.11.00 PM.png](/assets/images/screen-shot-2019-06-03-at-4.11.00-pm.png)

Awesome! We got the password for `admin` - `snoopy`. Back to the browser!

![Screen Shot 2019-06-03 at 4.13.45 PM.png](/assets/images/screen-shot-2019-06-03-at-4.13.45-pm.png)

Good, now I wanted to get access to the shell... somehow.

To achieve that, I decided to explore the Joomla interface. With my experience with WordPress, it is usually extensions, plugins, or modules that allow us to somehow _inject_ PHP code which can help us open a reverse shell.

In my journey, I came across an interesting page - `Extensions > Templates`

![Screen Shot 2019-06-03 at 4.17.15 PM.png](/assets/images/screen-shot-2019-06-03-at-4.17.15-pm.png)

I opened`Templates` in the left menu...

![Screen Shot 2019-06-03 at 4.19.05 PM.png](/assets/images/screen-shot-2019-06-03-at-4.19.05-pm.png)

Interesting. We can _preview_ these templates as well as their files. What about the option to edit? I clicked on the first one - `Beez3 Details and Files`

![Screen Shot 2019-06-03 at 4.20.30 PM.png](/assets/images/screen-shot-2019-06-03-at-4.20.30-pm.png)

Yeah - PHP files!

Now, I decided to edit `index.php` and replace its contents with the code for `php-reverse-shell`.  Back to the terminal!

The first step is to locate the `php-reverse-shell` code in Kali Linux and modify it to our configuration. I decided to use my trusty `vim` for this purpose...

![Screen Shot 2019-06-03 at 4.24.47 PM.png](/assets/images/screen-shot-2019-06-03-at-4.24.47-pm.png)

I changed the variables - `$ip` and `$port` to my Kali Linux IP address and a custom port number.

![Screen Shot 2019-06-03 at 4.28.06 PM.png](/assets/images/screen-shot-2019-06-03-at-4.28.06-pm.png)

Great, time to create a listener using the trusty netcat: `nc -lvp 7777`

![Screen Shot 2019-06-03 at 4.30.20 PM.png](/assets/images/screen-shot-2019-06-03-at-4.30.20-pm.png)

Time to trigger the `index.php` of the **Beez3** template! Once I saved the edited `index.php` file, I clicked on `Template Preview`...

![Screen Shot 2019-06-03 at 4.32.32 PM.png](/assets/images/screen-shot-2019-06-03-at-4.32.32-pm.png)

There's nothing? Doesn't matter... I got shell!

![Screen Shot 2019-06-03 at 4.32.17 PM.png](/assets/images/screen-shot-2019-06-03-at-4.32.17-pm.png)

I really wanted a TTY shell... something easier to work with. A great technique I picked up is as follows: `python -c 'import pty;pty.spawn("/bin/bash");'`

![Screen Shot 2019-06-03 at 4.33.59 PM.png](/assets/images/screen-shot-2019-06-03-at-4.33.59-pm.png)

There we go!

Okay, time for privilege escalation. The tough one!

I decided to try exploiting SUID executables - ones which can be executed with `root` privileges. The command `find / -perm -u=s -type f 2>/dev/null` prints a list of executables with the SUID bit set.

![Screen Shot 2019-06-03 at 4.41.58 PM.png](/assets/images/screen-shot-2019-06-03-at-4.41.58-pm.png)

`/bin/ntfs-3g` looked interesting. Some [Googling](https://www.rapid7.com/db/modules/exploit/linux/local/ntfs3g_priv_esc) revealed that it is possible to escalate privileges with `ntfs-3g`.

I immediately looked up any available exploits using `searchsploit` using the command: `searchsploit ntfs 3g`

![Screen Shot 2019-06-03 at 7.20.44 PM.png](/assets/images/screen-shot-2019-06-03-at-7.20.44-pm.png)

I was excited but both these exploits never worked. I had to find another way escalate my privileges on this machine. Kernel exploits perhaps?

I decided to go down the Linux kernel exploit path. First, I needed some context so I ran 2 useful commands:

1. `uname -a` which prints **all** system information
2. `cat /etc/os-release` which contains specific Linux distro information

![Screen Shot 2019-06-03 at 7.29.53 PM.png](/assets/images/screen-shot-2019-06-03-at-7.29.53-pm.png)

Cool, `Ubuntu 16.04`. Time to look for kernel exploits!

![Screen Shot 2019-06-03 at 7.31.09 PM.png](/assets/images/screen-shot-2019-06-03-at-7.31.09-pm.png)

A quick search on `searchsploit` revealed a decent list of exploits. What's next is a ton of trial-and-error...

One of the interesting exploits on this list is `Linux Kernel 4.4.x (Ubuntu 16.04) - 'double-fdput()' bpf(BPF_PROG_LOAD) Privilege Escalation`

I decided to give it a try as follows: `searchsploit -m 39772`

![Screen Shot 2019-06-03 at 7.39.47 PM.png](/assets/images/screen-shot-2019-06-03-at-7.39.47-pm.png)

Looking into the `.TXT` file reveals that we need to first download the exploit using [this link](https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip). I decided to use `wget` for this purpose. Make sure your network settings support downloading from the Internet. I had to change mine to NAT temporarily.

![Screen Shot 2019-06-03 at 7.43.29 PM.png](/assets/images/screen-shot-2019-06-03-at-7.43.29-pm.png)

Great! `39772.zip` needs to be transferred to the Victim Machine (DC3). I decided to unzip the files and have a look.

![Screen Shot 2019-06-03 at 7.48.41 PM.png](/assets/images/screen-shot-2019-06-03-at-7.48.41-pm.png)

As per the exploit's [ExploitDB webpage](https://www.exploit-db.com/exploits/39772), we are interested in `exploit.tar` Next, I ran the command: `tar -xvf exploit.tar`

![Screen Shot 2019-06-03 at 7.50.38 PM.png](/assets/images/screen-shot-2019-06-03-at-7.50.38-pm.png)

Lots of files which needs to be transferred to the Victim Machine! I decided to achieve this using the Joomla web interface itself. This simply involves uploading the files as images/media.

I navigated to the blog post on the Joomla website using the browser.

![Screen Shot 2019-06-03 at 8.19.49 PM.png](/assets/images/screen-shot-2019-06-03-at-8.19.49-pm.png)

Great, I just needed to upload the **4** files using this interface. However, Joomla is smart and stopped me from uploading files with the extensions `.c` and `.sh`. I needed to find an alternative route.

What worked was `.TXT`. I renamed all the files with the `.TXT`  extension using the `mv` command as follows: `mv <old_file_name> <new_file_name>`

![Screen Shot 2019-06-03 at 8.15.26 PM.png](/assets/images/screen-shot-2019-06-03-at-8.15.26-pm.png)

The uploads worked!

![Screen Shot 2019-06-03 at 8.19.12 PM](/assets/images/screen-shot-2019-06-03-at-8.19.12-pm.png)

Back to the reverse shell!

I navigated to the `/var/www/html/images` folder and renamed all the files back to their original extensions. Again, using the `mv` command as seen above.

![Screen Shot 2019-06-03 at 8.16.18 PM.png](/assets/images/screen-shot-2019-06-03-at-8.16.18-pm.png)

According to the exploit's instructions, the first step is to run `compile.sh` as follows: `./compile.sh`

But first... I needed to set the right executable permissions using the command: `chmod +x compile.sh`

This created `doubleput` which is the next program to be executed (as per the exploit's instructions). I set the right executable permissions using `chmod +x doubleput`

![Screen Shot 2019-06-03 at 8.17.20 PM.png](/assets/images/screen-shot-2019-06-03-at-8.17.20-pm.png)

Woohoo! `root` is what I am!

Time to get the flag inside `/root/` which was named `the-flag.txt`

![Screen Shot 2019-06-03 at 8.17.45 PM.png](/assets/images/screen-shot-2019-06-03-at-8.17.45-pm.png)

Finally! Awesome!

#### My Thoughts

That was a really interesting challenge! Props to @DCAU for coming up with a great challenge revolving around Joomla.

I really learnt a lot about Joomla and better understood the usage of Linux kernel exploits for _privilege escalation_ in CTFs. 
**TL;DR** - be ready for tons of trial and error.

If you enjoyed reading this, please check out my [DC: 6 walkthrough](/blog/vulnhub-dc-6-walkthrough) which is another challenge by @DCAU in the [DC series](https://www.vulnhub.com/series/dc,199/).
