## Introduction
In hack the box, root-me and similar war-game like challenges that are security related, we often follow an identical pattern when trying to get a certain flag from the so called "boxes".
In this article, I will try to outline this pattern in what I would like to call a ctf mold, or a typical ctf walkthrough.
In this article, we consider that we are connected to the vpn that allows us to interact with the victim machine. This victim machine will have the following ip address : 10.13.37.10

Please double check that you are connected to the vpn before starting.

## Discovery
The starting point is almost always the same : Enumerate to discover the open ports that may help us get an initial foothold in the machine.
To do so, we run nmap as follows:
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
The first step is to access the following url: http://10.13.37.10:80
If it resolves into a domain name, say for example: `http://sowon.mold`, it will have to be added to our /etc/hosts file.
To do so, we execute the following:
```bash
$ sudo echo '10.13.37.10 sowon.mold' >> /etc/hosts
```
Once in the webpage, check all pages and look for webpages that are of interest: a login page, a file submission page, a page that displays user-input ... Or CMS versions that are used in the webpage (joomla, ...)

### Login Page
If the website is based on a known ......? page ..??, google for possible default credentials and try them.
: Examples of default credentials are: admin/admin, user/password etc...

### File submission page

### Known CMS or ...?
The first thing to do is to google for known vuulnerabilities and exploits. To do that we can google "name_of_CMS_version CVE exploit".

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
$ gobuster todo!
```
We can also check for virtual hosts, which are directories and files that are hosted in virtual environments and have a different DNS subdomain:
```bash
$ gobuster vhost -u sowon.mold -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

## DNS
The first step is to execute a dig command to .....

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
For this example, the Authority section is interesting since it shows the www.sowon.mold address which is another webpage that may be of interest to us.

```bash
$ dig axfr sowon.mold @10.13.37.10

; <<>> DiG 9.19.17-1-Debian <<>> axfr sowon.mold @10.13.37.10
;; global options: +cmd
sowon.mold.       604800  IN      SOA     www.sowon.mold. sowon.mold. 2 604800 86400 2419200 604800
sowon.mold.       604800  IN      NS      www.sowon.mold.
sowon.mold.       604800  IN      A       10.13.37.10
www.sowon.mold.   604800  IN      A       10.13.37.10
sowon.mold.       604800  IN      SOA     www.sowon.mold. sowon.mold. 2 604800 86400 2419200 604800
;; Query time: 155 msec
;; SERVER: 10.13.37.10##53(10.13.37.10) (TCP)
;; WHEN: Fri Jan 05 09:21:27 EST 2024
;; XFR size: 5 records (messages 1, bytes 167)
```

TODO : DNS examples from other walkthroughs are probably better

## Databases : SQL, KeePass, 
