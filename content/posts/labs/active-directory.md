---
title: "Intro to Active Directory"
date: 2026-11-21T10:18:53-06:00
draft: false
toc: false
images:
tags:
  - HackTheBox
  - ActiveDirectory
  - DomainController
---

Straight to the point lets say a business has 200 computers with more than 500 different users located across four different offices. Would you be able to manage each computer as itself, manually configuring policies for each user across the network? The answer is no obviously.

The solution to that is we can use a Windows domain, which simply put is a group of users and computers under the administration of a business. The whole purpose of a domain is to centralize the administration of a Windows computer network in a single repository which I'm sure it clicked it's called Active Directory (AD). The server that runs Active Directory services is known as a Domain Controller (DC)

![](https://www.quest.com/images/patterns/text-image/adlearnpage-01.jpg)

> These topics are extensive but I'm overall cutting it short to provide a general idea, if you would like to find more information I suggest [this video](https://www.youtube.com/watch?v=T55AcTV_m7E). Side note always use more than one source of information just in case I make mistakes... cause I am human duh.

## Security Groups

Groups can have both users and machines as members to define users access rights to files or other resources defining them to entire groups instead of single users. It allows better manageability since you can add users to groups and automatically inherit all group's privileges.

- Domain Admins: Users of this group have administrative privileges over the entire domain. They can administer any computer on the domain, including the DC's

- Server Operators: Users in this group can administer Domain Controllers. They cannot change any administrative group memberships.

- Backup Operators: Users in this group can backup and restore files even if they normally wouldn't have access (backup/restore privileges). They are used to perform backups of data on computers.

- Account Operators: Users in this group can create or modify other accounts in the domain. They cannot mess with high privilege groups like Domain Admins, Enterprise Admins, Schema Admins, Administrators, Account Operators, or Server Operators.

- Domain Users: Includes all existing user accounts in the domain.

- Domain Computers: Includes all existing computers in the domain.

- Domain Controllers: Includes all existing DCs on the domain.

## Default Objects

Inside Active Directory you will probably notice default containers. These containers are created by Windows automatically and contain the following:

- Builtin: Contains default built-in groups for the domain (Administrators, Users, Guests, etc.).

- Computers: Any machine joining the network will be put here by default. You can move them if needed.

- Domain Controllers: Default [OU](https://www.manageengine.com/products/active-directory-audit/kb/what-is/ou-in-active-directory.html) that contains the DCs in your network.

- Users: Default users and groups that apply to domain-wide context.

- Managed Service Accounts: Holds Accounts used by services in your Windows domain.

![](https://ad4noobs.justin-p.me/how_to_use_active_directory/active_directory_users_and_computers/default_objects/overview.png)
> Quick note: Builtin, Computers, and Users are **containers**, Domain Controllers is an actual **OU**. Organizational Units (OUs) are container objects in Active Directory that allows you to organize and manage your network resources (and you can link GPOs to OUs). I should've explained it before I used the term above but well..... now you know... 
>
> For more information please visit [here](https://www.manageengine.com/products/active-directory-audit/kb/what-is/ou-in-active-directory.html) or [here](https://ad4noobs.justin-p.me/how_to_use_active_directory/active_directory_users_and_computers/default_objects/)
## Delegation

One main thing you can do with [AD](https://www.quest.com/solutions/active-directory/what-is-active-directory.aspx) is to give specific users some control over some OUs. Which is known as *delegation* and allows you to grant users specific privileges to perform advanced tasks on OUs without needing a Domain Administrator to step in.

For example help desk has permission to change password and other resources without giving them full permission, how do you do it? well you might've guessed it, delegation control. You would add a user for example user named `Jerry` we can simply add specific common or custom tasks rules it can access. 

> For a full step by step or more information (I've been summarizing for the sake of this blog) you can check [here](https://www.manageengine.com/products/ad-manager/active-directory-delegation.html)

![](https://www.manageengine.com/products/ad-manager/images/delegation-of-control-wizard.webp)
