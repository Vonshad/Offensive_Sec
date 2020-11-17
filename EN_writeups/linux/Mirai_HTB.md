# **Mirai**

![Mirai Info Card](https://i.imgur.com/jndB9lZ.png)

*Plateform* - [**HackTheBox**](https://www.hackthebox.eu/home/machines/profile/64)

*OS* - **Linux**

*Rating* - **Easy**

*Similar to a creeper, Mirai will make your Minecraft server explode.*

## **Step 1 : RECONNAISSANCE**

### *Looking at the box info*

The first reconnaissance step we can take is checking out the box info card.

![Info card](https://i.imgur.com/f2WGzLd.png)

Already, we can see there are more users owns than system owns, which indicates we won't log to the machine as root. However, since the difference isn't that big, I am guessing it is quite straightforward to privesc.

Furthermore, since the box is named [Mirai](https://en.wikipedia.org/wiki/Mirai_(malware)), an IoT botnet, we can guess we will be attacking IoT devices. However, we can't be sure quite yet.


## **Step 2 : SCANNING AND ENUMERATION**

> Here, we will take a look at a [pi-hole](https://pi-hole.net/) and will find default credentials for this service.

### *Nmap Output*

I used a standard full-range scan: `nmap -sC -sV -oN mirai_full -p- -v mirai.htb `

![Nmap output](https://i.imgur.com/CxRgaUn.png)

For this write-up, I will focus on port 22 and port 80.

### *PORT 22 - SSH*

While SSH isn't usually a way in on its own, it is important to note the service is available. That way, if we ever find credentials, we can use them in order to gain access to the machine through SSH (*foreshadowing*).

### *PORT 80 - HTTP*

#### A look at the website 

Traveling to the HTTP service, we find we are blocked by a [pi-hole](https://pi-hole.net/), which is a network-wide ad blocking that is set up using a [Raspberry Pi](https://www.raspberrypi.org/).

![Pi-hole screenshot](https://i.imgur.com/md5So8P.png)

The first thing we can learn from this page is that it is running Pi-hole v.3.1.4. If we query searchsploit, we will get a few results.

![Searchsploit results for Pi-hole](https://i.imgur.com/aqNqBqh.png)

However, we can notice that everything that could be of interest to us requires us to be authenticated before we can use it. Let's go another way and run **DirBuster**.

#### Running DirBuster

Running DirBuster is almost always a great idea. DirBuster allows us to find hidden directories, thus could lead us to incredible findings. Here are the results we got:

![DirBuster results](https://i.imgur.com/gL3p3df.png)

An admin directory! Interesting, let's take a look.

#### Visiting the /admin/ directory

![/admin/ directory](https://i.imgur.com/B9OiY7c.png)

We are able to view the pi-hole dashboard. Most importantly, we see a login page on the left. 

If we access it, we get a login prompt.

![Login prompt of /admin/](https://i.imgur.com/vDljy69.png)

This prompted me to look for default credentials used for pi-holes. After a bit of google-fu, I came across these [instructions](https://blog.cryptoaustralia.org.au/instructions-for-setting-up-pi-hole/) to set up pi-hole. The step I'm mostly interested in is step 4:

![Step 4 of the instructions](https://i.imgur.com/gG7E5Xs.png)

Now we are talking! We have a set of default crendials ***AND*** we can use them to SSH in the machine? 

**Count me in. Let's start exploiting.**

## **Step 3 : EXPLOITATION**

> Using the newly found credentials, we SSH in the machine.

### *Using SSH to get in the machine*

Now, there is no real way to justify such a lack of security. Being able to simply SSH into a machine using default credentials is, plainly put, very bad. However, we aren't blue team today, so we should be very happy about this little gift.

Let's try getting in the machine.

![Successful SSH attempt](https://i.imgur.com/sa2FNtc.png)

Simple, wasn't it? We are now connected as user `pi` and are a full user! 

**We are in. Let's see what we can do to improve our privileges.**


## **Step 4 : POST-EXPLOITATION**

> To privesc, we will do basic enumeration and realize we can abuse Sudo in order to gain root. We also work to get our flags.

### *Getting the user.txt flag*

While I don't normally provide flags in write-ups, this box gets tricky with the **root.txt** flag, so I decided I would include them in this write-up. First off, let's get the **user.txt** flag. This flag is located in `/home/pi/Desktop/`.

![user.txt flag](https://i.imgur.com/SqV4Jhf.png)

### *Getting root*

Once we have the **user.txt** flag, we will focus on trying to find a way to privilege escalate to root.

#### Basic enumeration of the `pi` user

Running basic commands for privilege escalation (see my [cheatsheet](https://github.com/Vonshad/Offensive_Sec/blob/main/Linux-PrivEsc-cheatsheet.md) if you want ;) ), we will run `sudo -l` at some point.

![sudo -l results](https://i.imgur.com/xhJv2bM.png)

Our user can run ***any*** command as root! What a gift -- and it's not even the Holiday season!

#### Getting root

Using the `sudo` privileges we have, we can get root a thousand different ways. I went with a simple `sudo /bin/bash`.

![Getting root](https://i.imgur.com/0vwQ1TA.png)

Just like that, we are the root user. However, before we can call it a day, we need to get the root flag.

### *Getting the root.txt flag*

This flag is trickier than the usual ones. Let's get to it!

#### Reading /root/root.txt

The first step we will normally take in order to get the flag is locating the **root.txt** file. As usual, it is inside the `/root/` folder. Let's take a look at it.

![root.txt file](https://i.imgur.com/4PePO9H.png)

Oh come on now! You lost it? Well, at least it'll be on the USB stick, right?

#### Reading the USB stick files

USB sticks will be in the `/media/` folder. In order to see what's in it, I ran a `ls -l /media` command.

![Content of /media/](https://i.imgur.com/zQafrvd.png)

All right, we are getting somewhere! What is on `/media/usbstick/`?

![Content of /media/usbstick/](https://i.imgur.com/gfd7ABm.png)

Oh, what now? Why is it named **damnit.txt**? Let's read it.

![Content of /media/usbstick/damnit.txt](https://i.imgur.com/p7AEj5j.png)

**James?** Who even are you and why are you deleting my flag?

Next step, recovering the flag.

### *Really getting the root.txt flag this time*

In order to recover the **root.txt** file that was lost, we will look at the **/dev/sdb** file. This is where our USB stick should be located, as it is *disk b*. Read more about [/dev/sdX](https://help.ubuntu.com/lts/installation-guide/armhf/apcs04.html).

#### Reading the /dev/sdb file and getting the root.txt flag

Run: `cat /dev/sdb`

![/dev/sdb content](https://i.imgur.com/mEeFELP.png)

This file is not very user-friendly, isn't it? While we did find the flag, it definitely isn't the cleanest way of retrieving it.

Using the `strings` command, we will get a cleaner output. `strings` will look through binary data in order to find human-readable text and will output it to the terminal.

Let's run it: `strings /dev/sdb`

![Results from strings /dev/sdb](https://i.imgur.com/JDS4Kn9.png)

That's it! We now have the flag and it is much cleaner.

**PWNED!**

## **SUMMARY**

While this box was easy, I guess we can take it as a warning about our minimally protected IoT devices. I strongly encourage you to read about the Mirai botnet or listen to [Jack Rhysider's Darknet Diaries episode on internet exposed objects](https://darknetdiaries.com/episode/31/). In this episode, Jack Rhysider and his guest discusses hacks on printers, Chromecasts, and more.

Furthermore, this box forced us to look at ways to recover a deleted file from a USB stick, which was a very nice way of challenging us before giving the root flag.

All in all, an easy box but one that proves to be interesting and forced us out of our usual comfort zones regarding flags.

# **That's it for now. Thank you for reading this write-up! :)**

### **Vonshad**
