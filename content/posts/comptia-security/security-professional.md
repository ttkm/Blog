---
title: "Log #1: Security Professional"
date: 2026-04-23T13:16:22-05:00
draft: false
toc: true
images:
tags:
  - untagged
---

We will learn about the basic iobjectives of cyber security: confidentiality, integrity and availability of your operations.

## Cybersecurity Objectives

It is important to understand that cybersecurity has three complementary objectives shown below:

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781394211418/files/images/c01f001.png)

## Confidentiality

Confidentiality ensures that unauthorized individuals are not able to gain access to sensitive information. 
Professionals develope and implement not limited to

- Firewalls
- Access control lists
- Encryption

Attackers may seek to undermine confidentiality controls to achive one of their goals: the unauthorized disclosure of sensitive information.

## Integrity

Integrity ensures that there are no unauthorized modifications to information or systems, either intentionally or unintentionally. Integrity controls, such as hashing and integrity monitoring solutions, seek to enforce this requirement.

Threats may come from attackers seeking the alteration of information without authorization or nonmalicious sources, such as a power spike causing the corruption of information.

## Availability

Availability ensures that information and systems are ready to implement the needs of legitimate users at the time those users request them.

Availability controls such as:

- Fault tolerance
- Clustering
- Backups

They seek to ensure that users may gain access if needed.

These three goals are often known as the `CIA triad`, when performing their work.

## Nonrepudiation

While not part of the CIA triad, is also an important goal. Nonrepudiation means that someone who performed some action, such as sending a message, cannot later deny having taken that action. Digital signatures are common example of nonrepudiation.

> If you're taking the CompTIA Security+ exam be sure to remember the main components of the CIA triad security model, which is:
>
> - Confidentiality
> - Integrity
> - Availability
>
> Also known that nonrepudiation is the assurance that something cannot be denied by someone.

# Data Breach Risks

Security incidents occur when an organization experiences a breach of the confidentiality, integrity and/or availability of information or information systems.

Incidents occur as result of malicious activity, accidental acctivity such as an employee leaving an unencrypted laptop in public, or a result of natural activity such as an earthquake destroying a datacenter.

## The DAD Triad

Not to get confused with the CIA triad, this model explains the three key threats to cybersecurity efforts.

![](https://learning.oreilly.com/api/v2/epubs/urn:orm:book:9781394211418/files/images/c01f002.png)

This model explains the three key threats to cybersecurity efforts:

- disclosure
- alteration
- denial

## Disclosure

Disclosure is the exposure of sensitive information to unauthorized individuals, otherwise known as data loss. Attackers who gain access to sensitive information and remove it from the organization are said to be performing data exfiltration. This can happpen due by an accident or administration misconfigures access controls.

## Alteration

Alteration is the unauthorized modifications of information and is a violation of the principle of integrity. Attackers may seek to modify records contained in a system for financial gain. Alteration may occur as the result of natural acticity such as a power surge causing a "bit flip" that modifies stored data.

## Denial

Denial is the disruption of an authorized user's legitimate access to information. Denial events violate the principle of availability. This availability loss may be intentional, such as when an attacker launches a distributed denial-of-service (DDoS) attack against a website. It may occur from accidental activity, such as failure of a critical server.

You can apply CIA and DAD for guidance to your organization website:

- Does the website contain sensitive information?
- If an attacker were able to modify information, would it cause unauthorized alteration and cause financial, reputational, or operational damage to the organization?
- Does the website perform mission critical activities that could damage the business if an attacker were able to disrupt the site?

That's just one example of using DAD triad to inform a risk assessment.


