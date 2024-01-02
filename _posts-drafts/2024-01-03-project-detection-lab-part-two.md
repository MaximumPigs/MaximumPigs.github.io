---
layout: post
title: Project - Detection Lab - Part One
categories: ["Projects"]
tags: ["Detection Lab", "Terraform", "AWS", "Ansible", "GitHub Actions"]
author:
  - MaximumPigs
---

So I made a commitment to a project, but we all know that planning is the first step towards failure.  

Did I get stuck in the procrastination stage? Did I hyper focus and finish the whole thing in one sleepless sprint? There is only one way to find out ... read on and follow along in version 0.1.0 of this project. [1] 

![Procrastination vs Hyperfocus](/assets/images/procrastination_vs_hyperfocus.webp "Make me a picture that shows a struggle between procrastination and hyper focus. Do not include any words on the picture. Make sure it will look good against a black background and make it in a 3:1 aspect ratio")  
*Kind of looks like my bloodshot eyes after staring at a screen all day. Also, ChatGPT isn't very good at image sizes.*  
<br />

Spoiler: I spent an unhealthy amount of time on this when I really should have been enjoying a few days off work.  
What did I learn about the importance of taking time to relax? ... Absolutely nothing, I'll do it again, just watch me.  

Let's talk about what I've done so far, some of the challenges I came up against and how I overcame them.  
<br />

