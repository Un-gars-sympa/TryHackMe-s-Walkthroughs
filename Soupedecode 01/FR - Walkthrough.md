<div align="center">
<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/618b3fa52f0acc0061fb0172-1753465052754" height="400"></img>
</div>

# Lien & Description
[https://tryhackme.com/room/soupedecode01](https://tryhackme.com/room/soupedecode01)

*Test your enumeration skills on this boot-to-root machine.*

Ce CTF fut pour moi mon premier sur un environnement Active Directory, ce fut très intéressant puisque cela m'a donné l'envie d'explorer davantage les vulnérabilitées sur l'AD. De plus, la [mindmap proposé par Orange Cyberdefense](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg) concernant les pentests sur un environnement Active Directory m'a été bien utile pour ce CTF.
# Walkthrough :

## **Première question** : *What is the user flag?*

Très bien, commençons tout pentest avec de la reconnaissance. Comme tout, je lance donc un scan nmap sur la machine cible `nmap -A -p- IP_VICTIME` :

<img width="895" height="1020" alt="Capture d&#39;écran 2026-07-02 154525" src="https://github.com/user-attachments/assets/c8d869c1-4103-412f-a404-c4ae5f81d862" />

Sur ce scan nous pouvons donc voir le nom de domain : *SOUPEDECODE.LOCAL* ainsi que de nombreux ports d'ouvert.

Je rajoute le nom de domain dans mon fichier `/etc/hosts`

Une fois cela fait, je continue par l'énumaration de dossiers SMB. j'utilise donc smbclient avec le compte guest. `smbclient -L soupedecode.local -U 'guest'`.

<img width="543" height="229" alt="Capture d&#39;écran 2026-07-02 155235" src="https://github.com/user-attachments/assets/e6f37a03-b61c-4269-97d2-900a9fa355ec" />

Je note donc qu'un dossier "backup" été créé. Cela me donne envie de jeter un coup d'oeil dedans. `smbclient //soupedecode.local/backup -U guest` mais on est vite stoppé puisqu'on a pas l'accès à celui-ci.

A partir de maintenant, cherchons un compte à commencer par un username. Pour cela, je fais utiliser NetExec a.k.a nxc qui est une sorte de couteau suisse du hacking. Je vais donc demander le rid des utilisateurs. `nxc smb soupedecode.local -u 'guest' -p '' --rid`

Et voila, on a une sacré liste d'utilisateur. 

<img width="1074" height="1034" alt="Capture d&#39;écran 2026-07-03 104532" src="https://github.com/user-attachments/assets/c76136b2-ca14-4081-906a-a8cb8861ed9a" />

Je copie le résultat que je mets dans un user.txt puis je fais cette commande `awk -F'\\' '{print $2}' user.txt | awk '{print $1}'` pour avoir une liste propre.

Bon, si on reprend la roadmap de OCD, nous avons 2 possibilités : Soit du Password Spraying soit ASREPRoasting, je serai d'avantage tenté de commencer par le second choix.

Je ne vais pas m'étaler sur le sujet de qu'est-ce que l'ASREPRosting mais pour faire simple, si un compte a l'option "Do not require Kerberos preauthentication" d'activé, il peut demander un TGT au près du KDC sans s'identifier. Ce TGT est donc chiffré en partie avec le mot de passe du compte (et donc crackable si le mot de passe est faible).

Il faut donc voir si dans notre big liste, est-ce qu'il existe un compte avec cette option d'activé.

Pour cela, j'utilise GetNPUsers.py de Impacket qui permet de faire des requets de TGT (AS-REQ) en espérant d'avoir un TGT pour un compte `GetNPUsers.py soupedecode.local/ -usersfile user.txt -format hashcat`

Il semblerait qu'aucun utilisateur n'ait l'option d'activée 🤡

Dans ce cas essayons de faire du password spraying. Première option simple qui s'offre à moi, tester si username==password.

Pour cela, j'utilise de nouveau nxc avec l'option --no-bruteforce & --continue-on-success `nxc smb soupedecode.local -u user.txt -p user.txt --no-bruteforce --continue-on-success`

Et bingo !

<img width="576" height="576" alt="6eaaaa0853971f48a4890cf72d4a86e7 576x576x1" src="https://github.com/user-attachments/assets/c151a9fe-45f2-4785-ac9e-88d474e2f2ca" />

<img width="1137" height="65" alt="Capture d&#39;écran 2026-07-03 112947" src="https://github.com/user-attachments/assets/7683903d-ae5f-4553-b0d3-04ae8e3fd478" />

J'en connais un qui va se faire taper sur les doigts, bref username==password => Pas bien 🫢

On est reparti sur de l'énumeration SMB pour voir si notre ami ybob317 a le droit d'aller dans le dossier SMB backup `smbclient //soupedecode.local/backup -U 'ybob317'` et aussi voir nous pouvons direct nous connecter en RDP `xfreerdp /d:soupedecode.local /v:soupedecode.local /u:ybob317 /p:ybob317`.

<img width="1551" height="733" alt="Capture d&#39;écran 2026-07-03 115824" src="https://github.com/user-attachments/assets/57c9d993-1999-4c03-97a5-58820bbdf31c" />

<img width="360" height="270" alt="1514241468-cestnon" src="https://github.com/user-attachments/assets/738928a9-612e-4168-8e81-3e3015f5c536" />

Bon après cette climm, la suite logique serait de prendre l'énumeration du SMB avec le dossier backup mais toujours pas d'accès à celui-ci. Bon dans ce cas alors dans le dossier Users et en allant dans le dossier de notre bon vieux ybob317, on y trouve notre premier flag !

<img width="1359" height="1081" alt="image" src="https://github.com/user-attachments/assets/f5adabbd-fcc2-41a0-b812-734b95dff53e" />

## **Seconde question** : *What is the root flag?* 

Zé parti pour la suiiiiite !

Bon, plusieurs possibilités s'offrent à moi à commencer par la possibilité d'un attaque Kerberoasting.

Pour faire court sur les explications sur ce que c'est le Kerberoasting, chaque service qui doit être accessible possède un compte service dans l'AD (exemple : svc_iis, svc_sql, svc_scanner, svc_...). Le but du Kerberoasting est de demander un TGS de ce service. Même principe que l'ASREPRoasting, le ticket est chiffré en partie avec le mot de passe du compte (et donc crackable si le mot de passe est faible).

Donc pour cela on allons utiliser un autre outil de Impacket : GetUserSPNs.py avec la commande qui va bien `GetUserSPNs.py -request -dc-ip soupedecode.local soupedecode.local/ybob317:ybob317`

<img width="2159" height="1218" alt="image" src="https://github.com/user-attachments/assets/180cfe0f-9507-4cb3-bb9c-d648d2c7911e" />

Donc 5 comptes nous ont donné leur TGS. Je mets les hashs dans un fichier hash.txt et etape suivante : Cracker les hashes avec Hashcat. `hashcat -m 13100 hash.txt /usr/share/wordlist/rockyou.txt`

Tadam !

<img width="1246" height="993" alt="image" src="https://github.com/user-attachments/assets/fcbe18ae-4827-4175-b8da-ca0bf28c0602" />

Aller retestons pour la 3ème fois le dossier backup  sur le SMB 💀 `smbclient //soupedecode.local/backup -U file_svc`

Mais c'est quoi ce poulet ?? 🐔

<img width="1013" height="422" alt="image" src="https://github.com/user-attachments/assets/5e3907ff-cce4-4fde-8df0-790165140290" />

Et bien on a accès à un packet de hash. Je vérifie quel compte peut être vulnérable à du Pass-the-hash.

Avant ça, je vais juste mettre les noms dans un nouveau fichier et les hashes dans un autre histoire de voir quel compte pourrais-je utiliser.

<img width="1551" height="637" alt="image" src="https://github.com/user-attachments/assets/ecb374a8-b502-457d-9582-995d99155480" />

Parfait, reste plus qu'à utiliser PsExec pour se connecter. `psexec.py 'SOUPEDECODE.LOCAL/FileServer$@IP_VICTIME' -hashes aad3b435b51404eeaad3b435b51404ee:e41da7e79a4c76dbd9cf79d1cb325559`

<img width="1810" height="419" alt="image" src="https://github.com/user-attachments/assets/197ce51d-2741-44da-bfa6-0750697403b3" />
