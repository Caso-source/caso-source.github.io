---
layout: default
title: Emdee-five-for-life
parent: HackTheBox
nav_order: 1
---

Introduction:


Challenge:

Navigating to the IP in our browswer we get:

![alt text](/flag-embeed.png "Flag")
After hopping on the target IP a simple web page pops us with the title "MD5 encrpyt this string" followed by a alphanumeric string and a submit field. 

The challenge here is to be able to submit the md5 encoded string within a very short window of time which can be done with a little scripting.



Solution:

Using the python libraries Selenium and Beautiful Soup a script