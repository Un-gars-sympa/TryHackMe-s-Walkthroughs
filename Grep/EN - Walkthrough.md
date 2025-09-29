<img width="1200" height="1200" alt="image" src="https://github.com/user-attachments/assets/4de86b43-f55e-4e0c-a701-fda0bb5a4655" />


# Link & Description
[https://tryhackme.com/room/gamingserver](https://tryhackme.com/room/greprtp)

*A challenge that tests your reconnaissance and OSINT skills.*

# Walkthrough :
## **First question** : *What is the API key that allows a user to register on the website?*
Ok let's start as usual with a nmap scan and let's see what we got: `nmap -sC -sV VICTIM_IP -p-`

<img width="916" height="592" alt="image" src="https://github.com/user-attachments/assets/2f3bdbe4-7fbc-4f22-af6c-028980f9fb8c" />

Ok so we have a SSH, 2 HTTP and HTTPS service, I will check first the HTTP page.

The apache2 default page... So let's check the HTTPS page ?

And how about the HTTP on the 51337 port ? "400 Bad Request". Maybe it's gonna be useful for later

*Forbiden* Mmmmh, I think i'm going too fast so let's do a gobuster on the first HTTP page first : `gobuster dir -u VICTIM_IP -w /usr/share/wordlists/dirb/common.txt`

<img width="711" height="499" alt="image" src="https://github.com/user-attachments/assets/e7c36844-4359-459a-9efe-f7189d083479" />

Ok so nothing very interesting, I missed something obviously. and after taken a look futher on the nmap scan, I noticed this on the nmap scan :

`443/tcp open  ssl/http Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
| ssl-cert: Subject: commonName=grep.thm/organizationName=SearchME/stateOrProvinceName=Some-State/countryName=US`

And we can add `grep.thm` in our `/etc/hosts` to see if anything change

We can access to the HTTPS page now 🙂

<img width="1806" height="766" alt="image" src="https://github.com/user-attachments/assets/b5afa47f-6424-4d14-813f-298f45b8ad39" />

I go on the login page and try the usual admin:admin, admin:password, admin:123456, etc... But I think i'm kinda wrong on that so let's go on the Registrer page

After completing all the informations, I have an error message

<img width="1806" height="1243" alt="image" src="https://github.com/user-attachments/assets/b3be7ab7-a3ff-4553-a088-cdb4b09cfd5f" />

Mmmh so the API key is the problem ? I tried to registrer with a key but nope 🫤

After a little time of thinking, I noted that we are on *SearchME* and you know what ? Let's do a research on google of it. 

I think I found the Github [page](https://github.com/supersecuredeveloper/searchmecms) . Why this page ? Because I don't know the existance of the *hack* language that compose the repo at 38.6% ahah 

I am browsing the repo and I see the `register.php` in the /API/ folder annnnnnd nothing 🤡. I mean there is the code of the page and it shows us how it works but no API key or something like that

Oh wait, not nothing because we have a *"Fix: remove key"* so let's go on the fix and voilaaaa we have our key 🙂

<img width="1784" height="911" alt="image" src="https://github.com/user-attachments/assets/8821cdcb-be3f-4c50-82b9-9719f5c82b21" />

*What is the API key that allows a user to register on the website?* : **ffe60ecaa8bba2f12b43d1a4b15b8f39**
## **Second question** : *What is the first flag?*
Well, let's continue our journey, now that we have the right API key, I think we are allowed to register an account so I changed the API key with Burp Suite and I tried to register a account

<img width="1163" height="1198" alt="image" src="https://github.com/user-attachments/assets/966515aa-f34b-4f7c-8e46-5488bbb359fd" />

And let's gooo, it works !

<img width="1870" height="733" alt="image" src="https://github.com/user-attachments/assets/dc973b46-0172-4a8b-870c-941e1a00f8c0" />

Logged with our account, we have our first flag 🙂

<img width="1903" height="889" alt="image" src="https://github.com/user-attachments/assets/68c9fa00-4b83-4ebb-a73d-b10576035905" />

*What is the first flag?* : **THM{4ec9806d7e1350270dc402ba870ccebb}**
## **Third question** : *What is the email of the "admin" user?*
Now I don't what to do :|, I mean, I am already in the dashboard, and I can't go anywhere else (except logging out but I don't think it can be useful for now)

I think we need to do some research (again)

Oh wait, I just saw a **upload.php** on the repo, so I supposed there is the same on the HTTPS page, let's take a look.

So now we have a upload page that can be useful for a reverse shell and as we can see, there an extension filter. So first we need a custom reverse shell

<img width="3574" height="1451" alt="image" src="https://github.com/user-attachments/assets/e321e454-fa52-47a2-95ed-f907b5f7c80c" />

For the reverse shell, I'm using [Revshell](https://www.revshells.com) and the PHP PentestMonkey, so I put my IP and the port 4444 and copy/past into a *shell.php*

<img width="2418" height="1187" alt="image" src="https://github.com/user-attachments/assets/c7456314-6428-43fb-8905-0cd74ebab259" />

Like I said, if we try to upload our .php file, the extension filter will stop us like a sherif so we need to bypass that.

Actually, the extension filter for this page check the magic number of the file (to make it simple, very file extension have his "signature" with bytes at the begging of the file, for example, .jpg -> ff d8 ff e0, .gif -> 47 49 46 38, ...) So the idea is to make the filter believe that we upload a .jpeg instead of a .php. For that, we are going to modifie our .php first

We add 8 bytes and after 4 bytes at the begging of our reverse shell like this (you will see why) : 

<img width="452" height="265" alt="image" src="https://github.com/user-attachments/assets/a78a7e58-e93f-425a-9761-79fdc94ae4b3" />

After, we can modifie the magic number of our file, for that, we are going to use hexeditor : `hexeditor shell.php` and we modifie the first 8 characters with the magic number of a .jpeg

<img width="1807" height="321" alt="image" src="https://github.com/user-attachments/assets/0372d5a6-0a39-4023-82c6-2e4a301f35f5" />

Because we modified the begging of our file that's why we added the 8 & 4 bytes, for preserving the code php code.

And now if we want to verifie what type is this file with the command `file` we have that : 

<img width="234" height="99" alt="image" src="https://github.com/user-attachments/assets/e7395da9-a73e-4094-a4f2-794887af12ff" />

So now let's try to upload our reverse shell annnnd yatta 🥳!

<img width="533" height="217" alt="image" src="https://github.com/user-attachments/assets/46d9256b-03f1-485c-ac96-1f1c55078e6f" />

Now, where the shell goes ? 

On the "upload.php" from the repo, we can see : "$uploadPath = 'uploads/';" so I supposed that our .php goes there and that's right. 

So before opening the .php file, I start a Netcat listener with a `nc -lvnp 4444` and TADAM, we have our reverse shell :)

<img width="1806" height="383" alt="image" src="https://github.com/user-attachments/assets/1c294f69-46ee-4927-a713-1af3fa411322" />

Well, my first move was to go to /home and check the content and nothing 🤡 so let's go on /var/www to see a config.php or something else that can contains information about the owner/admin of the http page.

So in the /var/www folder, we have a *backup* folder, and inside, there is a *users.sql* mmmmh, so let's `cat`this file and I think we have our answer with this email address : 

<img width="1101" height="1028" alt="image" src="https://github.com/user-attachments/assets/41114291-f97e-4741-aaca-87f5b100a86d" />

*What is the email of the "admin" user?* : **admin@searchme2023cms.grep.thm**
## **Forth question** : *What is the host name of the web application that allows a user to check an email for a possible password leak?*
We checked to backup folder but let's check the other folders/files ?

I check the *certificate.crt/.crs* and no host name. 

After I checked the *default_html/index.php* and it's just the default page of ubuntu.

I check all the folder */html* and nothing very interesting, just the structure of the internet page...

I check the *leak_certificate.crt/.crs* and still no host name.

Mmhh so I tried to open all the rest of the files in the *leakchecker* folder I don't have the permission. mmmmmmh

<img width="417" height="409" alt="image" src="https://github.com/user-attachments/assets/01ae2f73-1e7a-476c-9c50-86390269a882" />

So back to the start 🤡. Maybe the *leakchecker* folder have a link with the HTTP service on the 51337 port, now that we have register "grep.thm" in the */etc/hosts* file, maybe we can acces to it 😲

Still nothing 🥶

<img width="464" height="245" alt="image" src="https://github.com/user-attachments/assets/03d4f3cd-628d-4e34-b720-e07dada2138b" />

And after I checked the SSL certification I got my answer

<img width="411" height="237" alt="image" src="https://github.com/user-attachments/assets/8dedf10a-2d68-40d7-8e22-ea1b5d6f52e8" />

So let's add the **leakchecker.grep.thm** in the */etc/hosts* file to see if it's work and yep that's it !

<img width="1641" height="485" alt="image" src="https://github.com/user-attachments/assets/ec0e4ddc-4a69-4740-a06b-78ea32e69c17" />

*What is the host name of the web application that allows a user to check an email for a possible password leak?* : **leakchecker.grep.thm**
## **Fifth question** : *What is the password of the "admin" user?*
Sooo we need a email so let's take the email from the previous anwser and past it as easy it is

And that's it, we're done here !

<img width="787" height="445" alt="image" src="https://github.com/user-attachments/assets/0ddb549d-2726-43ba-8680-18f2ed0b9b74" />

A cool CTF, I'm not very strong at OSINT and it was a good way to practice it ;)

*What is the password of the "admin" user?* : **admin_tryhackme!**
