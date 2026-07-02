<div align="center">
<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/618b3fa52f0acc0061fb0172-1753465052754" height="400"></img>
</div>

# Lien & Description
[https://tryhackme.com/room/easyctf](https://tryhackme.com/room/soupedecode01)

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

## **Seconde question** : *What is the root flag?* 
