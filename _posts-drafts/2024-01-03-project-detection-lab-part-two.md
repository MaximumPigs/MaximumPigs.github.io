---
layout: post
title: Project - Detection Lab - Part Two
categories: ["Projects"]
tags: ["Detection Lab", "Terraform", "AWS", "Ansible", "GitHub Actions", "AWS"]
author:
  - MaximumPigs
---

Headline

![Image Title](/assets/images/procrastination_vs_hyperfocus.webp "ChatGPT prompt")  
*Image caption*  
<br />

So here's a lesson in relativity. If several weeks of real time passes between posts, but nobody will read them for months ... 

I've made a whole heap of progress since my last write-up, so much that I'm really regretting not breaking it down into smaller chunks between posts. I ran into a few problems, and made a few changes but I think it's working ... well, it's at least running and appears to be functional.

The great thing about Ansible is that it doesn't care what hypervisor I'm using, so I've been able to work on the playbooks at home on AWS, and at work on VCD. Ironing out a bug in one, also fixes the bug in the other.

The biggest hurdle I've run into so far is getting the base OS images to work, for several reasons. It's been a huge lesson for me in sysprep, packer, Windows evaluation images and perserverence. An absolutely worthwhile experience, but frustrating nonetheless.

In short, it appears that the Windows evaluation images (software-download.microsoft.com) are patched in place. They have all updates and security patches applied to them at the time of download, regardless of how long ago the image was released.

This doesn't sound like an issue, and it wouldn't be in most situations but in most situations people are probably not trying to build an intentionally vulnerable environment. A little while ago a security patch was released which removed the ability to disable Windows Defender Real-Time Protection in any way other than interactively. The only way to turn it off now is to boot the machine, open the settings and click the slider to disabled it. There are many traditional ways to Disable this feature through Powershell, or by modifying a registry entry ... and those methods will appear to still work, they don't throw any error messages or deny access, however this is nothing but a facade. Windows Defender Real-Time Protection will still be enabled.

The easy solution to this is just to fire up the machine, turn off Real-Time protection and then provision the machine with Ansible. I was hoping to have a fully automated provision process though, so while this would work it's not ideal.

## Windows Defender and why I can't provision from an existing AWS AMI.

## Sysprep

## Vagrant Folder

## WinRM ports

## Network ports for Zeek / Suricata

## Outcome

## What's Next? Make faster happen.

-----------------------

- [X] Write a post about starting a project
- [X] Don't procrastinate
- [X] Create a fork of Clong's Detection Lab. 
- [X] GitHub Actions
  - [X] Create some actions that will provision to cloud providers using a GitHub Runner
  - [X] Modify the AWS Terraform code to accept GitHub.Secrets
  - [X] Review AWS Terraform code for any references to things stored locally - Modify for GitHub Actions
  - [X] Test it
- [ ] AWS - Provision
  - [X] Create a new folder for this, as I want to preserve the user built AMI, Terraform provisioner deployment method
  - [ ] Find some basic OS AMIs that I can use  
      *I've decided to run more recent OS versions to future proof the build.*
      - [ ] Windows 10
      - [ ] Windows Server 2022
      - [X] Ubuntu-22.04
  - [ ] Modify Terraform code to include a provisioner  
    - [X] Linux Logger
    - [ ] Windows Domain Controller
    - [ ] Windows Event Forwarder
    - [ ] Windows 10 Host
  - [X] Decide whether to use scripts from Vagrant or Ansible playbooks from the Azure method.  
    *I've decided on Ansible because Clong already has great playbooks used in his ESXi build.*
  - [X] Write in any modifications to match the new provider
    - [X] Linux Logger
    - [X] Windows Domain Controller
    - [X] Windows Event Forwarder
    - [X] Windows 10 Host
  - [X] Test it
- [X] VCD
  - [X] Plan out the build, identify what resources are required and the VCD equivalent
  - [X] Write Terraform Code
  - [X] Provision using Ansible
  - [X] Test it
- [ ] Re-useable workflow
  - [ ] Modify my actions to make them reuseable
  - [ ] Write a template people can use
  - [ ] Test it
- [ ] Clean it all up and document
  - [ ] Refactor some code, and put global resources in a common folder (ie. ansible)
  - [ ] Document how to launch in own repo
  - [ ] Document how to call using github action
  - [ ] Document building own AMIs - Can I share these somewhere instead?

---

### Links and References

1. **AWS - (Amazon Web Services)**: [Amazon Web Services Official Site](https://aws.amazon.com/)

---

[1]: https://github.com/MaximumPigs/DetectionLab/tree/v0.1.0 "MaxiumumPigs DetectionLab Version 0.1.0"
