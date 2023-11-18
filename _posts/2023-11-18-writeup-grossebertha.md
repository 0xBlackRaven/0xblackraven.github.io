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
