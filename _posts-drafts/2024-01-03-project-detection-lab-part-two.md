---
layout: post
title: Project - Detection Lab - Part Two
categories: ["Projects"]
tags: ["Detection Lab", "Terraform", "AWS", "Ansible", "GitHub Actions"]
author:
  - MaximumPigs
---

Headline

![Image Title](/assets/images/procrastination_vs_hyperfocus.webp "ChatGPT prompt")  
*Image caption*  
<br />

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
