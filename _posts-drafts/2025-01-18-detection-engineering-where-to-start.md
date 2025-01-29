---
layout: post
title: Detection Engineering - Where to start?
categories: ["Cyber Security"]
tags: ["cyber-security", "detection-engineering", "DFIR", "threat-hunting"]
author:
  - MaximumPigs
---

Do you want to start writing effective detection rules? Start here.

![Image Title](/assets/images/ "ChatGPT prompt")  
*Image caption*  
<br />

I've heard it so many times that I've lost count. "I want to get into detection engineering, but I don't know where to start".

With that statement, I can sympathise. I spoke the same words verbatim when I first began my journey in the field, and I was given the same advice which I have given on several occasions. 

**"Just start"**

I know, that's about as useful as a screen door on a submarine and I felt equally as terrible giving only those two words of advice as the person I first received them from likely felt. So why do we all seem to give the same useless advice? For me, I think I struggled to answer because being asked "where do I start?" in the context of detection engineering musters up the same overwhelming sense of futility I felt when I first asked the question myself. There is simply so much ground to cover, too much to point anyone in a single direction and be confident that they will reach their destination.

I've thought about this from time to time. What guidance can I give to those who are seeking a pathway into detection engineering? or if I rephrase that ... "I want to guide people into detection engineering, but I don't know where to start".

Having gone full circle and arrived back to where I began, I've decided to take my own advice and **"Just start"**.

So here it is, my methodology for detection engineering.

## Intelligence

Before we can start, we need intel. Intel is everywhere, every major cyber security vendor is posting intelligence - but generally not the intel we need for this exercise.  
They type if intel often plastered all over the internet could be regarded as strategic intelligence, it tells a broad story about general threats to assist decision makers in directing their efforts.
This type of intelligence doesn't do too much to help us detect malicious activity though. The kind of intel we need is Operational intelligence.

Operational intelligence is highly technical in nature, and it provides highly specific details of a threat actors Tactics, Techniques and Procedures (TTPs) which makes it perfect
for identifying malicious activity within an environment.

There are two common places we can obtain Operational intelligence.

1) By being on the receiving end of an attack in progress.
1) By studying the events of a victim who has been attacked.

With option 1 being far too late to use for pre-emptive detection engineering, option 2 becomes our best source.

There is one big problem with option 2 though. Who is going to release that information?  
Unless we are on the inside of an Incident Response team, we will probably never see what occured outside of a PR team spun media release and speculation.

Enter [The DFIR Report](thedfirreport.com).  

![The DFIR Report](/assets/images/detection-engineering/thedfirreport.jpg "The DFIR Report logo")  

For those just beginning their journey, all the way to seasoned veterains, The DFIR Report is an incredibly valuable resource with highly technical details of (anonymised) real world compromise.

The DFIR Report offer many services which you may wish to avail yourself of, but the most important for this discussion is their public release of a detailed compromise breakdown every few months - and trust me, each report will keep you busy until the next one.

If you are going to start anywhere, I suggest you start here.

## Tactics, Techniques and Procedures - The three most important questions.

You can probably already see where I'm going with this post. In just one click, you've now got access to an extensive list of Indicators of Compromise (IoCs) and if you're even half savvy in querying your flavour of SIEM you've probably got enough here to create a handful of new detection rules.

Before you run off and create a bunch of rules to detect any of your hosts reaching out to any of the IP addresses listed in the report, I urge you to read on.

