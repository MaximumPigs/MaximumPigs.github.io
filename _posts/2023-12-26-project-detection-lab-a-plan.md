---
layout: post
title: Project - Detection Lab - A Plan
categories: ["Projects"]
tags: ["Detection Lab", "Terraform", "AWS", "VMWare", "Splunk"]
author:
  - MaximumPigs
---

What is a GitHub account without a perpetually ongoing project?

![Infinite Project](/assets/images/infinite_github.webp "Please make me a picture that embodies a never-ending GitHub project. Do not include any words on the picture. Make sure it will look good against a black background and make it 1024wx512h pixels in size.")  
*Kind of looks like a mask, but it's meant to represent an infinitely long GitHub project.*

First of all, I've decided to add my ChatGPT prompt for each image within the caption. Hover over the image to get a glimpse into my mind.

I've been thinking a lot about what to write in my first substantial post. With so many topics I'd like to explore, I've decided to feed two birds with one scone and write up a project that has been on my radar for quite some time - A Detection Lab. More specifically, forking [Chris Long's (Clong) Detection Lab](https://https://github.com/clong/DetectionLab), which is now unmaintained, and making it work for my purposes. It'll be a significant challenge to my Infrastructure as Code skills, as I will need to deconstruct what Chris has built and then remake at least some of it. I may ditch some features to reduce complexity in the early stages but will likely add them back later.

So, why don't I just use what Clong has already built? It's all still there...

Well, the answer is that Chris's AWS AMIs are no longer publicly available. This means I can't deploy to AWS as is. I'll need to create my own AMIs based on the local OVAs created through a Vagrant deployment. This will be the easiest method; the other way is to start with an unprovisioned OS image and provision it at build time. I'll probably go with the former just to get it into a usable state, but then work on the latter to make the provisioning step provider agnostic. The reason I'd like it to be provider agnostic is so I can deploy one at work on a VMware cloud, and there isn't a VCD option in the existing code. So, this will be the second stage of the project.

Clong has done some incredible work. I've used his Detection Lab many times, and all credit goes to him for the initial build *obviously*. I will be leveraging it a lot in my fork. I intend to use Terraform for the infrastructure build, and either the built-in SSH provisioner or Ansible for the host provisioning. Whichever way I go, I want to ensure it remains a "one-touch" deployment. So, a single GitHub action is all it should take to bring the whole thing up.

So, what am I going to do with it?

I'm going to use it for this blog! I plan on demonstrating some threat hunting techniques, as well as some of my detection engineering methodology. With this lab, I can prepare some logs and query them with Splunk. I'll also be using it professionally to simulate malicious behaviour and identify artifacts to use in detection queries.

Alright, that's a bit of a high-level overview - but now how am I going to do it?

With a list! Of course. Something immediately obvious about me is just how much I love lists. They are my second favourite thing on the planet, only bested by the satisfaction of ticking items off lists. So without further ado... a list.

- [X] Write a post about starting a project.
- [X] Don't procrastinate.
- [X] Create a fork of Clong's Detection Lab.
- [ ] GitHub Actions
  - [ ] Create some actions that will provision to cloud providers using a GitHub Runner.
  - [ ] Modify the AWS Terraform code to accept GitHub.Secrets.
  - [ ] Review AWS Terraform code for any references to things stored locally - Modify for GitHub Actions.
  - [ ] Test it.
- [ ] AWS - AMI
  - [ ] Run a local Vagrant provision.
  - [ ] Upload OVAs to S3.
  - [ ] Create AMIs from the OVAs.
  - [ ] Test it.
- [ ] AWS - Provision
  - [ ] Create a new folder for this, as I want to preserve the AMI method.
  - [ ] Find some basic OS AMIs that I can use.
      - [ ] Windows 10.
      - [ ] Server 2016.
      - [ ] Ubuntu-20.04.
  - [ ] Modify Terraform code to include a provisioner (either SSH or Ansible).
  - [ ] Decide whether to use scripts from Vagrant or Ansible playbooks from the Azure method.
  - [ ] Write in any modifications to match the new provider.
  - [ ] Test it.
- [ ] VCD
  - [ ] Plan out the build, identify what resources are required and the VCD equivalent.
  - [ ] Write Terraform Code.
  - [ ] Provision via the same method as the AWS step.
  - [ ] Test it.

So, there you have it. A list, complete with some easy wins to get me started.

Stay tuned for more words...

---

### Links and References

1. **Ansible - (IT Automation Tool)**: [Ansible by Red Hat](https://www.ansible.com/)
2. **AWS - (Amazon Web Services)**: [AWS Official Site](https://aws.amazon.com/)
3. **Azure - (Microsoft's Cloud Computing Service)**: [Microsoft Azure Official Site](https://azure.microsoft.com/)
4. **Detection Lab - (Security Research Environment)**: [Chris Long's Detection Lab on GitHub](https://github.com/clong/DetectionLab)
5. **GitHub - (Code Hosting Platform)**: [GitHub Homepage](https://github.com/)
6. **GitHub Actions - (CI/CD Automation Tool)**: [GitHub Actions Documentation](https://docs.github.com/en/actions)
7. **Infrastructure as Code (IaC) - (Infrastructure Management Approach)**: [Infrastructure as Code Explained](https://www.ibm.com/cloud/learn/infrastructure-as-code)
8. **SSH - (Secure Shell)**: [SSH Overview](https://www.ssh.com/academy/ssh)
9. **Splunk - (Data Analysis and Visualization Tool)**: [Splunk Official Site](https://www.splunk.com/)
10. **Terraform - (Infrastructure as Code Software)**: [Terraform by HashiCorp](https://www.terraform.io/)
11. **Vagrant - (Software for Building and Managing Virtual Machine Environments)**: [Vagrant by HashiCorp](https://www.vagrantup.com/)
12. **VCD - (VMware Cloud Director)**: [VMware Cloud Director Overview](https://www.vmware.com/products/cloud-director.html)
13. **VMWare - (Cloud Computing and Virtualization Technology Company)**: [VMWare Official Site](https://www.vmware.com/)

---