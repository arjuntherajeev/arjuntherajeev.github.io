---
full_title: "Entry 0x02 - VulnHub: The Necromancer: 1 Walkthrough"
date: "2019-05-26"
coverImage: "necromancer-00.png"
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
---

**The Necromancer: 1** is a challenge posted on [VulnHub](https://www.vulnhub.com/entry/the-necromancer-1,154/) created by [Xerubus](https://www.vulnhub.com/author/xerubus,117/). This is a write-up of my experience solving this awesome CTF challenge.


With my Attack Machine (**Kali Linux**) and Victim Machine (**Necromancer**) set up and running, I decided to get down to solving this challenge.


> Read more about my set up and environment [here](/blog/my-first-post)


I decided to start my journey with the classic `netdiscover`. My Kali Linux VM has an IP of `192.168.56.102` according to `ifconfig`. With this knowledge and the fact that I set up **host-only networking** for my VMs, I decided to run the following command: `netdiscover -r 192.168.56.0/24`.


![Screen Shot 2019-05-26 at 4.35.00 PM](/assets/images/screen-shot-2019-05-26-at-4.35.00-pm.png)


Now we can see that the IP of the victim machine is `192.168.56.101`. Great!


The next step involves finding a way to communicate with the Victim Machine. I decided to use the trusty `nmap` with an aim to find any open ports on the machine.


Let's start with the _common_ ports: `nmap -F 192.168.56.101`


![Screen Shot 2019-05-26 at 4.41.29 PM](/assets/images/screen-shot-2019-05-26-at-4.41.29-pm.png)


Hmm, that's strange. No open ports. What if some obscure port is open? The only way is to scan _every_ port from 1-65535: `nmap -p- 192.168.56.101`


Gee, still nothing. I decided to explore other avenues. There just _had_ to be either some channel of communication or existing communication between the machines.


With this is mind, I decided to fire up **Wireshark** and capture network traffic on `eth0`.


![Screen Shot 2019-05-26 at 4.53.50 PM](/assets/images/screen-shot-2019-05-26-at-4.53.50-pm.png)



The packet capture on Wireshark showed us traffic from the Victim Machine (`192.168.56.101`) to our Attack Machine (`192.168.56.102`) to destination port `4444`.


Okay, there's some communication going on. But what exactly? I needed a way to _listen_ for the traffic to Kali Linux on port `4444`. I decided to go with netcat: `nc -lvp 4444`


![Screen Shot 2019-05-26 at 7.48.55 PM](/assets/images/screen-shot-2019-05-26-at-7.48.55-pm.png)

Boom! We got something... This looks like **Base 64** encoding to me. I tried to decode it using the terminal itself with the handy `base64 -d` command: `echo <message> | base64 -d`



![Screen Shot 2019-05-26 at 8.00.02 PM](/assets/images/screen-shot-2019-05-26-at-8.00.02-pm.png)


Yes! The Necromancer has a message for us! If we read it carefully, we can pick up 2 important points:


1. The first flag (flag 1) is `e6078b9b1aac915d11b9fd59791030bf`
2. A clue on how to proceed - `Chant the string of flag1 - u666`

Easy, `the string of flag1` must refer to the flag and `u666` must refer to port `666` via UDP (User Datagram Protocol). I decided to continue using netcat: `echo e6078b9b1aac915d11b9fd59791030bf | nc -u 192.168.56.101 666`

![Screen Shot 2019-05-26 at 8.10.56 PM](/assets/images/screen-shot-2019-05-26-at-8.10.56-pm.png)

Oh boy, that never worked. Try in a _different tongue_? What do you mean, Necromancer?

This string wasn't Base 64 encoding. It looked like a hash to me. With the help of my trusty [Hash Analyzer](https://www.tunnelsup.com/hash-analyzer/), it was clear that the flag was an MD5 or MD4 hash. Okay, we need to _decrypt_ the hash. No big deal.

Well, I was not really going to decrypt it. Instead, I went over to my trusted [MD5 online website](https://www.md5online.org/md5-decrypt.html) in hopes of finding the matching hash in their database.

![Screen Shot 2019-05-26 at 8.18.21 PM](/assets/images/screen-shot-2019-05-26-at-8.18.21-pm.png)

Yay! The _un-hashed_ string is `opensesame`.

I proceeded to send _this_ to the Necromancer using netcat: `echo opensesame | nc -u 192.168.56.101 666`

![Screen Shot 2019-05-26 at 8.21.42 PM](/assets/images/screen-shot-2019-05-26-at-8.21.42-pm.png)

Awesome. We got the second flag. Flag 2 is `c39cd4df8f2e35d20d92c2e44de5f7c6`.

But **wait**, there's another clue hidden in the message - `Squinting your eyes from the light coming from the object, you can see the formation looks like the number 80.`

Almost instantly I thought of HTTP (port `80`). Did the message mean that port 80 is now open for traffic? The last line in the message increased my confidence: `666 is closed.`

I decided to verify my hunch using `nmap`. But this time I chose to be _specific_ with port `80`: `nmap -p 80 192.168.56.101`

![Screen Shot 2019-05-26 at 8.36.27 PM.png](/assets/images/screen-shot-2019-05-26-at-8.36.27-pm.png)

Yes, port `80` was open for business. Time to hit the browser!

![Screen Shot 2019-05-26 at 8.42.18 PM](/assets/images/screen-shot-2019-05-26-at-8.42.18-pm.png)

First things first, I checked the source code (`CTRL + U`). It did not reveal anything exciting except: `<img src="/pics/pileoffeathers.jpg">`

Hmm, an image file. I decided to download this _image_ and dissect. Who knows what treasures could be hidden...

I decided to start with the `file` command: `file pileoffeathers.jpg`

![Screen Shot 2019-05-26 at 9.20.39 PM](/assets/images/screen-shot-2019-05-26-at-9.20.39-pm.png)

Nothing exciting here. I decided to try again with the `binwalk` tool: `binwalk pileoffeathers.jpg`

![Screen Shot 2019-05-26 at 9.23.19 PM](/assets/images/screen-shot-2019-05-26-at-9.23.19-pm.png)

Oh my! I can see the words `Zip archive data`. Some more details follow:

- Compressed size: `121`
- Uncompressed size: `125`
- Name: `feathers.txt`

Obviously, I proceeded with unzipping this _pseudo_ image.

![Screen Shot 2019-05-26 at 9.27.40 PM](/assets/images/screen-shot-2019-05-26-at-9.27.40-pm.png)

Let's have a look at `feathers.txt`

![Screen Shot 2019-05-26 at 9.29.43 PM](/assets/images/screen-shot-2019-05-26-at-9.29.43-pm.png)

Once again, Base 64: `ZmxhZzN7OWFkM2Y2MmRiN2I5MWMyOGI2ODEzNzAwMDM5NDYzOWZ9IC0gQ3Jvc3MgdGhlIGNoYXNtIGF0IC9hbWFnaWNicmlkZ2VhcHBlYXJzYXR0aGVjaGFzbQ==`

Decoding this Base 64 string gave me: `flag3{9ad3f62db7b91c28b68137000394639f} - Cross the chasm at /amagicbridgeappearsatthechasm`

Woo! We got the third flag. Flag 3 is `9ad3f62db7b91c28b68137000394639f` and a hint on how to proceed. `Cross the chasm at /amagicbridgeappearsatthechasm` must refer to a URL. Back to the browser with `http://192.168.56.101/amagicbridgeappearsatthechasm`

![Screen Shot 2019-05-26 at 9.48.22 PM](/assets/images/screen-shot-2019-05-26-at-9.48.22-pm.png)

You know the drill. Look at source code, download the image, `binwalk` it and beat the Necromancer. Only, there was absolutely nothing! Did the Necromancer get the best of me?

In an attempt to beat the Necromancer, I decided to run the trust `dirb` (a web content scanner). You know... in case a hidden URL is revealed or something. 

![Screen Shot 2019-05-26 at 10.04.17 PM](/assets/images/screen-shot-2019-05-26-at-10.04.17-pm.png)

There appears to be nothing apart from `index.html`. I did not bother trying with another wordlist. Back to the message... 

`There must be a magical item that could protect you from the necromancer's spell.`

Hmm, I never really knew much about Necromancers or _magical items_ that could defeat them. Google to the rescue!

With lots of reading, trial and error, and plain guessing... I came up with the word `talisman`.

> What's a _talisman_? According to Google, _an object, typically an inscribed ring or stone, that is thought to have magic powers and to bring good luck._

Okay, we have the _magical item that could protect us from the necromancer's spell_. How do I use it?

Practically speaking, there is no form or inputs on the HTML web page.

Hence, I decided to manipulate the URL: `http://192.168.56.101/amagicbridgeappearsatthechasm/talisman`

![Screen Shot 2019-05-26 at 11.15.29 PM.png](/assets/images/screen-shot-2019-05-26-at-11.15.29-pm.png)

Well look what we have here! A binary file!

After setting the right permissions (`chmod +x talisman`), I ran it...

![Screen Shot 2019-05-26 at 11.18.55 PM.png](/assets/images/screen-shot-2019-05-26-at-11.18.55-pm.png)

Okay. So, the program demanded user input. `Do you want to wear the talisman?`

I typed `yes` and the program exited with the message: **Nothing happens**. I tried other strings and words but nothing worked. Time to reverse engineer this program...

In my past experiences with binary exploitation, I have used `gdb`. I decided to fire up `gdb` with the `talisman` program: `gdb ./talisman`

![Screen Shot 2019-05-27 at 2.41.00 PM.png](/assets/images/screen-shot-2019-05-27-at-2.41.00-pm.png)

Now, I wanted to have a look at the `main()` function which is the entry point of any program. Inside `gdb`, I typed: `disassemble main`

![Screen Shot 2019-05-27 at 2.42.47 PM](/assets/images/screen-shot-2019-05-27-at-2.42.47-pm.png)

A quick read through the **Assembly** code reveals a function call to the function `wearTalisman()`. Typically, I have seen programs which have some `if-conditional` as follows:

```
if input == password:
    call correct()
else:
    call incorrect()
```

However, in the above Assembly code, I could not see any `CMP` (compare) or `JMP` (jump) Assembly instructions. I concluded that there is **no** checking user input.

What next? I decided to review the functions within the program. This can be done within `gdb` using: `info functions`

![Screen Shot 2019-05-27 at 2.49.06 PM.png](/assets/images/screen-shot-2019-05-27-at-2.49.06-pm.png)

Interesting. The function `chantToBreakSpell()` looks like it might be the solution. Now, if `main()`is making a call to `wearTalisman()`... how about we break at `main()` and simply call `chantToBreakSpell()` instead?

I decided to set a breakpoint on `main()` as follows: `break main` Then, I ran the program with the command: `run`

![Screen Shot 2019-05-27 at 2.53.55 PM.png](/assets/images/screen-shot-2019-05-27-at-2.53.55-pm.png)

Okay, it hit the breakpoint. The program is at `main()`. I tried making a call to `chantToBreakSpell()` as follows: `jump chantToBreakSpell`

![Screen Shot 2019-05-27 at 2.56.02 PM.png](/assets/images/screen-shot-2019-05-27-at-2.56.02-pm.png)

Wow! We got the fourth flag. Flag 4 is `ea50536158db50247e110a6c89fcf3d3`. There's more... a hint to proceed... `Chant these words at u31337`.

I decided to work with my trusty netcat once again: `echo ea50536158db50247e110a6c89fcf3d3 | nc -u 192.168.56.101 31337`

![Screen Shot 2019-05-27 at 3.00.39 PM](/assets/images/screen-shot-2019-05-27-at-3.00.39-pm.png)

MD5? Again? Back to my trusted [MD5 online website](https://www.md5online.org/md5-decrypt.html) in hopes of finding the matching hash in their database. ![Screen Shot 2019-05-27 at 3.02.21 PM.png](/assets/images/screen-shot-2019-05-27-at-3.02.21-pm.png)

Found it - `blackmagic`

![Screen Shot 2019-05-27 at 3.03.53 PM.png](/assets/images/screen-shot-2019-05-27-at-3.03.53-pm.png)

Another flag! We got the fifth flag. Flag 5 is `0766c36577af58e15545f099a3b15e60`. We also got a clue on how to proceed:

`Above them, a word etched in blood on the wall.` `/thenecromancerwillabsorbyoursoul`

Based on what we saw earlier, I guess that this belongs in a URL as follows: `http://192.168.56.101/thenecromancerwillabsorbyoursoul`

![Screen Shot 2019-05-27 at 3.10.08 PM](/assets/images/screen-shot-2019-05-27-at-3.10.08-pm.png)

Wait... there's more if you scroll down...

![Screen Shot 2019-05-27 at 3.11.09 PM.png](/assets/images/screen-shot-2019-05-27-at-3.11.09-pm.png)

We got the sixth flag! Flag 6 is `b1c3ed8f1db4258e4dcb0ce565f6dc03`.

Reading through the message reveals 2 important points:

1. In the sentence - `It's the necromancer!` there is a reference link in the word `necromancer` to 
```
http://192.168.56.101/thenecromancerwillabsorbyoursoul/necromancer
``` 
which will download a binary file.
2. In the sentence -  `Looking closer at the skill, you can see u161 engraved into the forehead.` there is a hint that some traffic is passing through UDP port `161`.

I decided to start with the `necromancer` binary file. I changed the permission to executable using `chmod +x necromancer` and ran it using: `./necromancer`

![Screen Shot 2019-05-27 at 3.59.43 PM.png](/assets/images/screen-shot-2019-05-27-at-3.59.43-pm.png)

Uh oh! Not a binary? What?

![Screen Shot 2019-05-27 at 4.00.44 PM.png](/assets/images/screen-shot-2019-05-27-at-4.00.44-pm.png)

Bzip2 compression! Not a problem. Easy enough to decompress it with: `bzip2 -d necromancer`

![Screen Shot 2019-05-27 at 6.37.24 PM](/assets/images/screen-shot-2019-05-27-at-6.37.24-pm.png)

Okay, we now have `necromancer.out`. Let's see what is inside...

![Screen Shot 2019-05-27 at 6.38.51 PM.png](/assets/images/screen-shot-2019-05-27-at-6.38.51-pm.png)

Maybe the Necromancer can read that. I couldn't!

I decided to investigate the file further using the `file` command: `file necromancer.out`

![Screen Shot 2019-05-27 at 6.40.43 PM.png](/assets/images/screen-shot-2019-05-27-at-6.40.43-pm.png)

A POSIX tar archive, huh? Naturally, I unzipped it to find a `.cap` file.

I decided to open `necromancer.cap` with my trusty friend, Wireshark using: `wireshark necromancer.cap`

![Screen Shot 2019-05-27 at 6.56.23 PM.png](/assets/images/screen-shot-2019-05-27-at-6.56.23-pm.png) Looking at the packet capture, I understood that it was Wi-Fi traffic. With a `.cap` file and the knowledge that Wi-Fi traffic exists within it, I decided to go with my trusty tool - `aircrack-ng` with hopes of breaking the password.

Since we are cracking passwords, we need a proper wordlist. I decided to go with the infamous `rockyou.txt`. My only concern with `rockyou.txt` is that it is huge and the brute-force could take ages.

I proceeded with `aircrack-ng` as follows: `aircrack-ng -w /usr/share/wordlists/rockyou.txt necromancer.cap`

![Screen Shot 2019-05-27 at 6.55.46 PM.png](/assets/images/screen-shot-2019-05-27-at-6.55.46-pm.png)

Well Necromancer, `death2all`... really?

I got the password but was unsure what exactly to do with it. All this while, we have ignored the second clue -`Looking closer at the skill, you can see u161 engraved into the forehead.`

Something about `u161` bothered me. I tried communicating the password to that port as follows: `echo death2all | nc -u 192.168.56.101 161` ... but got nothing!

It was then I decided to run `nmap` again... just for the heck of it as follows: `nmap -p 161 192.168.56.101`

![Screen Shot 2019-05-27 at 7.09.50 PM.png](/assets/images/screen-shot-2019-05-27-at-7.09.50-pm.png)

That's when it hit me. I was dealing with `SNMP`.

I am going to admit. I did not know much about `SNMP` and its relevance to CTFs so I spent some time reading `snmp-check` and `snmpwalk` which are tools provided in Kali Linux. I decided to give `snmp-check` a try with different community strings...

![Screen Shot 2019-05-27 at 7.17.38 PM.png](/assets/images/screen-shot-2019-05-27-at-7.17.38-pm.png)

Oho, `death2all` was the community string after all!

The message tells us us that _the door_ is locked and it must be unlocked. There is a unique word - `death2allrw`. Is that another community string?

![Screen Shot 2019-05-27 at 7.22.54 PM.png](/assets/images/screen-shot-2019-05-27-at-7.22.54-pm.png)

Jackpot!

That's a lot of information. I felt like a kid inside a candy shop...

Not for long though. The message still haunted me - `The door is Locked. If you choose to defeat me, the door must be Unlocked.`

How can I unlock the door, Necromancer? Time to Google!

With lots of Googling and reading ([this](https://penturalabs.wordpress.com/2010/10/07/secure-your-snmp/), [this](https://tools.kali.org/information-gathering/snmp-check), and [this](https://www.comparitech.com/net-admin/snmpwalk-examples-windows-linux/)), I learnt about `SNMP` and `MIB`. Basically, there is a way to change information displayed in the `MIB` using the `snmpset` command. However, this requires something called `ObjectID` or `OID` of the field we wish to change.

In the above screenshot, I figured that we needed to change the `Location` field from `Locked` to `Unlocked`. But I could not spot its `OID`. Hence, I decided to go with `snmpwalk` instead as follows: `snmpwalk -c death2allrw 192.168.56.101`

This gave me an error: `snmpwalk: No securityName specified`

With some Googling, I found [this](https://www.svennd.be/snmp-error-snmpwalk-no-securityname-specified/) and tried with a different version: `snmpwalk -v 2c -c death2allrw 192.168.56.101`

![Screen Shot 2019-05-27 at 8.43.29 PM.png](/assets/images/screen-shot-2019-05-27-at-8.43.29-pm.png)

Got it! The `OID` for the `Location` field is `iso.3.6.1.2.1.1.6.0`. Time to use `snmpset` as follows: `snmpset -v 2c -c death2allrw 192.168.56.101 .1.3.6.1.2.1.1.6.0 s Unlocked`

![Screen Shot 2019-05-27 at 8.51.15 PM.png](/assets/images/screen-shot-2019-05-27-at-8.51.15-pm.png)

Awesome! That command took tons of Googling and trial-and-error.

Now that we _unlocked the door_, I decided to check the `SNMP` information again using `snmp-check`:

![Screen Shot 2019-05-27 at 8.53.47 PM.png](/assets/images/screen-shot-2019-05-27-at-8.53.47-pm.png)

Finally, the seventh flag! Flag 7 is `9e5494108d10bbd5f9e7ae52239546c4` and a clue - `t22` which obviously refers to TCP port `22`. Hello SSH, my old friend!

With all the excitement, the first thing I tried was to log in to SSH as `root` with the flag as the password...

![Screen Shot 2019-05-27 at 8.59.04 PM](/assets/images/screen-shot-2019-05-27-at-8.59.04-pm.png)

Obviously, this never worked.

I decided that I needed to figure out a username for my SSH adventure. After some more trial and error, I decided the brute-force was the way to go. Hello `rockyou.txt`!

I went with my trusty `hydra` to brute-force the **username** as follows: `hydra -L /usr/share/wordlists/rockyou.txt -p 9e5494108d10bbd5f9e7ae52239546c4 192.168.56.101 ssh`

While waiting for the brute-force to finish, it hit me that the flag is an MD5 string. So I went back to my trusted [MD5 online website](https://www.md5online.org/md5-decrypt.html) in hopes of finding the matching hash in their database and I did....

`9e5494108d10bbd5f9e7ae52239546c4 = demonslayer`

Here goes `hydra`: `hydra -L /usr/share/wordlists/rockyou.txt -p demonslayer 192.168.56.101 ssh`

Without much luck on the username... I decided that I was probably doing it wrong. Maybe `demonslayer` is the username?

I decided to fire up `hydra` again with `rockyou.txt` as my wordlist as follows: `hydra -l demonslayer -P /usr/share/wordlists/rockyou.txt 192.168.56.101 ssh`

![Screen Shot 2019-05-27 at 9.08.29 PM.png](/assets/images/screen-shot-2019-05-27-at-9.08.29-pm.png)

Got it! `12345678` ... really `demonslayer`? Time to SSH!

![Screen Shot 2019-05-27 at 9.10.01 PM.png](/assets/images/screen-shot-2019-05-27-at-9.10.01-pm.png)

I have to say... it was a proud moment for me!

So... `whoami`? Well, I am `demonslayer` with a mission to _escalate_ my privileges and get `root`.

First things first... I decided to run an `ls -l` in my current directory:

![Screen Shot 2019-05-27 at 9.14.17 PM.png](/assets/images/screen-shot-2019-05-27-at-9.14.17-pm.png)

The `ls -l` revealed `flag8.txt` and a peek at its contents revealed something interesting - `Defend yourself! Counter attack the Necromancer's spells at u777!`

I decided to go with netcat as follows: `nc -u localhost 777`

![Screen Shot 2019-05-27 at 9.28.00 PM.png](/assets/images/screen-shot-2019-05-27-at-9.28.00-pm.png)

I messed up! Turns out the Necromancer is super serious because I couldn't even connect to SNMP anymore. Time to go backwards...

After a painful trip backwards and running way too many old commands, I decided to give the Q and A session another try. This time, I kept Google really handy!

![Screen Shot 2019-05-27 at 9.38.38 PM.png](/assets/images/screen-shot-2019-05-27-at-9.38.38-pm.png)

Got it! A close call there but boy was that hard!

We can see the tenth flag. Flag 10 is `8dc6486d2c63cafcdc6efbba2be98ee4`.

Back to trying privilege escalation... My Linux senses were tingling. One of the most common things I try is checking my `sudo` privileges as a user. This is done as follows: `sudo -l`

![Screen Shot 2019-05-27 at 9.49.40 PM.png](/assets/images/screen-shot-2019-05-27-at-9.49.40-pm.png)

Wow! Are you seeing what I am seeing? `(ALL) NOPASSWD: /bin/cat /root/flag11.txt`

This means that we can run `/bin/cat /root/flag11.txt` as `root`... and that's what I did next: `sudo cat /root/flag11.txt`

![Screen Shot 2019-05-27 at 9.51.49 PM.png](/assets/images/screen-shot-2019-05-27-at-9.51.49-pm.png)

Finally! We got the eleventh and final flag. Flag 11 is `42c35828545b926e79a36493938ab1b1`

For what it was worth... flag 11 decrypts to `hackergod`.

#### My Thoughts

That was amazing. Seriously, I had a blast and learnt a plethora of things by solving this challenge. I also learnt a little bit about the Necromancer. He's a scary guy.

I have to admit that certain parts (like the `SNMP`) were challenging for me because of my lack of experience with `snmpwalk`, `snmp-check`, and `snmpset` but it was a great learning experience.

I respect [Xerubus](https://www.vulnhub.com/author/xerubus,117/) for helping me keep my sanity with the brute-force parts. The wordlist `rockyou.txt` is pretty long and agonizing to wait until completion. Passwords like `12345678` take seconds to find!
