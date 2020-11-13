# **Beep**

![Box picture](https://i.imgur.com/i7D1DSf.png)

*Plateform* - [**HackTheBox**](https://www.hackthebox.eu/home/machines/profile/5)

*OS* - **Linux**

*Rating* - **Medium**

*Sadly not a Road Runner themed box.*

## **Step 1 : RECONNAISSANCE**

### *Reading the box description*

From reading the box description, we learn this is a Linux machine and that we will most likely focus on CVE research.

![Reconnaissance on HTB](https://i.imgur.com/2Ltzq9a.png)


## **Step 2 : SCANNING AND ENUMERATION**

> At this stage, we get busy with the amount of ports opened on the machine. Lucky for us, only two ports need to be considered for this write-up. We enumerate an Elastix installation and find an awesome exploit. 

### *Nmap Output*

The nmap scan was a standard full-range scan. Nmap command: `nmap -sC -sV -oN <output_name> -v beep.htb`

![Nmap output](https://i.imgur.com/g4GylIa.png)

That's a lot of ports for a CTF machine! Luckily, our pentest will focus only on port 22 and port 443.

### *PORT 22 - SSH*

While it is important to take note that SSH is running on the machine, it usually isn't a way in on its own. However, if we find credentials, this could be a way in (*foreshadowing*).

### *PORT 443 - HTTPS*

Traveling to `https://beep.thm`, we are greeted by an [Elastix](https://www.elastix.org/) login prompt.

![Elastix login prompt](https://i.imgur.com/KtpfoH2.png)

#### Using DirBuster to find hidden directories

![DirBuster results](https://i.imgur.com/eSBGxAy.png)

Two results seem to be the most interesting: **/configs/** and **/admin/**. While I enumerated all of the directories, none of them will prove useful in the actual exploitation part. Still, I wanted to leave this part in the write-up as I believe it is a default section that needs to be left in.

#### Finding an exploit for the Elastix software

##### Using Searchsploit

Using searchsploit, we get a couple of hits for the Elastix software.

![Searchsploit results](https://i.imgur.com/8mzDldQ.png)

The one we will be interested in is the LFI exploit (Local File Inclusion). This LFI can be found on [ExploitDB](https://www.exploit-db.com/exploits/37637) if you are curious.

![LFI picture](https://i.imgur.com/8bu6pww.png)

##### Trying the LFI exploit and getting credentials

The exploit mentions we need **Elastix v.2.2.0**. However, I didn't find a version confirmation previously. Nevertheless, I decided to take a chance as it was a simple URL change I had to make and wouldn't make us waste a lot of time if it didn't work. Lucky for us, we won the jackpot and got a response:

![Response](https://i.imgur.com/T5Om7f8.png)

What an interesting page! Can't you see all the info it leaks? No? All right, let's make it prettier by doing `Right-click --> View Source`.

![Better Response](https://i.imgur.com/1OWSczr.png)

> I chose to hide the password, as it is the key to the whole box. You'll see.

Bingo! We have credentials! While we can use these credentials to log in to the **/admin/** dashboard, this box is actually much easier than that.

#### Finding users on the machine using the same exploit

Now that we know we can get file disclosures using this exploit, why not try to find users on the machine? By doing so, we will get usable SSH usernames.

In order to get usernames, we want to read the **/etc/passwd** file on the machine. To do so, all we have to do is modify our URL to specify the file location. See below.

![Getting machine users](https://i.imgur.com/1BDfrsF.png)

We now have two usernames that are present on the actual machine: `root` and `fanis`.

Let's move on to exploitation.

## **Step 3 : EXPLOITATION**

> Using a previously found username and password, we attempt to SSH into the machine and we make a very surprising discovery.

### *Attempting to SSH into the machine*

Remember port 22; the one we barely covered? It is now its time to shine. 

The password we found was the password of the `admin` user. Could this password be the same one the `root` user is using? Let's try!

Ready? *This could be it.*

Set? *Imagine if we log in as root and win this thing!*

Go! *Heart is pumping.*

![Failed SSH attempt](https://i.imgur.com/63sgzu1.png)

An ***error***? Disappointing. Let's try doing what it asks us to do.

*Get back up, reload, and go again.*

*Fire!*

![Successful SSH attempt](https://i.imgur.com/5gKgK0h.png)

Here is the final command: `ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7`

**We are in! Not only are we in the machine, we are in *AS ROOT***.

**PWNED!**

## **SUMMARY AND LESSONS LEARNED**

I definitely learned a lot doing this box and fell in some rabbit holes along the way (especially the admin dashboard -- I probably missed something there (looking at you Asterisk CLI), but it didn't stop me from rooting the box). 

I learned to take my time and read through the documentation: I actually found the LFI exploit a first time, but ditched it because I didn't take the time to really understand it. This box, like so many others, gave me the following reminder: **take your time. This is a marathon, not a race.** Once again, the enumeration process proves to be the most valuable, as it is the basis for your exploitation.

All in all, a very interesting and fast box once you know where you are going with it.

Bad joke of the day: *Maybe the true reason the box is named **Beep** is because, in the end, it was built with the Road Runner's image in mind?*

![Not so funny joke](https://i.imgur.com/tqMqpHT.png)

# **That's it for now. Thank you for reading this write-up! :)**

### **Vonshad**
