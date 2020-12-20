---
layout: default
title: Emdee-five-for-life
parent: HackTheBox
nav_order: 2
---
Challenge:

Navigating to the IP in our browswer we get:


![](/flag-embeed.png)

After hopping on the target IP a simple web page pops us with the title "MD5 encrpyt this string" followed by a alphanumeric string and a submit field. 

The challenge here is to be able to submit the md5 encoded string within a very short window of time which can be done with a little scripting.



Solution:

Looking at the challenge it became obvious that in order to be fast enough to complete the challenge I would need to automate this process.

The script I wrote used the libraries Selenium and BeautifulSoup.

Within the Script I included comments that explain the process of what the script does but I will also give a small summary:

1. Opens firefox and navigates to target IP
2. Locates and parses the string to be encoded
3. Encodes the parsed string into a md5 hash
4. Inputs the now encoded string into the submission box and clicks "submit" button
