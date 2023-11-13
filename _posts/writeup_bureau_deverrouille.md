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
On télécharge le fichier fourni, sobrement intitulé **capture.pcapng**, et on garde en pcapng parce que c'est next-gen et que notre wireshark peut le lire sans problème.

![Wireshark_le_goat](https://cdn.discordapp.com/attachments/822188888297963560/1173656631586865243/85vnqb.jpg?ex=6564bfea&is=65524aea&hm=216a9557f15a87dff50432c6b79ee7a1267ef92c73ecd29dc77af0bb14fa0479&){: .mx-auto.d-block :}

Et hop, on a notre fichier tout propre tout beau, que Wireshark peut lire.
On ouvre donc le fichier, et puis là... On est assez surpris par la longueur du fichier.
La capture fait 7491 trames, principalement des trames DNS. Une première intuition me fait dire que la plus grosse partie du challenge se déroulera là. Ensuite, nous avons au début quelque chose que j'interprète comme l'initialisation de quelque chose. Et là, détail me saute aux yeux : un GET de /malware.py, provenant du site ribt.fr sur le port 8000.

 ![Pas_discret](https://cdn.discordapp.com/attachments/822188888297963560/1173656521385705573/Capture_decran_24.png?ex=6564bfd0&is=65524ad0&hm=45274245d1a8992fc45430afab53f765d6d829bd0151cd1ae27d600e1b3b9a4b&){: .mx-auto.d-block :}
