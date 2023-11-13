---
layout: post
title: Bureau Déverrouillé
subtitle: Write-Up pour le NBCTF
tags: [writeup, reseau, prog]
comments: true
---
Bonjour ! Voici mon write-up sur le challenge **Bureau Déverrouillé** à l'occasion du NoBracket CTF. J'ai eu l'opportunité de terminer 2ème avec l'équipe nndy. Tout d'abord, un grand merci au créateur de 
ce challenge, ribt, ainsi qu'à tous les organisateurs et créateurs de challenges, que j'ai pu embêter à de nombreuses reprises à l'occasion de ce CTF. Mais sur ce, passons au challenge !

## Premier coup d'oeil
![Chall](https://github.com/0xBlackRaven/0xBlackRaven.github.io/blob/master/assets/img/challenge_description.png?raw=true){: .mx-auto.d-block :}

La description du challenge ne nous renseigne pas plus que ça sur ce qui aurait pu se passer, mais on peut supposer que des données ont été exfiltrées. Et que tout ça a été enregistré par une capture réseau.
Ca tombe bien, on est justement dans la catégorie réseau !
On télécharge le fichier fourni, sobrement intitulé **capture.pcapng**, puis on le convertit en .pcap, que Wireshark peut lire, avec la commande suivante :
```console
blackraven@blackraven:~$ tshark -F pcap -r capture.pcapng -w capture.pcap
```
Et hop, on a notre fichier tout propre tout beau, que Wireshark peut lire.
