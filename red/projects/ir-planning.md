---
icon: clipboard-list
---

# Planning an Incident Response Project

**Date:** 2025-06-12\
**Profile:** skrumf\
**Tags:** Blue Team, Incident Response, Project Management

***

This was the first half of my Cert IV incident response project, and where my [implementation write-up](incident-response.md) covers actually running the exercise, this one is about everything that has to happen before anyone touches a keyboard. I was the project lead, so my job was to set the project up, build the team, evaluate the plan that already existed, and write a new one. It turned out to be a good lesson in how much of security work is planning and communication rather than tooling.

## Getting the project off the ground

The first thing I had to do was pick a tool to actually manage and track the project. I looked at three options and compared them before deciding.

* **ClickUp** - a lot of features in the free version, different views to manage tasks, and it integrates well with other apps. The downside is a bit of a learning curve and a layout that can feel overwhelming at first.
* **Trello** - easy to use and great for small tasks with its Kanban boards, but not really suited to complex or solo projects.
* **Asana** - good for complex projects with long term automation, but the useful features sit behind a subscription and it is not great for a solo user.

I went with ClickUp, mostly because the free version did everything I needed and I could manage tasks, build a Gantt chart and keep my status reports all in one place.

## Building the team

With the tooling sorted, the next step was establishing the project team and assigning roles. The core roles I defined were the project sponsor/manager who oversees the project and aligns it with business goals, the incident response lead who makes the operational calls during an incident, the technical lead who oversees containment, eradication and recovery, and a documentation officer to keep records of everything that happens.

On top of that I had to outline the red, blue and purple teams, plus a couple of others that come into play, like a forensic team for post-exploit investigation and a legal and compliance team to keep the handling of data within the law throughout the engagement. I also looked at the quality assurance lead, who reviews internal procedures and makes sure documentation is actually followed, and the communications lead, who controls the flow of messages to stakeholders during and after an incident.

The part I found most useful was deciding how the team's performance would actually be measured. I settled on three metrics:

* **Mean Time to Detect (MTTD)** - how long it takes to detect an incident from the moment it occurs.
* **Mean Time to Respond (MTTR)** - how long it takes to respond once the incident has been detected.
* **Post-Incident Review Completion Rate** - a measure of how thoroughly the lessons learned are being captured after each incident.

Having numbers to point at changes the conversation from "I think we did ok" to something you can actually track and improve over time.

## Evaluating the existing plan against NIST

Before writing a new incident response plan, I had to evaluate the one that already existed against the NIST SP 800-61 Computer Security Incident Handling Guide, looking at both the plan elements and the procedure elements.

The procedure side was in reasonable shape. The plan used the PICERL model (Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned), it had a step-by-step triage process for detection and analysis, it treated containment, eradication and recovery as separate stages with clear responsibilities, and it referenced evidence preservation and escalation in line with the framework.

The plan elements were where the gaps showed up. There was no clear mission statement, which matters for giving the response direction. There were no metrics for things like time to detect, respond and recover. There was no roadmap for maturing the capability over time, and the plan did not explain how the incident response team fits in with the rest of the organisation. Those became the basis for my recommendations.

I also had to document the business impact of a few cyber threats, which was a good exercise in thinking past the technical side:

* **Ransomware** - operational disruption from encrypted systems, direct financial loss from downtime, reputational damage if customer data is lost, and compliance penalties for failing to report it.
* **Phishing and credential theft** - unauthorised access to critical systems, data breaches that could end up on the dark web, regulatory fines, and an erosion of trust with staff and partners.
* **Denial of service** - service outages for public banking, lost revenue from downtime, the cost of the response itself, and customers leaving for competitors.

## Writing the new plan

The biggest piece of the project was developing the new IRP for the scenario, which was built around a bank called TheReserve. This meant writing the rules of engagement, then the playbooks for each team.

The **rules of engagement** set out exactly what was in and out of scope. The scope covered the 10.200.223.0/24 range and hosts like corp.thereserve.loc and swift.bank.thereserve.loc, along with exposed mail and application servers, phishing of employee accounts, and testing the SWIFT payment mechanism. The out-of-scope list was just as important, ruling out the e-Citizen SSH portal, anything outside the subnet, external OSINT, touching the mail server configuration, and any denial of service activity. I also set out the operational rules, like the red team signing up through the e-citizen portal and validating every compromise with a UUID file in a set location, and logging all actions for later analysis.

The **red team playbook** laid out the range of attacks: credential harvesting through OSINT, brute force and phishing, privilege escalation through exposed credentials and misconfigured services, a reverse shell through a misconfigured VPN server, and network scanning across all hosts. A few specific methods came out of this, like a spoofed login page built to harvest credentials, a writable Windows service running a batch file that could be abused to create an administrator account, and captured MySQL credentials that led to a `.ovpn` file and RDP access. The tools were the usual suspects for that kind of work - Kali, Burp Suite, netcat, Hydra, WinPEAS and Mimikatz.

The **blue team playbook** defined the defensive side: firewalls and IDS/IPS with Cisco Umbrella and Suricata, endpoint protection with Palo Alto Cortex and Windows Defender, and network monitoring with Wireshark feeding into Splunk for analysis. It also set out how incidents would be categorised, how alerts would be routed to the right stakeholders depending on severity, and how logs and memory dumps would be collected before any decision to shut a system down, which has to happen before the attacker realises they have been spotted.

As the project manager I also acted as the observer on the purple team, so I wrote a checklist to evaluate how the red and blue teams performed against detection, response, documentation, communication, evidence collection and the post-incident review.

## What I took away

Going into this I thought incident response was mostly the technical work, the detecting and the responding. What this project showed me is how much of it is planning, measuring and communicating before any of that happens. Writing the rules of engagement and the playbooks forced me to think about the whole engagement end to end, and evaluating the existing plan against NIST gave me a clear sense of what a mature plan actually needs. It set everything up for the implementation side, which is where it all came together.
