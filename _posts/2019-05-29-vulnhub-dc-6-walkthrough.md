---
full_title: "VulnHub: DC: 6 Walkthrough"
date: "2019-05-29"
coverImage: "dc-6-00.png"
layout: post
---

**DC: 6** is a challenge posted on [VulnHub](https://www.vulnhub.com/entry/dc-6,315/) created by [DCAU](https://www.vulnhub.com/author/dcau,610/). This is a write-up of my experience solving this awesome CTF challenge.

With my Attack Machine (**Kali Linux**) and Victim Machine (**DC: 6**) set up and running, I decided to get down to solving this challenge.

> Read more about my set up and environment [here](/2019/05/25/entry-0x01-my-first-post.html/)

I decided to start my journey with `netdiscover` to complete the _host discovery_ phase as follows: `netdiscover -r 192.168.56.0/24`

![Screen Shot 2019-05-29 at 3.47.31 PM.png](/assets/images/screen-shot-2019-05-29-at-3.47.31-pm.png)

Great, the Victim Machine is `192.168.56.103`. Let's continue!

For the next step, I decided to run an `nmap` scan to determine open ports and services using the command: `nmap -sV -F 192.168.56.103`

![Screen Shot 2019-05-29 at 3.49.55 PM.png](/assets/images/screen-shot-2019-05-29-at-3.49.55-pm.png)

A _fast_ scan revealed that ports - `22` and `80` were open. Let's hit the browser!

![Screen Shot 2019-05-29 at 4.01.06 PM.png](/assets/images/screen-shot-2019-05-29-at-4.01.06-pm.png)

All I did was navigate to `http://192.168.56.103` but it immediately re-directed me to `http://wordy`.

Not a problem... I just need to edit my `/etc/hosts` file. I decided to use `vim` (still figuring out how to exit) with the command: `vim /etc/hosts`

![Screen Shot 2019-05-29 at 4.16.59 PM](/assets/images/screen-shot-2019-05-29-at-4.16.59-pm.png)

Back to the browser!

![Screen Shot 2019-05-29 at 4.18.07 PM.png](/assets/images/screen-shot-2019-05-29-at-4.18.07-pm.png)

Awesome, we got `Wordy - Just another WordPress site`.

I decided to start my exploration with `nikto` - a trusty tool to tell me a bit about the website and its backend using the command: `nikto -h http://wordy`

![Screen Shot 2019-05-29 at 4.31.04 PM.png](/assets/images/screen-shot-2019-05-29-at-4.31.04-pm.png)

Perfect, we can confirm that it is a WordPress website. Time to fire up `wpscan`!

`wpscan` is the _de-facto_ tool for any WordPress website. I decided to launch it to test for vulnerabilities and enumerate as much information as possible with the following command: `wpscan --url http://wordy --enumerate`

![Screen Shot 2019-05-29 at 8.31.14 PM.png](/assets/images/screen-shot-2019-05-29-at-8.31.14-pm.png)

![Screen Shot 2019-05-29 at 8.31.54 PM](/assets/images/screen-shot-2019-05-29-at-8.31.54-pm.png)

Wow! I found 2 interesting points from the results of `wpscan`:

1. WordPress version - 5.1.1
2. Identified users - `admin, sarah, graham, mark, jens`

So far we have access to the WordPress UI and a list of usernames. What next?

I decided to explore the possibility of breaking in as a user. So, I navigated to `http://wordy/wp-login.php`

![Screen Shot 2019-05-29 at 8.43.30 PM.png](/assets/images/screen-shot-2019-05-29-at-8.43.30-pm.png)

A pretty normal-looking WordPress login screen. Okay, time to brute-force!

But wait... there's a **clue** from the creator (@DCAU)...

> OK, this isn't really a clue as such, but more of some "we don't want to spend five years waiting for a certain process to finish" kind of advice for those who just want to get on with the job.
> 
> cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt That should save you a few years. ;-)

You're the man (or woman), @DCAU!

