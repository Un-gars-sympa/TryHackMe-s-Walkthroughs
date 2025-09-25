<img width="1200" height="680" alt="image" src="https://github.com/user-attachments/assets/ff9397cb-a5b7-4a0b-9fce-cafb85525308" />

# Lien & Description
https://tryhackme.com/room/gamingserver

*"You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!"*

# Walkthrough :
## **Première question** : *Deploy the machine.*
*Deploy the machine.* : **No answer needeed**

## **Seconde question** : *Find open ports on the machine.*
Commencons par un scan classique avec nmap : `nmap -sC -sV VICTIME_IP`

<img width="763" height="627" alt="image" src="https://github.com/user-attachments/assets/37dd4886-240a-400f-9287-c8313969234d" />

On note qu'on a un service FTP, SSH and HTTP d'ouvert

*Find open ports on the machine.* : **No answer needeed**
## **Troisième question** : *Who wrote the task list?*
Okkkk, zéé partiii, on va y aller avec le service FTP : `ftp VICTIME_IP`

On essaye de se connecter en tant que **anonymous** et comme on peut le voir, il y a 2 documents

<img width="600" height="284" alt="image" src="https://github.com/user-attachments/assets/40e43244-31d6-4848-a0b3-38dc54ef6fb5" />

On télécharge les documents avec la commande `get` et regardons ce qu'on vient de prendre

*locks.txt* on dirait une wordlist, on garde ça pour le futur

*task.txt* c'est une note signée par **lin** donc on a notre réponse

<img width="378" height="616" alt="image" src="https://github.com/user-attachments/assets/12d4c6c1-6aee-4e85-9335-8849bf0356d1" />


*Who wrote the task list?* : **lin**

## **Quatrième question** : *What service can you bruteforce with the text file found?*

Mmmh une supposition facile, le service SSH

*What service can you bruteforce with the text file found?* : **SSH**

## **Cinquième question** : *What is the users password?*

Bon pour brute force le service SSH, on va devoir utiliser hydra avec cette commande (on utilisera evidemment d'abord la wordlist qu'on vient d'obtenir)  `hydra -l lin -P locks.txt ssh://VICTIM_IP`

Et on a notre réponse !

<img width="1710" height="231" alt="image" src="https://github.com/user-attachments/assets/0ab299e4-0600-4b2e-af57-06291af205fb" />

*What is the users password?* : **RedDr4gonSynd1cat3**

## **Sixième question** : *user.txt*

Connectons nous au serveur avec les identifiants obtenu et on a notre premier flag

<img width="998" height="487" alt="image" src="https://github.com/user-attachments/assets/b286fad9-3e78-4540-b7f5-c384998bfa8d" />

*user.txt* : **THM{CR1M3_SyNd1C4T3}**

## **Septième question** : *root.txt*

Normalement j'upload linpeas.sh pour l'escalade de privilege mais quand j'ai le mot de passe de l'utilisateur, j'aime bien faire un `sudo -l` pour voir si je peux pas gagner du temps eeeeeeeet je crois qu'on a quelque chose

<img width="945" height="129" alt="image" src="https://github.com/user-attachments/assets/63f52b36-a6a8-4aef-a407-96044ee5a267" />

Regardons sur [GTFOBins](https://gtfobins.github.io/) et précisement sur la page de [tar](https://gtfobins.github.io/gtfobins/tar/)

Plus qu'à suivre les instructions de la catégorie sudo et voilà !

<img width="923" height="118" alt="image" src="https://github.com/user-attachments/assets/87b4f7a3-d081-450e-8516-18eb7beb32e7" />


*root.txt* : **THM{80UN7Y_h4cK3r}**

Probablement le CTF le plus facile que j'ai fais