Here's where we left off last time, a pretty solid start but not worth discussing again.  
- [X] Write a post about starting a project.
- [X] Don't procrastinate.
- [X] Create a fork of [Clong's Detection Lab](https://www.detectionlab.network/).  
<br />

## GitHub actions
This next bit was nice and easy, I've got plenty of templated GitHub workflows.

- [X] GitHub Actions
  - [X] Create some actions that will provision to cloud providers using a GitHub Runner.
  - [X] Modify the AWS Terraform code to accept GitHub.Secrets.
  - [X] Review AWS Terraform code for any references to things stored locally - Modify for GitHub Actions.
  - [X] Test it.  

If you're just beginning with Github Actions, I suggest starting here.

```
Your Terraform Github Repo > Actions > New Workflow > Search for Terraform > Configure > Commit
```

There are a few nice steps I've added in here to help the deployment process along a bit. Whenever I deploy from a public GitHub runner, I will include a step to fetch the runners public IP and export it as a GitHub environment variable.  By exporting it as "TF_VAR_runner_ip" I can then call it in Terraform as var.runner_ip for the purposes of whitelisting it. It needs to be whitelisted so that Ansible can contact the hosts after the infrastructure has been deployed.

```yaml
# https://github.com/MaximumPigs/DetectionLab/blob/v0.1.0/.github/workflows/build.yml  

# Get the public IP of the github runner to use in TF code
- name: Runner IP
  id: Runner_IP
  run: |
    echo "TF_VAR_runner_ip=$(curl ifconfig.me)" >> $GITHUB_ENV
```

While I believe it's fairly low risk leaving this whitelisted, I do like to clean it up in the last step by running a Terraform Apply with a duplication of my own IP address.

```yaml
# Removes Runner access
- name: Terraform Remove runner access AWS
  working-directory: ./AWS/Terraform
  run: terraform apply -auto-approve -var="runner_ip=${{ secrets.MY_IP }}"
```

Another nice thing I've added is the cost calculator from Clong's AWS README.md file. [2] As well as a step summary of the Terraform outputs, for ease of reference later.

```yaml
# Gets Terraform state and generates cost using cost.modules.tf
- name: Terraform Get Cost AWS
  working-directory: ./AWS/Terraform
  run: |
    echo "--------COST---------" >> $GITHUB_STEP_SUMMARY
    terraform state pull |  curl -s -X POST -H "Content-Type: application/json" -d @- https://cost.modules.tf/ >> $GITHUB_STEP_SUMMARY

# Gets Terraform Outputs and displays them in summary
- name: Terraform Display outputs in summary
  working-directory: ./AWS/Terraform
  run: |
    echo "--------OUTPUTS---------" >> $GITHUB_STEP_SUMMARY
    terraform output >> $GITHUB_STEP_SUMMARY
```  

It's also important to note here, that whenever you're running terraform code in the commandline and exporting it to GitHub variables like I have, you must disable the terraform wrapper. I'm not quite sure how this wrapper manipulates the STDOUT of the commands, but I think it adds extra lines - so GitHub variables don't like the format.

Anyway, it's an easy fix.

```yaml
# Just add this "with" parameter.
- name: Setup Terraform
  uses: hashicorp/setup-terraform@v1
  with:
    terraform_wrapper: false
```

Finally, I always add a "Destroy" workflow. There is nothing worse than trying to chase down all of your deployed resources and manually destroying them.  
<br />

## AWS build with pre-provisioned AMIs

With that out of the way, the next hurdle was to get the DetectionLab running from AWS again. As I mentioned in the previous post, Clong is no longer hosting the AMIs - so we need to make our own.

- [X] AWS - AMI
  - [X] Run a local Vagrant provision.
  - [X] Upload OVAs to S3.
  - [X] Create AMIs from the OVAs.
  - [X] Test it.

This was actually much easier than I expected, I just followed Clong's "Customization" steps for building your own AMIs. [3] I was expecting some technical resistance here, but it actually went very smoothly - so hats off to you Clong.

### Velociraptor

I did end up running into two issues with Velociraptor when I was unable to access the portal.
The first issue was an easy fix. Once I found the velociraptor_server.service was not running I did a quick runthrough of the /Vagrant/logger_bootstrap.sh file [4] and found that the latest version of the Velociraptor.deb file had changed its naming convention. A quick change to the logger_bootstrap.sh [5] script and it came good.

```shell
# Original
if dpkg -i velociraptor_*_server.deb >/dev/null; then
  echo "[$(date +%H:%M:%S)]: Installation complete!"
else

# Fixed
if dpkg -i velociraptor_server_*.deb >/dev/null; then
  echo "[$(date +%H:%M:%S)]: Installation complete!"
else
```

Once this was resolved, I attempted to browse to the URL again only to be met with a "Certificate Expired" message. It turns out that the certificates within the client and server configuration file [6] included in the repo expired not long after Clong ended support of DetectionLab.  
With the Lab up and running and the Velociraptor server installed, I just ran through the configuration generation wizard again and dumped the new certificates into my repo.  
I made a couple of changes to the 

It turned out that both the server and client certificates were quite old and had been pre-generated and included within the repo as a static file. I had to add and change a couple of lines to the server config file to make it work.

```yaml
# Change the GUI bind_address from 127.0.0.1 to 0.0.0.0 to allow inbound connections
GUI:
  bind_address: 0.0.0.0
  bind_port: 9999

# Add the vagrant user and password - I'd like to make this user specified eventually.
initial_users:
- name: admin
  password_hash: 1b7fb3a9255f498ff0755355912e6ee5ca218491417087b04a4953c90415e65a
  password_salt: 6e82c2611e99c200ee2dca4d7b3a4d599cfcfb3b023a226b2bb0c4f738a06246
```

I also had to point the Velociraptor client configuration step to my own repo within /AWS/Terraform/scripts/bootstrap.ps1 script [7] to ensure the Windows clients receive the correct files.

### Success ... for now

With these changes made, I destroyed all of my infastructure and set it to rebuild.

Success. I had a successful deployment from start to finish. Velociraptor, Guacamole, Fleet, and Splunk were all available and working as expected.

This isn't where I had planned on stopping though, I had 40gb of .OVA files sitting in my S3 storage and another 240gb of AMIs just waiting around racking up my AWS bill.

The next step is building it up from scratch using existing AMIs in AWS so that I can do away with the associated storage costs of uploading my own.

I would also like to tidy up the code a bit. Files from other build methods are being referenced and I'm not sure I like it. Perhaps a root level folder of shared resources would be best. I'll think about it and let you know.

As of version 0.1.0 I have already started building this out, but I've already rambled quite a bit too much - so I'm going to save that for chapter two.

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
2. **Fleet - (Open-source osquery Management)**: [Fleet for osquery](https://fleetdm.com/)
3. **Guacamole - (Clientless Remote Desktop Gateway)**: [Apache Guacamole](https://guacamole.apache.org/)
4. **Splunk - (Data Analysis and Visualization Tool)**: [Splunk Official Site](https://www.splunk.com/)
5. **STDOUT - (Standard Output Stream)**: [Understanding STDOUT](https://en.wikipedia.org/wiki/Standard_streams#Standard_output_(stdout))
6. **Terraform - (Infrastructure as Code Software)**: [Terraform by HashiCorp](https://www.terraform.io/)
7. **Vagrant - (Software for Building and Managing Virtual Machine Environments)**: [Vagrant by HashiCorp](https://www.vagrantup.com/)
8. **Velociraptor - (Advanced Digital Forensics and Incident Response Tool)**: [Velociraptor](https://www.velocidex.com/)

---

[1]: https://github.com/MaximumPigs/DetectionLab/tree/v0.1.0 "MaxiumumPigs DetectionLab Version 0.1.0"
[2]: https://github.com/clong/DetectionLab/blob/master/AWS/Terraform/README.md "Clong's DetectionLab AWS readme."
[3]: https://www.detectionlab.network/customization/buildami/ "DetectionLab.network - Build your own AMIs"
[4]: https://github.com/clong/DetectionLab/blob/master/Vagrant/logger_bootstrap.sh "Clong's DetectionLab/Vagrant/logger_bootstrap.sh"
[5]: https://github.com/MaximumPigs/DetectionLab/blob/v0.1.0/Vagrant/logger_bootstrap.sh "MaximumPigs Clong's DetectionLab/Vagrant/logger_bootstrap.sh"
[6]: https://github.com/clong/DetectionLab/tree/master/Vagrant/resources/velociraptor "Clong's DetectionLab Velociraptor Config Files"
[7]: https://github.com/MaximumPigs/DetectionLab/blob/v0.1.0/AWS/Terraform/scripts/bootstrap.ps1 "MaximumPigs DetectionLab/AWS/Terraform/scripts/bootstrap.ps1"