Basically, the next step was to _brute-force_ the login to WordPress. To do this, I planned to use `wpscan` itself with the `rockyou.txt` wordlist. But, with @DCAU's advice, I decided to run that Linux command first.

![Screen Shot 2019-05-29 at 9.16.32 PM](/assets/images/screen-shot-2019-05-29-at-9.16.32-pm.png)

This is pretty significant because the `rockyou.txt` wordlist has 14344392 words. The fact that we reduced it to 2668 words is a huge time-saver.

Back to `wpscan` for a brute-force attempt. I ran it using the command: `wpscan --url http://wordy -U admin -P passwords.txt`

This never gave me any matches. Then, it hit me that we have a total of **5** identified users according to `wpscan`. With this knowledge, I changed my command to: `wpscan --url http://wordy -U admin,sarah,mark,jens -P passwords.txt`

![Screen Shot 2019-05-29 at 9.26.54 PM](/assets/images/screen-shot-2019-05-29-at-9.26.54-pm.png)

Boom! We got a hit...

- Username: `mark`
- Password: `helpdesk01`

Almost immediately, I tried to log in to the WordPress UI as `mark`

![Screen Shot 2019-05-29 at 9.28.42 PM](/assets/images/screen-shot-2019-05-29-at-9.28.42-pm.png)

Well hello, I am `Mark Jones`!

One thing that I usually keep in mind is that most users re-use passwords. I wondered... would `Mark Jones` re-use his password for SSH? Only one way to find out!

![Screen Shot 2019-05-29 at 9.32.14 PM](/assets/images/screen-shot-2019-05-29-at-9.32.14-pm.png)

Back to the browser...

![Screen Shot 2019-05-29 at 10.42.13 PM.png](/assets/images/screen-shot-2019-05-29-at-10.42.13-pm.png)

One of the first things that I observed was the plugin - **Activity monitor**. Plugins on WordPress are infamous for being vulnerable. I decided to do some research about this plugin.

![Screen Shot 2019-05-29 at 10.45.53 PM](/assets/images/screen-shot-2019-05-29-at-10.45.53-pm.png)

I found that there is a vulnerability (and exploit) available for this plugin on ExploitDB. To confirm this, I used my favorite tool - `searchsploit` as follows: `searchsploit wordpress activity monitor`

![Screen Shot 2019-05-29 at 10.48.03 PM.png](/assets/images/screen-shot-2019-05-29-at-10.48.03-pm.png)

Awesome! We have the exploit on Kali Linux (our Attack Machine). To use it, I decided to take a note of the ID number - `45274` mentioned in the file name. I proceeded with the command: `searchsploit -m 45274`

![Screen Shot 2019-05-29 at 10.50.22 PM.png](/assets/images/screen-shot-2019-05-29-at-10.50.22-pm.png)

Simply put, this command copies the exploit to our current directory. Time to use it!

First things first, I decided to check the exploit and configure it (if required). To do this, I used my trusty `vim` text editor:

![Screen Shot 2019-05-29 at 10.54.57 PM.png](/assets/images/screen-shot-2019-05-29-at-10.54.57-pm.png)

Cool, there are proper instructions on how to use it. I made 3 changes:

1. I changed `http://localhost:8000` to `http://wordy`
2. I changed `127.0.0.1` to my IP address - `192.168.56.102`
3. I removed the arguments `-nlvp` (I decided to run the listener on my Attack Machine - reverse shell technique)

After saving the changes, I opened the `.HTML` file on my browser:

![Screen Shot 2019-05-29 at 11.01.07 PM.png](/assets/images/screen-shot-2019-05-29-at-11.01.07-pm.png)

Simple and straightforward - a button called **Submit request**. While I was tempted to click it, I realized that I needed to first run the listener on my terminal.

A reverse shell is easily setup using the trusty netcat: `nc -lvp 9999`

![Screen Shot 2019-05-29 at 11.05.46 PM](/assets/images/screen-shot-2019-05-29-at-11.05.46-pm.png)

Time to click the **Submit request** button and stare at netcat:

