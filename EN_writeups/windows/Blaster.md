# **Blaster**

![](https://i.imgur.com/sjlFQ4w.png)

*Plateform* - [**TryHackMe**](https://tryhackme.com/room/blaster)

*OS* - **Windows**

*Rating* - **Easy**

*I am Khan*

## **Step 1 : RECONNAISSANCE**

### *General information*

There isn't much information we can get at this point. However, the box's description indicates it is a Windows machine.

## **Step 2 : SCANNING AND ENUMERATION**

> At this step, we take a look at a hidden directory on the web server. From this directory, we find a set of credentials that we will use at step 3.

### *Nmap Output*

I ran a very standard scan: `nmap -A -T4 -p- -oN <output> -v blaster.thm`

![nmap output](https://i.imgur.com/GEFba6w.png)

We have a lot of results. It would not be efficient to list everything as its own section, so I will focus on the main ports of interest: port 3389 and port 80.

### *PORT 3389 - RDP*

Similar to SSH on the Linux side, RDP is not usually a way in on its own. However, as we make progress and find credentials, it may be worth considering as a way in.

### *PORT 80 - HTTP*

#### A look at the website

Traveling to the web server, we can believe, at first glance, that nothing interesting is here. The IP address simply leads to a default IIS Windows Server page.

![Default IIS page](https://i.imgur.com/AbL5HXN.png)

#### Running DirBuster

The next step in our enumeration process is to see if there's anything else on this web server by fuzzing it. Using DirBuster, we get an interesting result: a **/retro** directory.

![Discovery of the /retro directory using DirBuster](https://i.imgur.com/9FZwonl.png)

#### Enumerating the /retro directory

This directory seems to be where the website is hosted. We see an old-fashioned style of website (read: retro) which is quite appealing.

Already we get a potential username from reading the author of the topmost blog post: User `Wade`.

![Discovery of a potential user : Wade](https://i.imgur.com/DcSecAG.png)

If we scroll down to the 2nd ever blog post, we discover a potential password :

![Discovery of a potential password : parzival](https://i.imgur.com/A15OPrq.png)

After a quick Google search (because my movie culture is clearly lacking), we get a name: `parzival`. If we put this information in the blog post's context, this seems to be a password.

We now have credentials : `Wade:perzival`


## **Step 3 : EXPLOITATION**

> Here, we will gain access to the machine using RDP and credentials found at step 2.

### *Getting access to the machine using RDP*

Remember the RDP service hosted on port 3389 we covered in *Step 2 - Scanning & Enumeration*? It's time to put it to use with our newly found credentials. 

### *Using Remmina to gain a foothold on the machine*

As it was the recommended way, I chose to use `remmina` to remotely connect to the machine.

First, get it installed : `apt install remmina`.

Next, start it by typing `remmina` in your console.

Then, connect to it by providing the required information.

![Logging in remotely using Remmina](https://i.imgur.com/aRFgJgJ.png)

That's it! We are now connected to the machine as user `Wade`! We didn't end up in the Oasis, but this will have to do.

![We are in!](https://i.imgur.com/N0XFih9.png)

![whoami results](https://i.imgur.com/nN6buKL.png)

**We are in! Easy, wasn't it?**

## **Step 4 : POST-EXPLOITATION**

> After exploiting an interesting privesc method using a badly hidden file and cheating a little bit, we will be NT Authority\System !

### *Enumerating the Wade user*

After seeing the Desktop screenshot in the previous step, I am sure you have a burning question: what is inside the recycle bin? Well, wait no further. Here is the result:

![What's inside the recycle bin?](https://i.imgur.com/jf6k3wK.png)

Disappointed? I hope not! If we google `hhupd.exe exploit`, we can quickly find this is referring the [CVE-2019-1388](https://nvd.nist.gov/vuln/detail/CVE-2019-1388) which is a "Windows Certificate Dialog Elevation of Privilege Vulnerability".

### *Getting root using CVE-2019-1388*

In order to privesc using what we've learned, I followed along [Zero Day Initiative's video](https://www.youtube.com/watch?v=3BQKpPNlTSo) on the matter. He provides a very detailed step-by-step guide. 

Here is how it goes.

#### 1. Get the hhupd.exe file out of the recycle bin

![Get the .exe out of the recycle bin](https://i.imgur.com/Llp3CpR.png)

#### 2. Run the hhupd.exe file as admin

![Running hhupd.exe as admin](https://i.imgur.com/idAQNvV.png)

#### 3. From the password prompt, select "Show more details"

![Showing more details](https://i.imgur.com/O3ymyiH.png)

#### 4. Select "Show information about the publisher's certificate"

![Showing information about the publisher's certificate](https://i.imgur.com/XL0etKO.png)

#### 5. Select VeriSign Commercial Software Publisher CA

![Clicking on the CA](https://i.imgur.com/Wq1JnNl.png)

Once you've clicked on it, simply click "OK" at the bottom.

#### 6. Close the admin prompt

![Closing the admin prompt](https://i.imgur.com/qMbKWqg.png)

We should now have a web page opened on the desktop. This is a key element, as this **webpage was opened as admin**.

#### 7. Open an explorer window by "saving the page"

!["Saving the page" to open an explorer](https://i.imgur.com/BCTmHRs.png)

#### 8. Travel to the System32 folder using the explorer

First, close the popup warning you that your *Location is not available*.

Then, travel to the System32 folder by typing in the **File name** field the following: `C:\Windows\System32\*.*`

![Traveling to System32](https://i.imgur.com/yW9XwYc.png)

Once you press "save", you should be inside System32.

#### 9. Open a command prompt from the high-privilege explorer and get root!

![Opening a command prompt as admin](https://i.imgur.com/PIZ9VlJ.png)

#### Pwned! We are NT Authority\System!

![Rooted!](https://i.imgur.com/Ebyg7du.png)

## **SUMMARY**

This box was special, because it was the first box I've done that would make us remotely connect to an actual desktop - not to a CLI. The fact this box leads us to a GUI changes how we should perceive post-exploitation enumeration. "Think like a user" -- Check the bin, check the internet history, check the logged accounts in the web browser, etc.

This was a very nice box.

Now, I need to go watch *Ready Player One* and improve my movie culture :)

# **That's it for now. Thank you for reading this write-up! :)**

### **Vonshad**
