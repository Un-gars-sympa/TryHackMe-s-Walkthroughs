<div align="center">
<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/618b3fa52f0acc0061fb0172-1753465052754" height="400"></img>
</div>

# Link & Description

https://tryhackme.com/room/soupedecode01

*Test your enumeration skills on this boot-to-root machine.*

This CTF was my first one on an Active Directory environment. It was very interesting, as it made me want to explore AD network vulnerabilities further. Moreover, the [mind map proposed by Orange Cyberdefense](https://orange-cyberdefense.github.io/ocd-mindmaps/img/mindmap_ad_dark_classic_2025.03.excalidraw.svg) about AD pentesting was very useful to me during this CTF.

# Walkthrough:

## **First question**: *What is the user flag?*

Well, let's start every pentest with some reconnaissance. As always, I start by running an Nmap scan against the target machine: `nmap -A -p- VICTIM_IP`

<img width="895" height="1020" alt="Screenshot 2026-07-02 154525" src="https://github.com/user-attachments/assets/c8d869c1-4103-412f-a404-c4ae5f81d862" />

From this scan, we can see the domain name: *SOUPEDECODE.LOCAL*, as well as many open ports.

I add the domain name to my `/etc/hosts` file.

Once that's done, I continue with SMB share enumeration. I use smbclient with the guest account: `smbclient -L soupedecode.local -U 'guest'`.

<img width="543" height="229" alt="Screenshot 2026-07-02 155235" src="https://github.com/user-attachments/assets/e6f37a03-b61c-4269-97d2-900a9fa355ec" />

I notice that a "backup" share exists, which makes me want to take a look inside it. `smbclient //soupedecode.local/backup -U guest`, but we are quickly stopped because we don't have access to it.

From this point on, let's look for a user account, starting with a username. To do this, I use NetExec, aka nxc, which is kind of a Swiss Army knife for hacking. I will enumerate the users' RIDs: `nxc smb soupedecode.local -u 'guest' -p '' --rid`

And there we go, we have quite a long list of users.

<img width="1074" height="1034" alt="Screenshot 2026-07-03 104532" src="https://github.com/user-attachments/assets/c76136b2-ca14-4081-906a-a8cb8861ed9a" />

I copy the results into a `user.txt` file and then run this command: `awk -F'\\' '{print $2}' user.txt | awk '{print $1}'` to get a clean list.

Alright, if we go back to the OCD roadmap, we have two possibilities: either Password Spraying or AS-REP Roasting. I would be more inclined to start with the second option.

I won't go into too much detail about what AS-REP Roasting is, but basically, if an account has the **"Do not require Kerberos preauthentication"** option enabled, it can request a TGT from the KDC without authenticating. This TGT is partially encrypted using the account's password, meaning it can potentially be cracked if the password is weak.

So, we need to check whether one of the accounts in our big list has this option enabled.

For this, I use `GetNPUsers.py` from Impacket, which allows us to request TGTs (AS-REQs) and hopefully retrieve one for an account: `GetNPUsers.py soupedecode.local/ -usersfile user.txt -format hashcat`

It looks like none of the users have the option enabled 🤡

In that case, let's try Password Spraying. The first simple option that comes to mind is testing whether `username == password`.

For this, I use nxc again with the `--no-bruteforce` and `--continue-on-success` options: `nxc smb soupedecode.local -u user.txt -p user.txt --no-bruteforce --continue-on-success`

And bingo!

<img width="576" height="576" alt="6eaaaa0853971f48a4890cf72d4a86e7 576x576x1" src="https://github.com/user-attachments/assets/c151a9fe-45f2-4785-ac9e-88d474e2f2ca" />

<img width="1137" height="65" alt="Screenshot 2026-07-03 112947" src="https://github.com/user-attachments/assets/7683903d-ae5f-4553-b0d3-04ae8e3fd478" />

Someone is definitely going to get in trouble for this. Anyway, `username == password` => Not good 🫢

Let's go back to SMB enumeration to see whether our friend `ybob317` has access to the `backup` SMB share: `smbclient //soupedecode.local/backup -U 'ybob317'`, and also check whether we can connect directly via RDP: `xfreerdp /d:soupedecode.local /v:soupedecode.local /u:ybob317 /p:ybob317`.

<img width="1551" height="733" alt="Screenshot 2026-07-03 115824" src="https://github.com/user-attachments/assets/57c9d993-1999-4c03-97a5-58820bbdf31c" />

<img width="360" height="270" alt="1514241468-cestnon" src="https://github.com/user-attachments/assets/738928a9-612e-4168-8e81-3e3015f5c536" />

Well, after this disappointment, the logical next step would be to continue enumerating SMB and investigate the `backup` share, but we still don't have access to it.

In that case, let's go into the `Users` share and then into our good old friend `ybob317`'s directory. There, we find our first flag!

<img width="1359" height="1081" alt="image" src="https://github.com/user-attachments/assets/f5adabbd-fcc2-41a0-b812-734b95dff53e" />

## **Second question**: *What is the root flag?*

Let's gooooo!

Alright, several possibilities are available to us, starting with a potential Kerberoasting attack.

To briefly explain what Kerberoasting is: every service that needs to be accessible has a service account in AD (e.g., `svc_iis`, `svc_sql`, `svc_scanner`, etc.). The goal of Kerberoasting is to request a TGS for one of these services. Just like with AS-REP Roasting, the ticket is partially encrypted using the account's password, meaning it can potentially be cracked if the password is weak.

So, for this, we are going to use another Impacket tool: `GetUserSPNs.py`, with the appropriate command: `GetUserSPNs.py -request -dc-ip soupedecode.local soupedecode.local/ybob317:ybob317`

<img width="2159" height="1218" alt="image" src="https://github.com/user-attachments/assets/180cfe0f-9507-4cb3-bb9c-d648d2c7911e" />

So, five accounts provided us with their TGS. I put the hashes into a `hash.txt` file, and the next step is to crack them using Hashcat: `hashcat -m 13100 hash.txt /usr/share/wordlist/rockyou.txt`

Tada!

<img width="1246" height="993" alt="image" src="https://github.com/user-attachments/assets/fcbe18ae-4827-4175-b8da-ca0bf28c0602" />

Let's test the `backup` SMB share for the third time 💀: `smbclient //soupedecode.local/backup -U file_svc`

But what's that cook?? 🐔

<img width="1013" height="422" alt="image" src="https://github.com/user-attachments/assets/5e3907ff-cce4-4fde-8df0-790165140290" />

Well, we now have access to a bunch of hashes. I check which account could potentially be vulnerable to a Pass-the-Hash attack.

Before that, I'll simply put the usernames into a new file and the hashes into another one, just to figure out which account I could use.

<img width="1551" height="637" alt="image" src="https://github.com/user-attachments/assets/ecb374a8-b502-457d-9582-995d99155480" />

Perfect, all that's left is to use PsExec to connect: `psexec.py 'SOUPEDECODE.LOCAL/FileServer$@VICTIM_IP' -hashes aad3b435b51404eeaad3b435b51404ee:e41da7e79a4c76dbd9cf79d1cb325559`

<img width="1810" height="419" alt="image" src="https://github.com/user-attachments/assets/197ce51d-2741-44da-bfa6-0750697403b3" />