![Screen Shot 2019-05-29 at 11.08.27 PM](/assets/images/screen-shot-2019-05-29-at-11.08.27-pm.png)

Yeah! We got shell!

I checked my user and it was `www-data` (as expected). I decided to explore and figure out a way to escalate my privileges to `root`.

Based on our previous efforts, we are aware that there are 4 other users on WordPress apart from `admin`. I decided to check if these users exist on the box in the `/home` directory:

![Screen Shot 2019-05-30 at 11.08.29 AM.png](/assets/images/screen-shot-2019-05-30-at-11.08.29-am.png)

They did! I wonder what _hidden treasures_ exist in each directory...

![Screen Shot 2019-05-30 at 11.09.37 AM](/assets/images/screen-shot-2019-05-30-at-11.09.37-am.png)

Oh look, `mark` has an interesting directory called `stuff`. Let's look inside...

![Screen Shot 2019-05-30 at 11.13.01 AM.png](/assets/images/screen-shot-2019-05-30-at-11.13.01-am.png)

`things-to-do.txt` looks interesting. What are its contents? Only one way to find out...

![Screen Shot 2019-05-30 at 11.14.08 AM.png](/assets/images/screen-shot-2019-05-30-at-11.14.08-am.png)

Woah! We hit something - `graham` and `GSo7isUM1D4`. Could it be that this is the username and password for SSH? I decided to exit from the shell and give it a shot as follows: `ssh graham@wordy`

![Screen Shot 2019-05-30 at 11.18.08 AM.png](/assets/images/screen-shot-2019-05-30-at-11.18.08-am.png)

Jackpot!

Still, I was adamant to further escalate my privileges on this box.

I started with checking out my `sudo` as `graham` using the command: `sudo -l`

![Screen Shot 2019-05-30 at 11.22.27 AM.png](/assets/images/screen-shot-2019-05-30-at-11.22.27-am.png)

Interesting. We can run the Bash script - `/home/jens/backups.sh` as the user `jens`

But first, I decided to take a look at the script and check out its commands as follows: `cat /home/jens/backups.sh`

![Screen Shot 2019-05-30 at 11.28.30 AM.png](/assets/images/screen-shot-2019-05-30-at-11.28.30-am.png)

Woah! Line number 3... `/bin/bash` means that it opens a shell. To put it all into perspective, if we run `/home/jens/backups.sh` as the user `jens`, then it opens a shell and we will be user `jens`.

I decided to run the script as follows: `sudo -u jens /home/jens/backups.sh`

![Screen Shot 2019-05-30 at 11.31.04 AM.png](/assets/images/screen-shot-2019-05-30-at-11.31.04-am.png)

Yeah! Time to check my `sudo` privileges as `jens` using the command: `sudo -l`

![Screen Shot 2019-05-30 at 11.34.27 AM](/assets/images/screen-shot-2019-05-30-at-11.34.27-am.png)

Nmap! `jens` can run `/usr/bin/nmap`  as `root`.

This is our shot at privilege escalation to `root` because of known techniques which exist for `nmap`.

I came across [this webpage](https://gtfobins.github.io/gtfobins/nmap/) which details how to achieve a shell using `nmap`. The code to achieve this is as follows:

```
TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
nmap --script=$TF
```

![Screen Shot 2019-05-30 at 11.52.04 AM.png](/assets/images/screen-shot-2019-05-30-at-11.52.04-am.png)

Woohoo! we got `root`!

Time to _capture the flag_...

![Screen Shot 2019-05-30 at 11.54.10 AM.png](/assets/images/screen-shot-2019-05-30-at-11.54.10-am.png)

Finally! We did it!

### My Thoughts

Such an awesome challenge. Huge props to @DCAU for creating an interesting challenge revolving around WordPress!

In my opinion, **DC: 6** provided me a huge opportunity to practice my Linux skills especially the _privilege escalation_ parts.

I look forward to solving and writing-up the rest of challenges in the [DC series](https://www.vulnhub.com/series/dc,199/)!
