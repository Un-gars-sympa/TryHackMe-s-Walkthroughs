<img width="1200" height="1200" alt="image" src="https://github.com/user-attachments/assets/4de86b43-f55e-4e0c-a701-fda0bb5a4655" />


# Link & Description
[https://tryhackme.com/room/gamingserver](https://tryhackme.com/room/greprtp)

*A challenge that tests your reconnaissance and OSINT skills.*

# Walkthrough :
## **First question** : *What is the API key that allows a user to register on the website?*
Ok let's start as usual with a nmap scan and let's see what we got: `nmap -sC -sV VICTIM_IP`

<img width="1016" height="559" alt="image" src="https://github.com/user-attachments/assets/3d96ff53-2dbd-461f-ae20-25465068101a" />

Ok so we have a SSH, HTTP and HTTPS service, I will check first the HTTP page.

The apache2 default page... So let's check the HTTPS page ?

*Forbiden* Mmmmh, I think i'm going too fast so let's do a gobuster on the HTTP page first : `gobuster dir -u VICTIM_IP -w /usr/share/wordlists/dirb/common.txt`

<img width="711" height="499" alt="image" src="https://github.com/user-attachments/assets/e7c36844-4359-459a-9efe-f7189d083479" />

Ok so nothing very interesting, I missed something obviously. and after taken a look futher on the nmap scan, I noticed this on the nmap scan :

`443/tcp open  ssl/http Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
| ssl-cert: Subject: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US`

And we can add `grep.thm` in our `/etc/hosts` to see if anything change

Well nothing change for the HTTP page but for the HTTPS, we can access to the page now :)

<img width="1806" height="766" alt="image" src="https://github.com/user-attachments/assets/b5afa47f-6424-4d14-813f-298f45b8ad39" />

I go on the login page and try the usual admin:admin, admin:password, admin:123456, etc... But I think i'm kinda wrong on that so let's go on the Registrer page

After completing all the informations, I have an error message

<img width="1806" height="1243" alt="image" src="https://github.com/user-attachments/assets/b3be7ab7-a3ff-4553-a088-cdb4b09cfd5f" />

Mmmh so the API key is the problem ? I tried to registrer with a key but nope :/

After a little time of thinking, I noted that we are on *SearchME* and you know what ? Let's do a research on google of it. 

I think I found the Github [page](https://github.com/supersecuredeveloper/searchmecms) . Why this page ? Because I don't know the existance of the *hack* language that compose the repo at 38.6% ahah 

I am browsing the repo and I see the `register.php` in the /API/ folder annnnnnd nothing :/. I mean there is the code of the page and it shows us how it works but no API key or something like that

Oh wait, not nothing because we have a *"Fix: remove key"* so let's go on the fix and voilaaaa we have our key :)

<img width="1784" height="911" alt="image" src="https://github.com/user-attachments/assets/8821cdcb-be3f-4c50-82b9-9719f5c82b21" />

*What is the API key that allows a user to register on the website?* : **ffe60ecaa8bba2f12b43d1a4b15b8f39**
## **Second question** : *What is the first flag?*


*What is the first flag?* : **THM{4ec9806d7e1350270dc402ba870ccebb}**
## **Third question** : *What is the email of the "admin" user?*

*What is the email of the "admin" user?* : **admin@searchme2023cms.grep.thm**
## **Forth question** : *What is the host name of the web application that allows a user to check an email for a possible password leak?*

*What is the host name of the web application that allows a user to check an email for a possible password leak?* : **leakchecker.grep.thm**
## **Fifth question** : *What is the password of the "admin" user?*


A cool CTF, I'm not very strong at OSINT and it was a good way to practice it ;)
*What is the password of the "admin" user?* : **admin_tryhackme!**
