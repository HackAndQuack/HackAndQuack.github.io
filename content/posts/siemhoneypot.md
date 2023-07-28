---
title: "The SIEMingly Amazing Honeypot"
date: 2023-07-25T22:23:29-04:00
---
# A Sweet Solution to a Sour Problem

At the beginning of 2021, I began my journey into the world of technology, more specifically, cyber security. Whenever I ran into a technology-based issue in my home, I always saw Cyber Security/Computer Science had the solution to it, which I always found very inspiring. This was one of the many reasons I found myself looking toward this career path.

An example issue I experienced was a bombardment of advertisements on websites I commonly visited (the majority being malicious.) This issue led to the construction of my very first [Raspberry Pi](https://www.raspberrypi.com/) machine, running the software [Pi-hole](https://pi-hole.net/) and [PiVPN](https://www.pivpn.io/). I will certainly delve deeper into that topic and discuss my experience in a future post but the topic of this post revolves more around security and [blue team defense](https://en.wikipedia.org/wiki/Blue_team_(computer_security)).

# The Open Door 🚪

When building my first Rasberry Pi, I had this persistent thought in the back of my mind: *Is this secure enough?* After completing a quick Google search, I found that there are many normal, easy, quick solutions when it comes to defending your hardware from malicious attackers. For example, [disable SSH as root](https://www.howtogeek.com/828538/how-and-why-to-disable-root-login-over-ssh-on-linux/), change the port of the SSHD, make sure random ports are not open, manage users, and update software regularly. Even though I did those suggested actions and more, I found that as I read more, I quickly become engrossed in the very interesting topic of cybersecurity defense.

# A Sticky Situation 🍯

> "Some of the things that people do are very clever and crafty, and they can be great little infinite loops for attackers to fall into" -Neil Weitzel, director of security research at Cygilant

As I sunk deeper into the rabbit hole of cyber security defense, I discovered the concept of honeypots. Wikipedia defines honeypots as "computer security mechanisms set to detect, deflect, or, in some manner, counteract attempts at unauthorized use of information systems." These honeypots mimic likely targets for cyber attacks like a website, and FTP (File Transfer Protocol) server, fake ports, or an AD (active directory). When I started understanding how honeypots can be useful within a network's infrastructure it led to me implementing them into my very own network. In future blogs, I will go deeper into other honeypots and the process of me implementing them but for now, I wanted to go over my current personal favorite.

## [Cowrie](https://github.com/cowrie/cowrie)

> Cowrie is a medium to high interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by the attacker. In medium interaction mode (shell) it emulates a UNIX system in Python, in high interaction mode (proxy) it functions as an SSH and telnet proxy to observe attacker behavior to another system.

To explain it without all of those difficult words, Cowrie acts like an [SSH](https://en.wikipedia.org/wiki/Secure_Shell) service and will pretend to be one and document what the attacker is doing. I started up a fresh Rasberry Pi without any applications installed and thought to myself if I was a hacker my first step would be reconnaissance. Running an [nmap](https://nmap.org/) scan would be something I would naturally do to see what ports would be open and vulnerable. Naturally, since SSH would be enabled, nmap would show that port 22 would be opened. [Brute Forces](https://www.fortinet.com/resources/cyberglossary/brute-force-attack) are a common attack you may see from a hacker, so being aware of the situation, I had to find a way not only to avoid being attacked this way but also, to document my findings!

## Following the Paper Trail

When initially setting up Cowrie I had two main objectives, diverting the hacker from my actual services, and documenting what attacks they are doing. The outcome in the end was that Cowrie was able to deliver on both of those plus so much more! The first thing I did was move my actual SSH port to a different port and then moved my Cowrie service to port 22. (Even though the service identifies as "ibm-db2" in actuality it is my real ssh port")

Once set up, I started configuring the login information.

When configuring the login information I wanted to do something common the attacker may try something such as logging in as root. I also designed it as if an attacker may know I little information about me such as my name Ben and that I love ice cream.

I then decided on creating fake files within this fake environment. I started by putting a fake file name SecretFile and just putting false information on it. Cowrie already setup a user called Phil and I put a fake file within the environment.

This is me creating the "SecretFile"

Once the attacker got into the Cowrie environment and tried messing around with it Cowrie was able to capture those actions and be able to compile them into a log!

# It SIEMs perfect, right?

I had achieved my goal of building a honeypot that allowed me to divert the attention of the hacker but, there were still two questions on my mind. How much is too much when it comes to honeypots and what happens if I wanted to expand on a much larger scale? I decided to touch on the ladder question and found myself thinking about the logs, and how to quickly document them, which led me to SIEMs. [SIEM](https://www.ibm.com/topics/siem) also known as (Security Information and Endpoint Management) allowed me to automate many of the manual processes associated with threat detection and incident response, which included the logs I gathered with Cowrie. After doing a lot of research I landed on either using [Splunk](https://www.splunk.com/) or [Wazuh](https://wazuh.com/). I then decided to try both of them out on [TryHackMe](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwim2ZaskaCAAxU7EVkFHai2AogQFnoECB8QAQ&url=https%3A%2F%2Ftryhackme.com%2F&usg=AOvVaw2gvAQndb0wWgMfXtgVVb0c&opi=89978449) since they have virtual environments to try them out with. Both felt great, but I decided to go with Wazuh because of its community, price, as well as being open source.

The first step in incorporating Wazuh into my network was figuring out how to even host it. I had two options to either host it virtually or locally within my network. I decided since I never learned too much about hosting on the cloud and was interested in it, I decided to work with [Linode](https://www.linode.com/lp/free-credit-100/?promo=sitelin100-02162023&promo_value=100&promo_length=60&utm_source=google&utm_medium=cpc&utm_campaign=11178784975_109179237043&utm_term=g_kwd-2629795801_e_linode&utm_content=466889955310&locationid=9009979&device=c_c&gclid=Cj0KCQjw2eilBhCCARIsAG0Pf8vkA7dNc2eU3GJqBhZQvbqzStacXPHI0XjFoRuQApMWMdh4CdDgL_YaAnCtEALw_wcB). After deploying the service with ease I was able to log in and be able to finally start playing with it!

## The mission, if you choose to accept it

So how does Wazuh collect logs? Wazuh works as a manager to agents. Agents are like little spies in your computers you set up to tattle or secretly tell the manager all the logs within that device, and Wazuh will collect, and manage them. So the first plan is to set up my Rasberry Pi to be an agent and monitor with Wazuh. With just a few simple commands I was able to add it to my collections of agents and configure it.

When working with Wazuh I documented what modules I found myself using and my reasonings for why they helped further my understanding in Blue Team.

## Security Events

With these modules, I was able to view certain threat alerts such as authentication monitoring. What piqued my interest was the timesteps which at the default 30 minutes allowed me to get exact seconds of when certain events happen.

## Security Configuration Assessment

With this module, I can assess my configurations and compare them to [MITRE's](https://www.mitre.org/) common vulnerabilities to show what I need to fix within my system.

As you can see the ID vulnerability 29652 "Ensure SSH access is limited" notified me that it failed to pass the vulnerability assessment and explained to me not only how to patch it, but also why this is a vulnerability.

I was able to patch it and it gave me this response!

## Vulnerabilities

The last Wazuh module I wanted to talk about was Agent Vulnerabilities. Like the SCA (Security Configuration Assessment) it scans your entire agent's machine and will respond if it sees any software or file that can scale from low severity up to critical and will explain how to respond to the alert.

Since it was just the honeypot software that was installed on my machine no vulnerabilities were found which is just what I want to hear!

# Taking a Break and Slacking Off

When working on the project I found myself constantly checking back on the event manager checking if it picked up my attack on the honeypot. What I noticed was that it would be better if I could just be notified either on my phone or email that an attack is even going down. I then found out that you can integrate external API's to Wazuh. I started to read up on the documentation and found [Slack](https://slack.com/) and decided to go with implementing it. I first had to create a channel in Slack which my API is going to interact with and then get a [webhook](https://www.redhat.com/en/topics/automation/what-is-a-webhook) and install it within the configuration files of Wazuh.

The code was easy to add and after running the script I checked my Slack and got the notification!

Interestingly I also got attack alerts from my Wazuh manager meaning someone was attacking my Linode given IP address. I took the IP address that was attacking me and threw it into [Shodan](https://www.shodan.io/) to see where that attack is coming from.

# Final Thoughts

Configuring a honeypot to bait attackers as well as SIEM Integration within my network, I learned much about Blue Team Security. Im going to keep on improving the logging system I have created as well adding a ton more honeypots! I do want to find the answer to the question "When is it too much when it comes to honeypots" and hope to find it the more I study and work on my Blue Team Skills.

A big thank you to everyone who helped me get to this point, writing my first blog, and pushing me to always do my best. Sam T, Jim K, Dan W, Willie P, Jamie B, Scott J, Paul Z, Ava K, and so many more thank you so much.
