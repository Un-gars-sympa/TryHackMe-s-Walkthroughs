<div align="center">
<img src="https://tryhackme-images.s3.amazonaws.com/room-icons/f28ade2b51eb7aeeac91002d41f29c47.png" height="500"></img>
</div>

# Link & Description
https://tryhackme.com/room/easyctf

*Beginner level ctf*

# Walkthrough :

First question : *How many services are running under port 1000?*

So let's begging with a basic Nmap scan :

`Nmap VICTIME_IP -sC -SV -p 0-1000`

And we have this result :

