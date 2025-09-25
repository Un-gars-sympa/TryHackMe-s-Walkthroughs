<img width="1200" height="680" alt="image" src="https://github.com/user-attachments/assets/ff9397cb-a5b7-4a0b-9fce-cafb85525308" />

# Link & Description
https://tryhackme.com/room/gamingserver

*"You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!"*

# Walkthrough :
## **First question** : *Deploy the machine.*
*Deploy the machine.* : **No answer needeed**

## **Second question** : *Find open ports on the machine.*
Let's begging with a regular nmap : `nmap -sC -sV VICTIM_IP`

<img width="763" height="627" alt="image" src="https://github.com/user-attachments/assets/37dd4886-240a-400f-9287-c8313969234d" />

And we note that we have a FTP, SSH and HTTP service open

*Find open ports on the machine.* : **No answer needeed**
## **Third question** : *Who wrote the task list?*
Okkkk, let's start the dig with the FTP service : `ftp VICTIM_IP`

And let's try to log as **anonymous** and as we can see, there is 2 documents

<img width="600" height="284" alt="image" src="https://github.com/user-attachments/assets/40e43244-31d6-4848-a0b3-38dc54ef6fb5" />

We download the documents with the  `get` command and let's take a look at what we have here

*locks.txt* looks like a wordlist, let's keep it for the futur

*task.txt* is a note signed by **lin** so we have our answer

<img width="378" height="616" alt="image" src="https://github.com/user-attachments/assets/12d4c6c1-6aee-4e85-9335-8849bf0356d1" />


*Who wrote the task list?* : **lin**

## **Forth question** : *What service can you bruteforce with the text file found?*

Mmmh an easy guess, the SSH service

*What service can you bruteforce with the text file found?* : **SSH**

## **Fifth question** : *What is the users password?*

So for brute force the SSH service, we are going to use Hydra with this command `hydra -l lin -P locks.txt ssh://VICTIM_IP`

And we have our answer !

<img width="1710" height="231" alt="image" src="https://github.com/user-attachments/assets/0ab299e4-0600-4b2e-af57-06291af205fb" />


*What is the users password?* : **RedDr4gonSynd1cat3**

## **Sixth question** : *user.txt*

Let's connect us to the server with the credentials obtained and we have our first flag

<img width="998" height="487" alt="image" src="https://github.com/user-attachments/assets/b286fad9-3e78-4540-b7f5-c384998bfa8d" />


*user.txt* : **THM{CR1M3_SyNd1C4T3}**

## **Seventh question** : *root.txt*

Normaly I upload linpeas.sh for the privilege escalation but when I have the user's password, I like to do a `sudo -l` to see if I can gain some time annnnnnnnnd I think I have something

<img width="945" height="129" alt="image" src="https://github.com/user-attachments/assets/63f52b36-a6a8-4aef-a407-96044ee5a267" />

Let's take a look on GTFOBins and precisely on the tar page https://gtfobins.github.io/gtfobins/tar/

Now we need to follow the sudo's instruction and voila !

<img width="923" height="118" alt="image" src="https://github.com/user-attachments/assets/87b4f7a3-d081-450e-8516-18eb7beb32e7" />


*root.txt* : **THM{80UN7Y_h4cK3r}**

Probably the easier CTF I ever made
