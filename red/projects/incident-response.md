---
icon: shield-halved
---

# Incident Response, End to End

**Date:** 2025-05-23\
**Profile:** skrumf\
**Tags:** Blue Team, Incident Response, TryHackMe

***

This was the big project for my Cert IV, an eight week assignment where I had to plan, implement and review an incident response capability for a fictional company using the TryHackMe incident response and red team paths. The [planning write-up](ir-planning.md) covers setting the project up and writing the plan. This post is the other half, which is how you actually detect and respond to the attacker once they are inside.

## Building the team first

Before any tooling, the project had me design the Incident Response Team itself, and I think doing that first is important. An IR team is more than just a group of analysts. There is a team lead who makes the decisions and delegates duties, a communication lead who manages messaging to the media, government and customers, a lead investigator who chases down attack vectors, the analysts who monitor and triage events as they come into the SOC, and legal counsel to advise on what needs to be disclosed and to who.

Around those people are the things that make them effective. This includes the incident response plan itself, the tools and technologies like a SIEM, IDS and IPS, regular training such as tabletop exercises and purple team engagements, and communication channels that are set up before an incident rather than during one. When all of this is in place the team is able to detect and respond to threats efficiently and with confidence, and knowing there is a plan in case of an emergency is a big advantage against any adversary.

I also had to outline the wider set of roles you see in a real incident, including the SOC analysts and SOC lead, the forensic and malware analysts, threat hunters, first responders, security engineers, the information security officer who acts as the conduit between the SOC and the incident response team, and the incident and crisis managers who take over as things escalate up to executive level.

## The incident response lifecycle

The plan follows four phases and the order they happen in matters:

1. **Preparation** - creating readiness in the environment so that calm heads prevail if an attack happens.
2. **Detection and analysis** - finding out what happened and how it occurred.
3. **Containment, eradication and recovery** - done in that order, so that you understand what you are removing before you remove it.
4. **Post-incident activity** - the full review of everything that was done, which feeds back into preparation.

Something I had not thought much about before was the Business Continuity Plan that sits above all of this. A BCP is only invoked when an incident meets a high threshold, after containment measures have taken effect, and only with the authorisation of senior business leaders, because it supersedes standard operating procedures. It is also measured in hard numbers like the recovery point objective, which is the threshold of data loss, the recovery time objective for recovering hardware, the maximum tolerable downtime, and the mean time to repair. Seeing recovery written down as metrics like this changed the way I think about getting a business back to normal.

## First responder and documentation

The key tasks of a first responder come down to three things. You preserve the evidence, which is treated as volatile and handled in a way that keeps it legally permissible. You alert the relevant stakeholders, usually through a call tree or playbook. And you isolate the incident to reduce the attack surface, whether that is through network, physical or virtual isolation.

Documentation runs through all of it. Once a BCP has been invoked and you are bypassing standard business procedures, it is important to trace the decisions you make throughout the whole process. Good documentation maintains accountability, enables an accurate handover, supports the post-incident review, fulfils compliance and improves coordination. It sounds obvious, but it is the kind of thing that is easy to skip when you are in the middle of an incident.

## Red, blue and purple

A big part of the project was evaluating the different team types, and getting the differences clear in my head was genuinely useful:

* **Red team** - the offensive team trying to exploit the weaknesses of the network, simulating an attack as if they were unknown hackers.
* **Blue team** - the defenders, responsible for keeping attackers out and remediating the holes they find.
* **Purple team** - the conduit between red and blue, there to help both teams communicate their findings.
* **White team** - they create the rules of engagement and act as the referees, and they also provide the environment the tests are performed in.

The rules of engagement are just as important as the techniques. The red team must stay within the agreed scope, cannot destroy or steal data, has to keep detailed logs, and will stop if the white team tells them to. The blue team gets no prior warning, logs all of their activity and detections, and follows the incident response plan. The purple team stays neutral and the white team can stop and start the activity if needed.

## Going purple

The part I enjoyed the most was the purple team room, where you run an attack and then go and find it in the logs. Using the Atomic Red Team framework I would invoke a technique and then move over to the defender's side to hunt for the evidence in the Windows Event Logs and Sysmon.

One example that stuck with me was after a command modified the hosts file. Using Event Viewer I searched Sysmon for Event ID 11 at the time the command was run, and I could see that the `C:\Windows\System32\drivers\etc\hosts` file had been modified. Seeing an attacker action and the log it generates side by side is the fastest way I have found to understand why a particular detection rule exists.

## Lessons learned from the Swiftspend incident

Part of the review walked through a simulated breach, and the lessons learned phase of any incident response is something that should not be overlooked. The weaknesses were all preventable:

* **Identification and scoping** - endpoint protection for key stakeholders, specifically the head of IT, was outdated, and a lack of SPF, DMARC and DKIM records meant the attacker could spoof email addresses, which is how the phishing attack succeeded.
* **Containment and threat intel** - malicious IPs from well known attackers could have been blocked with up to date rules in the IDS/IPS, and insufficient MFA meant the attacker could log straight into the compromised accounts with just an email and password.
* **Eradication and recovery** - the overuse of the same password across multiple accounts is what let the attacker move laterally through the systems.

None of these needed a particularly clever attacker. They needed someone to identify the weakness first, which is exactly what the lessons learned phase is there to do.

## What I took away

The biggest thing I took from this project was learning to work from both sides at once. You plan the defence, but you only really understand it once you have attacked it and watched what it looks like in the logs. It emphasised how important it is that even if you are a defender, at times you will need to think like an attacker to understand what is happening within a networked environment. That is the mindset I am carrying into everything I do next.
