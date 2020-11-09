# **Daily Bugle**

![Box banner](https://i.imgur.com/JXkKIrJ.png)

*Plateform* - [**TryHackMe**](https://tryhackme.com/room/dailybugle)

*OS* - **Linux**

*Rating* - **Hard**

*When all you want is pictures of Spider Man, but they only bring you this box.*


## **Step 1 : RECONNAISSANCE**

### *Finding a login form and a potential username*

![Picture of the index.php](https://i.imgur.com/CUeCFxM.png)

Already, this box seems to be giving us great information. While we took notes of these elements, they won't be of critical importance later on, so I will not be covering them extensively.


## **Step 2 : SCANNING AND ENUMERATION**

During this step, we find that the machine is running a webserver on which a CMS (Joomla! version 3.7.0) is running. We also find the /administrator dashboard (can't log in yet) on the webserver.

### *Nmap Output*

I ran the following command: `nmap -A -T4 -p- -oN <output_name> -v <Target_IP>`. This is a classic full-range scan.

![Picture of the Nmap output](https://i.imgur.com/LKZYEQg.png)

### *PORT 22 - SSH*

While it is important to take note that the SSH service is running on the target machine, it usually isn't a way in on its own.

### *80 - HTTP*

This is the webserver we found during *Step 1 - Reconnaissance*. The Nmap results indicated to us it is running the **Joomla!** CMS and has disallowed entries in the **robots.txt** file.

#### A look at the website

The elements that can be found "as is" on the websites are the following: a potential username (**Super User**) and a login form. Attempting to use SQL injections won't be of any help and we can't seem to be able bypass the login form.

#### Robots.txt content

This robots.txt file validates that the website is running the Joomla! CMS and gives us the location of a few interesting folders, especially the **/administrator** one.

![Picture of the robots.txt file](https://i.imgur.com/fsKHuQF.png)

#### Visiting the /administrator folder

The /administrator folder prompts us to log in. SQL injections won't work here either. Even though we can't exploit it right now, this administrator panel will be our way in during *Step 3 - Exploitation*.

![Picture of the /administrator folder](https://i.imgur.com/n7X9fgt.png)

#### Running dirbuster and finding a Joomla! Version

After exploring the obvious options, it is time to bust some directories.

### *3306 - MySQL*

We learn MySQL is running on the target machine.

## **Step 3 : EXPLOITATION**

> Rough explanation of what we will do. Do make sure to post a screenshot for every ### step here.

### *Step 3.1*

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.

### *Step 3.2*

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.

#### Substep 3.2.1

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.


## **Step 4 : POST-EXPLOITATION**

> Rough explanation of what we will do. Do make sure to post a screenshot for every ### step here.

### *Step 4.1*

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.

### *Step 4.2*

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.

#### Substep 4.2.1

Lorem ipsum dolor sit amet, ex soleat dicunt consectetuer quo, qui id unum menandri, nec in minim laoreet indoctum. Mea insolens recusabo an, quo eu erant definitiones. Has nullam putant phaedrum ei. Qui ei pertinacia consequuntur reprehendunt. Ad vix homero placerat inciderint, ad quod justo luptatum sit.


## **SUMMARY**

> Summary of the box with main take aways

# **That's it for now. Thank you for reading this write-up!**

### **Vonshad**
