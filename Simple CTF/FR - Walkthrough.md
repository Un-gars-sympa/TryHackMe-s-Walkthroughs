<div align="center">
<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png" height="400"></img>
</div>

# Lien & Description
https://tryhackme.com/room/easyctf

*Beginner level ctf*

# Walkthrough :

## **Première question** : *How many services are running under port 1000?*
Bon commencons par un basic scan Nmap basique :
`Nmap -sC -sV VICTIME_IP`

[![Capture-d-cran-2025-09-24-135308.png](https://i.postimg.cc/FHb6t1Mh/Capture-d-cran-2025-09-24-135308.png)](https://postimg.cc/Yv9RY22P)

Comme on peut le voir, on a 2 services en dessous du port 1000

*How many services are running under port 1000?* : **2**
## **Deuxième question** : *What is running on the higher port?*
Très simple, on regarde le port numéro 2222

*What is running on the higher port?* : **SSH**
## **Troisième question** : *What's the CVE you're using against the application?*
On a besoins de regarder plus profondément la page http qu'on a scanné et on fait un gobuster dessus :
`gobuster dir -u VICTIME_IP -w /usr/share/wordlists/dirb/common.txt`


[![Capture-d-cran-2025-09-24-140414.png](https://i.postimg.cc/vTgQX5SY/Capture-d-cran-2025-09-24-140414.png)](https://postimg.cc/GTRwmTCV)

Bon allons sur le */simple*. On découvre alors un "CMS made simple" avec la version "2.2.8" et le titre "Home - Pentest it", je suppose qu'on est sur la bonne voie.

Après une recherche sur Exploit-DB, j'ai trouvé un payload qui peut être utile : https://www.exploit-db.com/exploits/46635 donc on le télécharge et c'est parti pour l'utiliser

*What's the CVE you're using against the application?* **CVE-2019-9053**
## **Quatrième question** : *To what kind of vulnerability is the application vulnerable?*
C'est marqué sur la page Exploit-DB

*To what kind of vulnerability is the application vulnerable?* : **SQLI**
## **Cinquième question** : *What's the password ?* 
Pour ça, démarrons notre POC avec cette commande : `python2.7 46635.py -u http://VICTIME_IP/simple/ -w /usr/share/wordlists/rockyou.txt -c`

the *-u* the URL, the *-w* for the wordlist and the *-c* for the crack option

Et après avoir attendu, on a les identifiants 

[![Capture-d-cran-2025-09-24-144114.png](https://i.postimg.cc/Wzvn7cnX/Capture-d-cran-2025-09-24-144114.png)](https://postimg.cc/R6gKM82H)

*What's the password ?* : **secret**
## **Sixième question** : *Where can you login with the details obtained?*
C'est juste une déduction mais j'ai supposé que c'était SSH et c'est le cas

*Where can you login with the details obtained?* : **SSH**
## **Septième question** : *What's the user flag?* 
Pour ça, essayons de nous log sur le service SSH avec les identifiants obtenus

`ssh mitch@VICTIM_IP -p 2222`

Et voilà !

[![Capture-d-cran-2025-09-24-144339.png](https://i.postimg.cc/d1Mmqczw/Capture-d-cran-2025-09-24-144339.png)](https://postimg.cc/LqTZTw9b)

*What's the user flag?* : **G00d j0b, keep up!**
## **Huitième question** : *Is there any other user in the home directory? What's its name?*
Facile, on fait un  `ls /home`

*Is there any other user in the home directory? What's its name?* : **sunbath**
## **Neuvième question** :  *What can you leverage to spawn a privileged shell?*
En vrai, je devrais upload **linpeas.sh** pour vraiment voir toutes les possibilités d'évélations de privileges mais je vais d'abord faire un simple `sudo -l`

Et on a notre réponse

[![Capture-d-cran-2025-09-24-145026.png](https://i.postimg.cc/gk5w9H9P/Capture-d-cran-2025-09-24-145026.png)](https://postimg.cc/bsxYkbN6)

Bon allons jeter un oeil sur GTFOBins, un site internet pour indiquer certaines possibilités d'évélations de privileges et on cherche VIM

https://gtfobins.github.io/gtfobins/vim/

Maintenant on suit les instructions pour Sudo : `sudo vim -c ':!/bin/sh'` et on profite de nos privilege en tant que root

[![Capture-d-cran-2025-09-24-145327.png](https://i.postimg.cc/fT3LvSVq/Capture-d-cran-2025-09-24-145327.png)](https://postimg.cc/fJNDzLjY)

*What can you leverage to spawn a privileged shell?* **VIM**
## **Dixième question** : What's the root flag?
On fait simplement un `cat /root/root.txt`

*What's the root flag?* **W3ll d0n3. You made it!**
