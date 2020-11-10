# **Daily Bugle**

![Box banner](https://i.imgur.com/JXkKIrJ.png)

*Plateform* - [**TryHackMe**](https://tryhackme.com/room/dailybugle)

*OS* - **Linux**

*Rating* - **Hard**

*When all you want is pictures of Spider Man, but they only bring you this box.*


## **Step 1 : RECONNAISSANCE**

### *Finding a login form and a potential username*

![Picture of the index.php](https://i.imgur.com/CUeCFxM.png)

Already, this box seems to be giving us great information. While it is important to take notes of these elements, they won't be of critical importance later on, so I will not be covering them extensively.


## **Step 2 : SCANNING AND ENUMERATION**

> Here, we find that the machine is running a webserver on which a CMS (Joomla! version 3.7.0) is running. We also find the /administrator dashboard (can't log in yet) on the webserver.

### *Nmap Output*

I ran the following command: `nmap -A -T4 -p- -oN <output_name> -v <Target_IP>`. This is a classic full-range scan.

![Picture of the Nmap output](https://i.imgur.com/LKZYEQg.png)

### *PORT 22 - SSH*

While it is important to take note that the SSH service is running on the target machine, it usually isn't a way in on its own.

### *Port 3306 - MySQL*

We learn MySQL is running on the target machine. We could potentially learn crucial information using [sqlmap](https://tools.kali.org/vulnerability-analysis/sqlmap), but as it is not allowed on the OSCP exam, I chose to avoid it.

### *Port 80 - HTTP*

This is the webserver we found during *Step 1 - Reconnaissance*. The Nmap results indicated to us it is running the **Joomla!** CMS and has disallowed entries in the **robots.txt** file.

#### A look at the website

The elements that can be found "as is" on the websites are the following: a potential username (**Super User**) and a login form. Attempting to use SQL injections won't be of any help and we can't seem to be able bypass the login form.

#### Robots.txt content

This robots.txt file validates that the website is running the Joomla! CMS and gives us the location of a few interesting folders, especially the **/administrator** one.

![Picture of the robots.txt file](https://i.imgur.com/fsKHuQF.png)

#### Visiting the /administrator folder

The /administrator folder prompts us to log in. SQL injections won't work here either. Even though we can't exploit it right now, this administrator panel will be our way in during *Step 3 - Exploitation*.

![Picture of the /administrator folder](https://i.imgur.com/n7X9fgt.png)

#### Running DirBuster to find new elements

After exploring the obvious options, it is time to bust some directories. At that point, I was mostly trying to find a CMS version in order to find exploits.

I used the **directory-list-2.3-medium.txt** wordlist and included **.php** and **.txt** files to the search. These were the results:

![dirbuster output](https://i.imgur.com/ilabr5a.png)

The only new and interesting element was the **README.txt** file.

#### Reading the README.txt file

README.txt content:

![README.txt content](https://i.imgur.com/cuAs0tQ.png)


The README.txt file proved to be very useful, as it showed us the CMS' version to be 3.7. Searching for `Joomla 3.7 exploits`, it is possible to find the [CVE-2017-8917](https://www.exploit-db.com/exploits/42033). This will give us the elements to break in.

Time to start attacking.


## **Step 3 : EXPLOITATION**

> To exploit, I used the Joomblah python script and found credentials. Using these credentials, it is possible to access the /administrator folder and, from there, exploit the "preview" feature of templates in order to get a reverse shell.

### *Exploiting CVE-2017-8917*

As found in *Step 2 - Scanning & Enumeration*, this version of the Joomla! CMS is vulnerable to SQL Injections ([CVE-2017-8917](https://www.exploit-db.com/exploits/42033)). After digging around a little bit, I found a python script that would help with this exploitation: [Joomblah by NinjaJc01](https://github.com/NinjaJc01/joomblah-3).

#### Running the joomblah.py script

From running this script, we found a SQL table named **fb9j5_users** which contained a hash. We can also notice the presence of a couple of users, including the one we found during *Step 1 - Reconnaissance*: `Super User`.

![Joomblah.py results](https://i.imgur.com/acx4yUa.png)

### *Cracking the hash*

Using [hashid.py](https://github.com/psypanda/hashID), we can guess that this is most likely a **bcrypt** hash (UNIX).

Once the hash type was known, I switched to my host OS in order to crack the hash using my GPU. For my wordlist, I used the very popular rockyou.txt (more info about this wordlist [HERE](https://www.kaggle.com/wjburns/common-password-list-rockyoutxt)).

![cracked hash and hashcat results](https://i.imgur.com/9J9WmwX.png)

**Bingo!** We have what seems to be a password.

### *Logging in to the /administrator dashboard*

Using a combination of usernames we found previously, it is possible to log in to the administrator dashboard. The user "jonah" worked for me and the password is the cracked hash.

After logging in, we can notice we are actually logged in as `Super User`! Hurray us! 

![We are Super User!](https://i.imgur.com/ZtFwByw.png)

Now, onto finding a way to get a reverse shell.

### Getting a reverse shell

Now, this process is a bit tricky and took me a while (and a lot of trial/error). Here is the step-by-step:

#### 1. Pre-exploitation

Get a php reverse shell. I personally like [this one from pentest monkey](https://github.com/pentestmonkey/php-reverse-shell). Get the file and edit the LHOST and LPORT and put in your own.

Next, start a netcat listener to your chosen port. `nc -nvlp <LPORT>`

#### 2. Travel to the "Templates" section

![Picture to illustrate the point](https://i.imgur.com/8lAjdUX.png)

#### 3. Select the Beez3 Template

![Picture to illustrate the point](https://i.imgur.com/v7UuDWp.png)


#### 4. Edit the index.php file with your php reverse shell code and preview the template to start the reverse shell 

![Steps to get the rev shell](https://i.imgur.com/eKwsouA.png)

Once you click on `Template Preview`, you should get a connection on your netcat listener. As we can see, we are now connected as user `Apache`.

![We are in!](https://i.imgur.com/dyjvavK.png)

**We are in!**


## **Step 4 : POST-EXPLOITATION**

> In order to gain root access, we first take a look at credentials stored inside our website's configuration file. Now logged in as a normal user, we privesc to root by exploiting a sudo permission.

### *Upgrade our shell to a tty shell*

In order to avoid annoyance at the later stages, I always upgrade our basic shell to a tty shell. This can be done using various [NetSec one-liners](https://netsec.ws/?p=337). I went for the python one and it worked fine. The only element I changed was the `/bin/sh` to a `/bin/bash` in order to get a bash shell.

My command goes like this: `python -c 'import tty; pty.spawn("/bin/bash")'`

![Illustrating the previous point](https://i.imgur.com/A9nTCFY.png)

### *Finding users on the machine*

Next, I enumerated through the users on the machine in order to see if jonah existed. We do find his user account (`jjameson`), but his password doesn't work.


### *Step 4.2*

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.

#### Substep 4.2.1

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.


## **SUMMARY**

> Summary of the box with main take aways

# **That's it for now. Thank you for reading this write-up!**

### **Vonshad**
