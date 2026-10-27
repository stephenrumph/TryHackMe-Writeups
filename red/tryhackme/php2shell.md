---
icon: user-hoodie
---

# PHP2Shell

**Date:** 2026-10-25 **Profile:** skrumf **Platform:** TryHackMe **Room URL:** [Link to Room](https://tryhackme.com/room/cvephp2shell) **Difficulty:** Easy **Machine:** Ubuntu **Tags:** php2shell, CVE-2026-109618, CVE-2026-109619, THM

***

## Framework - MITRE ATT\&CK

* ✅ Initial Access The entry point is an HTTP POST request.
* ✅ Execution A dangling memory pointer gets overwritten to run arbitrary system commands.
* ✅ Persistence Dropping a web shell or backdoor to maintain access after the initial exploit connection is closed.
* ✅ Privilege Escalation Existing server vulnerabilities can be exploited to escalate from www-data to root.
* ✅ Actions on Objectives The root shell is the objective.

***

## Initial Access (TA0001)

An attacker can use the `PHP_SESSION_UPLOAD_PROGRESS` parameter to send an HTTP POST and manipulate the server's internal memory management before a single line of code has even begun to run.

## Defense Evasion (TA0005)

The PHP2Shell chain can bypass Address Space Layout Randomization (ASLR), which is there to protect the memory locations of a running OS. The exploit deliberately triggers Out-of-Memory (OOM) errors during the upload progress check. This leaks memory pointers in the response, which lets the attacker map out the memory layout and potentially carry out the follow-up exploit. Specifically, the attacker abuses the `Set-Cookie: PHPSESSID` header in the HTTP response.

## Execution (TA0002)

Once the memory is mapped, the attacker can send another request that overwrites the dangling memory pointers and hijacks the PHP execution flow. As a result, the attacker can run commands as `www-data`, which gives them the potential to run a reverse shell.

## Persistence & Privilege Escalation (TA0003 & TA0004)

PHP2Shell is really just the launchpad. Once an attacker is inside, you can bet your bottom dollar they'll do whatever it takes to root that machine. Patching everything behind the PHP vulnerability is almost as important as dealing with the PHP vulnerability itself, since other weaknesses might be sitting on the server.

## Target & Vulnerabilities

* Target: =< PHP 8.4.26
* CVE-2026-109618 (Memory Disclosure)
* CVE-2026-109619 (Remote Code Execution)

## Lessons Learned

While disabling the feature by setting `session.upload_progress.enabled` to `0` and restarting all workers can disrupt the public exploit chain, it is only a temporary band-aid. This workaround blocks the specific Remote Code Execution path (CVE-2026-109619), but it does not fix the underlying memory disclosure flaw (CVE-2026-109618). An attacker could still find alternative ways to trigger the OOM bailout and leak server memory. Ultimately, the only true defense is to fully patch the PHP engine to a secure version.
