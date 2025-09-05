# TryHackMe CTF Challenge: Biling


## Skill used:
- OSINT
- Nmap

## Prompt:
Gain a shell, find the way and escalate your privileges!
Note: Bruteforcing is out of scope for this room.

- Need to find the flag inside `user.txt`
- Need to find the flag inside `root.txt`

## Recon
IP Address given: 10.201.74.150

Entering the IP address on our browser reveals a login page for banking website. The organization name is MagnusBiling, we'll keep this in mind as there might be some info on the web about them. Since password bruteforcing is out of the question, we won't pay much attention to it for now

I did some searching and it seems that MagnusBiling is an actual legitimate business lol, and they are open source. 

[insert recon_01.png here]

### Nmap
Using Nmap to scan the ports of the server, looking for any open ports that we can exploit. I would have used rustscan but I'm doing this CTF on VM, on a MacOS and so the Kali Linux infrastructure is sightly modified which makes me unable to install `.deb` file onto my machine :/ 

`nmap 10.201.74.150 -sS -sV -p- -T5`
- -sS: TCP SYN scan, much more faster than TCP connect scan
- -p-: Scan all ports
- sV: Gather info what services are listening on the open ports
- -T5: Timing template, increasing num of threads mean faster scanning time (not recommend for stealth recon)

[insert recon_02.png here]

#### Output:
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
80/tcp   open  http     Apache httpd 2.4.62 ((Debian))
3306/tcp open  mysql    MariaDB (unauthorized)
5038/tcp open  asterisk Asterisk Call Manager 2.10.6
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

4 ports are available, 2 ports that caught my attention is the mysql(3306) (which is currently unathorized) and the asterisk port(5038). 

### Gobuster 

I decide to do some more snooping around for any other files with Gobuster, ran this command:
`gobuster dir -u "http://10.201.74.150/mbilling" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.js,.html`

- -u: specify host to be enumerated 
- -w: path to wordlist 
- -x: extensions to be filtered and look for 

[insert recon_03.png]

We found some JUICYY stuff from our enumeration attempt. There seems to be an `archive` page containing folder storing assets, which open for opportunities to carry out reverse shell attack 

Looking into one of the dir `lib`, we found a bunch of other folders, and just a quick search to to see if there are any vulnerabilities related to those services, we found a website that MagnusBiling ver 6.x and 7.x are vulnerable to a Remote Command Execution (RCE) attack within the `lib/icepay/icepay.php` via using Metasploit

P.S: I didn't scanned it well enough, but there was a README.md file, revealing the web application is in version 7.x.x

[insert recon_04.png]

## Exploitation

### Metasploit 
We use the exploit available on Metasploit `msfconsole> use exploit/linux/http/magnusbilling`, all we need to do insert our IP address and the targeted machine IP address. 

[insert exploit_01.png]

Nice it seems we got access to the server, and by snooping around the dir and files, we found our `user.txt` within `home/magnus`. 

[insert exploit_02.png]

I created a shell to access the server via my linux terminal, and see if there's any ways to escalate our privileges. If we run `sudo -l' which list all of the commands you're allowed to run with sudo, which return an interesting result.

[insert exploit_03.png]


### Privilege Escalation 

Fail2Ban is a service that we can run apparently with sudo, and after some research [link:https://juggernaut-sec.com/fail2ban-lpe/] on ways I can escalate my privileges to root.

Gonna briefly explain this since I personally find it useful for writeups to explain their stuffs and findings rather than just blatantly pasting commands :P 

- Fail2Ban is a security program to prevent brute force attack (prob that's why they told us not to brute force)
- Scans log files and ban IP from too many failed login attempts 
- By default, the ban last for 10 mins when 5 auth failures has been detected within 10 mins 

Key config files:
- fail2ban.conf: Used to configure operational settings, how will daemon(background process that runs continuously) logs info, the socket, and pid file that will be use - but not very useful to attackers
- jail.conf - Main config file
- jail.local - An exntesion of `jail.conf` and used to dnable jails 
- iptables-multiport.conf: Action file that set up firewall and allow edits for banning malicious hosts & adding/removing hosts 
- iptables.conf: the 'new' multiport action file used in Fail2Ban ver >= 1.0.1

2 important binaries: `fail2ban-server` & `fail2ban-client` which start/stop Fail2Ban, view banned IPs, unban IPs, view configured jails..etc 

#### Upgrading Our shell
- Need to upgrade our dumb shell to full TTY(teletypewriter) - provide us with an interactive & full functional command-line interface

```
python3 -c 'import pty;pty.spawn("/bin/bash");'
CTRL + Z         #backgrounds netcat session
stty raw -echo
fg               #brings netcat session back to the foreground
export TERM=xterm
```

#### Hunting for Users in the fail2ban group
We run 





## Final Words 
FUCK VM, more time was spent troubleshoot the VM than the actual CTF I felt like xD 


