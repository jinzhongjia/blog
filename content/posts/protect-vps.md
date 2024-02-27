---
title: Protect your vps!
date: 2023-02-09 16:32:09
tags:
    - vps
    - linux
---

## Preface

Everyone connected to the internet is a target for hackers.

The Most Incredible Cyber Attack Statistics of 2021 Find out the number of cyber attacks happening every day around the world.

- Worldwide, 30,000 websites are hacked every day.

- 64% of companies worldwide have experienced at least one form of cyber attack.

- 20 million records compromised in March 2021.

- In 2020, ransomware cases increased by 150%.

- Email accounts for approximately 94% of all malware.

- Every 39 seconds, a new attack occurs somewhere on the network.

- On average, approximately 24,000 malicious mobile applications are blocked on the Internet every day.

Therefore, network security protection is very important!

## Start protection work

> This article takes Debian as an example, the operation of Ubuntu is similar, and the CentOS system will be different, please search for relevant content by yourself. (It is strongly recommended that you use Ubuntu or Debian system)

### SSH protection

In fact, many vps are regarded as broilers because the ssh login password has been cracked, especially the root user has been cracked by violence and installed a backdoor program, and the whole chick has become a compelling puppet chicken (broiler)

#### Change ssh port

```bash
vim /etc/ssh/sshd_config
```

Then change the `port` field, which is to change the ssh port. After the changes are saved, use `systemctl sshd restart` to restart the ssh service

#### Change ssh password

Very simple, use the `psswd` command to change the password.
Note: _Password is not visible when entered_!

#### Create a new normal user

1. Create a new user

```bash
adduseradmin
```

You can create a new user, and you will be asked to enter the password twice and some information. The information other than the password can be entered all the way to end. If necessary, you can understand the information and set it by yourself.

2. Install sudo

After creating a new ordinary user, we need to use sudo to allow ordinary users to obtain root privileges

```bash
apt install sudo #Debain does not have it by default, Ubuntu itself should have been installed
```

3. Make sudo settings

```bash
visudo
```

Use `ctrl+K O` to save, press Enter, then `ctrl + K Q`, you can save and exit!

Look for the `User Privilege Specification` field, and add a new line:

```bash
admin ALL=(ALL) NOPASSWD: ALL
#admin is the username
```

Setting "NOPASSWD" allows the admin user to use root privileges without entering an additional password. If you are required to enter a password when using root privileges, you can change it to `sadmin ALL=(ALL:ALL) ALL`

4. Disable root login

Forbid root to log in to vps through ssh, which can greatly avoid the situation that vps becomes a bot. At the same time, it can also be used by switching to the root user through the `su` command after logging in through other users ssh

Edit the ssh configuration file:

```bash
vim /etc/ssh/sshd_config
#If you log in as a normal user, you need to use sudo
```

Change the `PermitRootLogin` field to no

Then save and exit, restart the ssh service `sudo systemctl sshd restart`

After that, when using root for ssh login, the vps will directly deny access, and has been asking for a password

### Ufw firewall

#### Install

Install ufw firewall (based on iptable), Debian needs to be installed manually, Ubuntu already comes with it by default

```bash
apt install ufw
```

#### Basic configuration

Set basic defaults

```bash
ufw default deny incoming
ufw default allow outgoing
```

Allow ssh connections

```bash
ufw allow ssh
```

Allow http and https connections

```bash
ufw allow http
ufw allow https
```

#### Start ufw

```bash
ufw enable
```

#### Manage firewall rules

View firewall status

```bash
ufw status
```

```bash
ufw status numbered
```

delete rule

```bash
ufw delete 5
```

add rules

Allow port 81, ufw rules are more complicated

```bash
ufw allow 81
```

Overload configuration

```bash
ufw reload
```

#### Ban ping

Directly edit the rule of ufw, you can also edit the rules of iptable

```bash
vim /etc/ufw/before.rules
```

Find the `# ok icmp codes for INPUT` area, and change `ACCEPT` in `echo-request -j ACCEPT` in the last line to `DROP`

#### No brute force cracking

Use file2ban to ban bruteforcing

Install file2ban

```bash
apt install fail2ban
```

copy configuration file

```bash
cd /etc/fail2ban # Enter the fail2ban directory
cp fail2ban.conf fail2ban.local # backup
```

edit configuration file

```bash
vim fail2ban.conf
```

```bash
[sshd]
enable = true
port = 22 #port
filter=sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = -1
```

service management

```bash
service fail2ban restart #restart
fail2ban-client status #View status
fail2ban-client status sshd #View detailed status of sshd
```

Unban ip

```bash
  fail2ban-client set sshd unbanip 192.0.0.1 #Unban the specified IP
```