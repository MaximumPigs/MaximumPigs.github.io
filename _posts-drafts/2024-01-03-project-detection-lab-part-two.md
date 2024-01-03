---
layout: post
title: Project - Detection Lab - Part Two
categories: ["Projects"]
tags: ["Detection Lab", "Terraform", "AWS", "Ansible", "GitHub Actions"]
author:
  - MaximumPigs
---

So I made a commitment to a project, but we all know that planning is the first step towards failure.  

![Procrastination vs Hyperfocus](/assets/images/procrastination_vs_hyperfocus.webp "Make me a picture that shows a struggle between procrastination and hyper focus. Do not include any words on the picture. Make sure it will look good against a black background and make it in a 3:1 aspect ratio")  
*Kind of looks like my bloodshot eyes after staring at a screen all day. Also, ChatGPT isn't very good at image sizes.*  
<br />

Spoiler: I spent an unhealthy amount of time on this when I really should have been enjoying a few days off work.  
What did I learn about the importance of taking time to relax? ... Absolutely nothing, I'll do it again, just watch me.  

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
  - [ ] Write in any modifications to match the new provider
    - [X] Linux Logger
    - [ ] Windows Domain Controller
    - [ ] Windows Event Forwarder
    - [ ] Windows 10 Host
  - [ ] Test it
- [ ] VCD
  - [X] Plan out the build, identify what resources are required and the VCD equivalent
  - [X] Write Terraform Code
  - [ ] Provision using Ansible
  - [ ] Test it

---

### Links and References

1. **AWS - (Amazon Web Services)**: [Amazon Web Services Official Site](https://aws.amazon.com/)

---

[1]: https://github.com/MaximumPigs/DetectionLab/tree/v0.1.0 "MaxiumumPigs DetectionLab Version 0.1.0"
