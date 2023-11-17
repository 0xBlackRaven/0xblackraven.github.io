---
layout: post
title: RF Surveillance
subtitle: Write-Up pour le NBCTF
tags: [writeup, misc, nbctf, rf]
comments: true
---
Bonjour ! Voici mon write-up pour le challenge `RF Surveillance` à l'occasion du NoBracket CTF. Il s'agit de mon deuxième write-up, alors n'hésitez pas à aller jeter un coup d'oeil au premier !
Tout d'abord, je tenais à remercier `Endeavxor` pour son challenge qui m'a particulièrement plu. Mais sur ce, passons au write-up ! 

## Premier coup d'oeil
![Chall](https://cdn.discordapp.com/attachments/822188888297963560/1175095822724108419/Capture_decran_28.png?ex=6569fc44&is=65578744&hm=495b544beb01cdd37c9cb371abca46257800538154b9117ad3df6f1d922e37d1&){: .mx-auto.d-block :}
Ce challenge est accompagné de deux ressources. La configuration de l'émetteur sur GNU Radio, puis la configuration du récepteur.
Regardons tout d'abord le flowgraph de l'émetteur :

![emetteur](https://cdn.discordapp.com/attachments/822188888297963560/1175096364762419210/EMISSION_GNU_Radio.png?ex=6569fcc5&is=655787c5&hm=84cf46f5e25c3b6fab9ece1495dfd034c963865cde6b30cbb40824902a12b530&){: .mx-auto.d-block :}

Nous avons pas mal de choses intéressantes ici. Nous voyons qu'un fichier intitulé `message_to_transmit.wav` est passé à travers un bloc `NBFM Transmit`. En gros, ce bloc produit une sortie en bande de base complexe modulée en FM. Encore un peu trop barbare. Pour faire simple, ça ![module](https://fr.wikipedia.org/wiki/Modulation_du_signal) juste le signal en FM. Les paramètres ne nous intéressent pas trop ici.
Le bloc `Rational Resampler` ne nous intéresse pas non plus. Mais par contre, le `PlutoSDR Sink` nous intéresse fortement. Nous allons ici nous intéresser surtout au Sample Rate, qui nous servira plus tard.
Donc résumons : nous avons un signal modulé en FM, dont la fréquence d'échantillonage est égale à `132300 Hz`. Nous pourrions d’ores et déjà regarder sur Audacity, mais regardons le flowgraph du receveur, pour ne pas louper d'informations qui pourraient être cruciales.

![receveur](https://cdn.discordapp.com/attachments/822188888297963560/1175096365303468092/RECEPTION_GNU_Radio.png?ex=6569fcc5&is=655787c5&hm=5e3dabd33f11240002894450c68d9932d231978437f8f847087e76562bed5c26&){: .mx-auto.d-block :}

Bon, toutes les informations essentielles ici ont déjà été identifiées avant, donc passons à la suite.

Allons maintenant sur Audacity ! Mais pourquoi Audacity ? Tout simplement parce que :
1. Je préfère le nom.
2. C'est plus simple et plus compréhensible (même pour vous, ce sera beaucoup plus plaisant car je ne devrais pas m'attarder à expliquer des zinzineries mathématiques que moi-même je ne comprends pas).


![meme](https://cdn.discordapp.com/attachments/822188888297963560/1175101705495523328/86e0w3.gif?ex=656a01be&is=65578cbe&hm=861e8d44c9d92a70e7c2955d17672f32632ad909a1379d37b1b9553c1aaeda42&){: .mx-auto.d-block :}

## Retrouver un signal compréhensible
On lance donc Audacity, puis on va dans `Fichier -> Ouvrir...` puis on choisit notre `signal_saved`. Et puis là, erreur !

{: .box-error}
Ça ne marche pas... Eh bien, c'est parce que notre fichier est un fichier signal brut ! C'est par ailleurs ce que nous dit Audacity, qui pour le coup ne peut pas être plus clair sur ce qu'il faut faire.

On va donc dans `Fichier -> Importer -> Données brutes`, puis on sélectionne notre fichier. On clique ensuite sur `Détecter`, ce qui nous évite de faire des réglages sur l'encodage. Dernier réglage : on change la fréquence d'échantillonage à `132300 Hz`. Et on est bon !
On importe... Et on run et on écoute. Il me semble que j'ai déjà entendu ça... Eh oui, nous avons du morse ! On va donc devoir décoder ça.
Je regarde tout d'abord comment je peux faire pour solve le challenge en python. Et c'est honnêtement l'enfer. J'essaie de nombreuses librairies, sans résultat convaincant. Je me résous donc à utiliser des outils en ligne, car le temps presse. Les équipes nous talonnent, et il n'y a pas de temps à perdre dans un script qui de toute façon ne marchera sans doute pas, ou tout du moins qui fonctionnera après la guerre (Vous avez le jeu de mots ? Parce que le thème de ce NBCTF est la Première Guerre mondiale... Enfin bref).
On essaie donc de regarder via différents sites de décodage de morse parce qu'il est hors de question que je décode tout à la main. Vous savez, une greffe de mains, ça coûte cher et c'est pas pratique, c'est donc pour ça que je privilégie la solution des décodeurs présents sur Internet.
![meme2](https://cdn.discordapp.com/attachments/822188888297963560/1175113834877427732/86e8wf.jpg?ex=656a0d0a&is=6557980a&hm=4334dea6946625cbd8b221d7b4d20923af214fd8de14497f47c54461fb48b3d2&){: .mx-auto.d-block :}

On exporte par la suite notre fichier pour le rendre dans un format lisible par les décodeurs en ligne. On va donc dans `Fichier -> Exporter l'audio...`, et nous voilà avec un `signal_saved.wav` tout neuf.

## Flag
Nous allons sur notre ![site](https://morsecode.world/international/decoder/audio-decoder-adaptive.html), nous rentrons notre .wav précédemment exporté... Et puis ça ne veut rien dire. C'est du charabia incompréhensible. Nous avons donc une mauvaise méthode... Mais comment faire pour résoudre ça ? Qu'est-ce qui ne va pas ? 
Je constate que quand le fichier est lu, la valeur des fréquences oscillent sans discontinuer... C'est peut-être de là que vient le problème ! On va donc essayer de trouver cette fréquence.


![meme3](https://i.imgflip.com/86eadb.jpg){: .mx-auto.d-block :}

On sélectionne donc un passage où un point ou un tiret peut être aperçu, c'est-à-dire un passage où l'information est transmise.
![image](https://cdn.discordapp.com/attachments/822188888297963560/1175114607413694494/Capture_decran_30.png?ex=656a0dc2&is=655798c2&hm=8e5653aedd1e493a6a15c1a501d774e2178f4192a8687ac972098987da2711f6&){: .mx-auto.d-block :}

Puis, après ça, nous allons dans `Analyse -> Tracer le spectre` ce qui nous donne ce beau graphique ci-contre.

![image](https://cdn.discordapp.com/attachments/822188888297963560/1175114608177053778/Capture_decran_31.png?ex=656a0dc3&is=655798c3&hm=dceaa6aa3dbe813ed2ee2b197ce6a3c43a656415a7e91fb12bbeaf5bc079f86d&){: .mx-auto.d-block :}

On peut ainsi voir la fréquence haute ! Qui est ici égale à `2163 Hz` !

On rentre donc notre nouvelle fréquence... Puis on laisse tourner l'audio. Le début semble déjà plus convaincant. Mais... C'est de l'allemand ! Parfait, étant mosellan, je vais enfin pouvoir exécuter l'ordre de mission ! Hum... Excusez-moi, je m'égare.
![flag](https://cdn.discordapp.com/attachments/822188888297963560/1175129394096906341/Capture_decran_33.png?ex=656a1b88&is=6557a688&hm=8069b0bdf5db0ed38f97953bfa20008208d757e4c53a369e57990ad5e9294e28&){: .mx-auto.d-block :}

Parfait, nous avons notre flag ! Un petit coup d'oeil au format demandé nous renseigne sur ce qu'est le flag : `NBCTF{VOUSAVEZDECODERAVECSUCCESGG}` !

## Conclusion

J'ai particulièrement aimé ce challenge, mêlant radiofréquences et histoire. J'ai appris de nombreuses choses sur les radiofréquences, notamment la modulation, sujet qui m'intéresse tout particulièrement. Je remercie tout particulièrement le créateur de ce challenge, `Endeavxor`. J'ai fortement aimé le faire. J'ai hâte de voir la finale pour avoir de nouveaux challenges de cette qualité ! Bon allez, je retourne envoyer des messages en allemand.