For this section, I am going to use The DFIR Report [Cobalt Strike and a Pair of Socks lead to Lockbit Ransomware](https://thedfirreport.com/2025/01/27/cobalt-strike-and-a-pair-of-socks-lead-to-lockbit-ransomware/).

I'm sure you've already heard everything there is to hear about IoCs vs TTPs, so I won't bore you again with the details - the crux of it is that IoC's regularly change from compromise to compromise, whereas TTPs can be used to detect similar activity repeatedly.

The key step in this whole process is to extract the TTPs from the report, for use in your detection rules.

I've personally found that a great way to achieve this is to first isolate a specific step within the report - I'll focus on Persistence, using Scheduled Tasks.
Keeping in mind that we are looking to only detect a single step, a lot fall into the trap of trying to find everything with a single rule. I'll tell you from experience, that doesn't work.

Once you've had a read and understood what has occured, ask yourself 3 simple questions.

### **1) How can I describe this as simply as possible?**

   By thinking about a step in the simplest terms possible is. The point of answering this question is to give you a place to start looking.

   ![Scheduled Tasks](/assets/images/detection-engineering/scheduled_tasks.jpg "Process command line of scheduled task creation, using schtasks.")  

   Starting with this example, you would not be faulted describing this step as "Use of the *schtasks /create* command to create a scheduled task which leverages rundll32 to call a function from a malicious library file"

   However, what I mean by "as simply as possible", is to describe it like this:

   *"Creation of a scheduled task"*

   There is nothing more to it than that. A scheduled task was created. The contents of the task, and the method used to create it is largely irrelevant at this stage. All we need to know is that the malicious actor created a scheduled task.

### **2) What would this look like if all of the IoCs were changed?**

   This step will either test your existing knowledge of event logs, or your ability to google.

   If you're familiar with Windows logging, you'll have recognised the pictured event above as a Process Creation event. Either Windows Event Id 4688, or Sysmon Event Id 1. My best advice to you is not to start here. While process creation events are a wealth of information, they suffer from a lot of variation. The same activity performed in a slightly different way can present differently.

   Instead, look for something similar - and this is supported by your answer to the first question. All we're looking for is evidence that a scheduled task has been created.

   A quick google search for "windows scheduled task creation event" yields plenty of results. Where possible, I recommend clicking on any links to [Ultimate Windows Security](www.ultimatewindowssecurity.com). I've found this page to be the authority on all things Windows Events.

   In this instance, we are taken to [Event Id 4698](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4698) which simply shows that a scheduled task has been created. The reason we care about this Event Id is the fact that it will log whenever a scheduled task has been created - it doesn't care if it was created using the command line or the graphical interface, it also doesn't care what parameters were set, or what command it will call. It will log in all instances.

### **3) How does this differ from normal activity?**

   So now we know what has occured, in as simple terms as possible. We also know that it will present as an Event Id 4698 within the Windows Event Logs.

   Anybody who has attended a SANS course in their life has probably heard the term "Know normal, find evil". For this step, you need to first "know normal".

   It's time to begin your hunt. Pick up your favourite SIEM query language and start looking!

   Search for all the 4698 events and start reading them. Look for similarities and differences. This is the perfect time to start grouping events by common fields. 

   - How many unique task names exists?
   - How many unique command lines exist?
   - How many unique argument lines exist?

   Basically, what does a normal scheduled task look like in your environment?

   I'm willing to bet there are hundreds. You're definitely going to have tasks created by Microsoft and Google, and you've probably got a bunch created by every other piece of third party software running in the environment as well but ... 

   ![Command and Arguments](/assets/images/detection-engineering/scheduled_task_command.jpg "Scheduled Task Command and Arguments from task XML")  

   - How many of them start by executing the command "cmd"?  
   - How many of them contain rundll32 in the arguments?  
   - How many of them call files from the "users\public" directory?

   Probably not too many, if any at all. These things would be considered "abnormal" in most environments.

   Remember to avoid using things like filenames, IP addresses, hashes etc. Under no circumstance should you be searching for "svcmc.dll" or "MainFunc".

   Finally, if you were to look for 4698 events within your environment, with any of the above criteria (one at a time) - would your search return 0 results? What if you expanded your search to cover the last 6 months? (a few false positive)

   If so, you might have found yourself one (or three) new detection rule(s) to create.

   - Scheduled task created executing CMD
   - Scheduled task created containing runddl32
   - Scheduled task created referencing user public directory

   I think it's very important to note at this point that there will be many people asking "What if?" in response to these three rules - and while it's a valid question, it is also a victim of the perfect solution falacy. "Just because something is not perfect, does not mean it is not worth doing". No, these rules will not cover all possible variances of this activity - but they will certainly have detected it in this instance and they are broad enough that they have a good chance of detected similar activity.

## Rule creation

This step is the most fun, in my opinion. Actually crafting a working rule.

If you've already done some searching through your environment, then you're probably 90% of the way there. You just need to put what you've done into a simple query which can be run repeatedly.

I use the term "simple query" intentionally as I have found that the biggest risk to a detection rule is overcomplication. I've seen this lead to logic problems and cause queries to become completely ineffective as a result. Keep them as simple as you can.

Below are three example queries for the above detection rules, and hopefully you won't need to extract any fields like I have - but they are otherwise simple, appropriately broad, and effective.

  ![Command and Arguments](/assets/images/detection-engineering/4698_cmd.jpg "Scheduled Task Command and Arguments from task XML")  

  ![Command and Arguments](/assets/images/detection-engineering/4698_rundll32.jpg "Scheduled Task Command and Arguments from task XML")  

  ![Command and Arguments](/assets/images/detection-engineering/4698_user_public.jpg "Scheduled Task Command and Arguments from task XML")  

Nobody is winning any awards for "Most complex detection query".

## Testing

By now you have your query, but you need to push it to it's limits before you implement it.  

There are exactly two outcomes you want to avoid.

1) Your query triggers when you don't want it to.
1) Your query doesn't trigger when you want it to.

The second is FAR worse than the first, but also the hardest to monitor. You'll be conveniently notified every time your rule triggers incorrectly, but if it fails to trigger you may never find out. This fear keeps me up at night - but there is plenty you can do to prevent it.

Avoiding unfavourable outcome number 1 is easy. Just open up your search to the longest time period possible, let it run and count how many results you get. This is the number of alerts you can expect to receive in the future too. Is it too many? Only you can decide that - adjust the query as necessary.

Avoiding unfavourable outcome number 2 is much harder. You need to make sure your logic is bulletproof, it's usually at this stage I suggest you have someone else read your query and try and find fault with it. It doesn't matter how experienced you are, a second set of eyes is always a good thing - and they've saved me many times.

Here are some suggestions on how you can "stress test" your query:

1) Reverse the logic and check that you are getting the OPPOSITE results.  
   *If you are looking for the string "rundll32" and you are returning results as expected, change your query to EXCLUDE "rundll32" and make sure you are returning everything except the results you want. - If not, there might be something wrong with your query. I've found and fixed a lot of potential problems this way.*
1) Case sensitivity.  
   *If you are looking for the string "rundll32" and you're returning results as expected, try changing your search to look for "runDLL32" and see if you're still getting the same expected results.*
1) Combined logic problems.  
   *If you've unavoidably used multiple logic operators in your query (AND, OR, NOT), make sure your parentheses are correctly placed. This is harder than it sounds. X NOT Y AND Z will return events which contain both X and Z but not Y, whereas X NOT (Y AND Z) will return events containing X, but do not contain BOTH Y and Z - however Y OR Z may be present. This comes down to understanding the query language you are using, take the time to understand it.*

Once you are confident, you can now enable your detection rule. Congratulations!