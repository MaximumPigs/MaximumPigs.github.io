---
layout: post
title: Project - Detection Lab - Part Two
categories: ["Projects"]
tags: ["Detection Lab", "Terraform", "AWS", "Ansible", "GitHub Actions", "AWS"]
author:
  - MaximumPigs
---

What would Brian Boitano do?

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

This caused me an unfathomable amount of pain. Likely after a raft of complains from people turning off Windows Defender, only to get popped later - and then  blaming Microsoft, Microsoft have introduced some updates which make it impossible to disable Windows Defender programatically. If you run a powershell command to disable Defender, it will show as succeeded - but Defender will still be enabled. The only way to do it is to navigate through the GUI of each machine and disable it manually. This is a good thing, it keeps people from shooting themselves in the foot and disabling their Endpoint Detection and Response (EDR) capability for the entire environment, so I really shouldn't complain ... but I'm going to.  

This is an issue for me, because I'm TRYING to build a vulnerable machine but Microsoft (rightly so) are making this difficult for me. How is this a problem using AWS? Well, all the exsting AWS Machine Images (AMIs) are up to date with this new feature, obviously. Everything works, all the way up to disabling Windows Defender, and then the whole process breaks down.

Luckily, I've found a solution - I simply need to build my own AMI using an old version of Windows.

No! Says Microsoft, who apply in-place security updates to all of their ISO's. Even if you download a version of Windows which pre-dates this functionality, it is already applied to the ISO retrospectively.

So I ask myself, [what would Brian Boitano do?](https://www.youtube.com/watch?v=sNJmfuEWR8w) _(language warning)_ in this situation?

He'd make a plan, and he'd follow through, that's what Brian Boitano'd do.

Also, he'd probably try and use the Windows Virtual Machines that CLong uploaded to the Vagrant cloud, as they still seem to work when deploying the Detection Lab locally.

## Sysprep and VMWare OVTool

I learned quite a lot about Sysprep at this stage, and I won't bore you with all the details but essentially I got it all working by doing this:

- Provision the Detection Lab locally using Vagrant to download all of the Virtual Machines.
- Using the VMWare OVTool, convert each of the Virtual Machines to the Open Virtualization Format (OVF), which is compatible with AWS.
- Upload each of the new OVF Files AWS S3.
- Import each of the OVF files as a new Image in AWS.
- Create an EC2 instance from each of the new Images.
- Launch each EC2 instance and run Sysprep to generalise the image.
- Create a new AMI from each of the sysprepped EC2 instances.
- Delete everything from AWS except the completed AMIs (because $$)

The main reason this needed to be done in as many steps is the fact that I didn't want a separate AMI for each Windows Server, the whole point of this project was to provision the environment from base images - if I had an image file for each host then I may as well just provision it manually and take a snapshot.

In order to use the same Windows Server images for two hosts I had to strip it of it's GUID as you can't join two machines with a matching GUID to the same domain. Sysprep will revert the system to a near out-of-the-box state, and it will receive a new GUID on first boot.

## What's Next?

Well, it works. Unfortunately I've had to use my own AMIs, which I am reluctant to share publically because I feel that would make me liable in some way were bad things to happen.

I'll leave the code in a public repository though, feel free to take a leaf out of my book and make it your own when you feel like bashing your head against a brick wall for a few days! I learned a lot, and I hope you do too.

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
    - [X] Windows Domain Controller
    - [X] Windows Event Forwarder
    - [X] Windows 10 Host
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
