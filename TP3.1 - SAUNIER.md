# TP3 - Gestion des paquets

**SAUNIER Arthur**

## Exercice 1. Commandes de base

Commencez par mettre à jour votre système avec les commandes vues dans le cours. Donnez les commandes répondant aux questions suivantes : 
1. Quels sont les 5 derniers paquets installés sur votre machine ? 
commande: 
**grep "installed" /var/log/dpkg.log**

```
2020-11-02 08:11:37 status installed libx11-data:all 2:1.6.9-2ubuntu1.1
2020-11-02 08:11:37 status installed man-db:amd64 2.9.1-1
2020-11-02 08:11:44 status installed python3-problem-report:all 2.20.11-0ubuntu27.6
2020-11-02 17:33:29 status installed mlocate:amd64 0.26-3ubuntu3
2020-11-02 17:33:30 status installed man-db:amd64 2.9.1-1
```


3. Utiliser dpkg et apt pour compter le nombre de paquets installés (ne pas hésiter à consulter le manuel !). Comment explique-t-on la (petite) différence de comptage ? 

Les commandes:
**apt list --installed | wc -l**
**dpkg -l | wc -l**

On a une différence de 4 entre dpkg et apt
Cette différence est du a quelque ligne de texte affiché en plus au début de la sorite des commandes, si on oublie ces lignes, on a le même nombre de paquets.

**dpkg -l | grep ^ii | wc -l**
le **^** permet de choisir uniquement les lignes qui **commencent** par ii.

**apt list --installed 2> /dev/null | grep installed | wc -l**
pour ne pas prendre en compte les erreurs: **2> /dev/null** (TP1)


4. Combien de paquets sont disponibles en téléchargement ? 
commande: **apt list | wc -l**

On a 61531 paquets disponible en téléchargement

4. Créer un alias “maj” qui met à jour le système 

Dans .bashrc: 
**alias maj='sudo apt full-upgrade'**
ne pas oublier de faire:
**source .bashrc**

5. A quoi sert le paquet fortunes ? Installez-le. 

Fortune affiche des citations, des proverbes a chaque connexion sur le terminal.

**sudo apt install fortune**

6. Quels paquets proposent de jouer au sudoku ? 
commande:
**apt search sudoku**

résultat:
```
fltk1.1-games/focal 1.1.10-26ubuntu2 amd64
  Fast Light Toolkit - example games: checkers, sudoku

fltk1.3-games/focal 1.3.4-10build1 amd64
  Fast Light Toolkit - example games: checkers, sudoku

gnome-sudoku/focal 1:3.36.0-1 amd64
  Sudoku puzzle game for GNOME

hitori/focal 3.34.0-1 amd64
  logic puzzle game similar to sudoku

ksudoku/focal 4:19.12.3-0ubuntu1 amd64
  Sudoku puzzle game and solver

libqqwing-dev/focal 1.3.4-1.1build1 amd64
  tool for generating and solving Sudoku puzzles (development)

libqqwing2v5/focal 1.3.4-1.1build1 amd64
  tool for generating and solving Sudoku puzzles (library)

nudoku/focal 1.0.0-1 amd64
  ncurses based sudoku games

qqwing/focal 1.3.4-1.1build1 amd64
  tool for generating and solving Sudoku puzzles (application)

sudoku/focal 1.0.5-2build3 amd64
  console based sudoku

texlive-games/focal 2019.202000218-1 all
  TeX Live: Games typesetting


```

7. Lister les derniers paquets installés explicitement avec la commande apt install



## Exercice 2. 

A partir de quel paquet est installée la commande ls ? Comment obtenir cette information en une seule commande, pour n’importe quel programme (indice : la réponse est dans le poly de cours 2, dans la liste des commandes utiles) ? Utilisez la réponse à pour écrire un script appelé origine-commande (sans l’extension .sh) prenant en argument le nom d’une commande, et indiquant quel paquet l’a installée.

Commande: 
**which -a ls**
```
/usr/bin/ls
/bin/ls
```
**-a** pour all permet de donner tout les chemin possible

**dpkg -S /bin/ls**
```
coreutils: /bin/ls
```
ls est avec le paquet coreutils.

en une seule commande:
**which -a ls | xargs dpkg >2/dev/null **

xargs car dpkg ne prend que des arguments.
Pour le script:
```
#!/bin/bash
dpkg -S $(which -a $1) 2>/dev/null
```

## Exercice 3. 
Ecrire une commande qui affiche “INSTALLÉ” ou “NON INSTALLÉ” selon le nom et le statut du package spécifié dans cette commande. 
voici la commande:
``
dpkg -l "nom package" >2/dev/null| grep "^ii"> /dev/null && echo "INSTALLE" || echo "NON INSTALLE"
``

## Exercice 4. 
Lister les programmes livrés avec coreutils. 
A quoi sert la commande ’[’ et comment afficher ce qu’elle retourne ?

pour lister les programmes: 
**dpkg -L coreutils**

la commande 
 
## Exercice 5. aptitude 
Installez le paquet emacs à l’aide de la version graphique d’aptitude. 

sudo aptitude pour lancer aptitude.
/emacs pour faire une recherche.
"+" pour ajouter a la liste
"g" pour aller sur la liste et de nouveau g pour installer les package.

## Exercice 6. Installation d’un paquet par PPA 
Certains logiciels ne figurent pas dans les dépôts officiels. C’est le cas par exemple de la version ”officielle” de Java depuis qu’elle est développée par Oracle. Dans ces cas, on peut parfois se tourner vers un ”dépôt personnel” ou PPA. 
1. Installer la version Oracle de Java (avec l’ajout des PPA) 
```
sudo add-apt-repository ppa:linuxuprising/java 
sudo apt update 
sudo apt install oracle-java11-installer 
```
2. Vérifiez qu’un nouveau fichier a été créé dans /etc/apt/sources.list.d. Que contient-il ?

 il y a bien un nouveau fichier: 
 **linuxuprising-ubuntu-java-focal.list**
 et il contient l'adresse:
http://ppa.launchpad.net/linuxuprising/java/ubuntu 
qui est le lien du package sur ppa.

## Exercice 7. Création de dépôt personnalisé 

Dans cet exercice, vous allez créer vos propres paquets et dépôts, ce qui vous permettra de gérer les programmes que vous écrivez comme s’ils provenaient de dépôts officiels. 

### Création d’un paquet Debian avec dpkg-deb 1.

1. Dans le dossier **scripts** créé lors du TP 2, créez un sous-dossier **origine-commande** où vous créerez un sous-dossier **DEBIAN**, ainsi que l’arborescence **usr/local/bin** où vous placerez le script écrit à l’exercice 2 

Attention a bien suivre ce positionnement de répertoire/fichier pour ne pas avoir d'erreur

2. Dans le dossier DEBIAN, créez un fichier control avec les champs suivants : 
```
Package: origine-commande #nom dup paquet 
Version: 0.1 #numéro de version 
Maintainer: Foo Bar #votre nom 
Architecture: all #les architectures cibles de notre paquet (i386, amd64...) 
Description: Cherche l'origine d'une commande 
Section: utils #notre programme est un utilitaire 
Priority: optional #ce n'est pas un paquet indispendable 
```

Enlever les commentaire (#blabla) sinon ça ne fonctionne pas.

3. Revenez dans le dossier parent de origine-commande (normalement, c’est votre $HOME) et tapez la commande suivante pour construire le paquet :
```
 dpkg-deb --build origine-commandecd
```
si message d'erreur, vérifier que bien ds le bon répertoire.

 Félicitations ! Vous avez créé votre propre paquet !  
 
 ### Création du dépôt personnel avec reprepro 
 
 1. Dans votre dossier personnel, commencez par créer un dossier repo-cpe. Ce sera la racine de votre dépôt 

>mkdir repo-cpe

 2.  Ajoutez-y deux sous-dossiers : conf (qui contiendra la configuration du dépôt) et packages (qui contiendra nos paquets) 

>15:22 - arthur@server:~$ cd repo-cpe/
>15:22 - arthur@server:~/repo-cpe$ mkdir conf
>15:22 - arthur@server:~/repo-cpe$ mkdir packages


 3. Dans conf, créez le fichier distributions suivant : 
```
Origin: Un nom, une URL, ou tout texte expliquant la provenance du dépôt 
Label: Nom du dépôt 
Suite: stable 
Codename: cosmic #le nom de la distribution cible 
Architectures: i386 amd64 #(architectures cibles) 
Components: universe #(correspond à notre cas) 
Description: Une description du dépôt 
```
Voici ce que ça donne ds le fichier distributions:
```
Origin: ExoCPE
Label: Origine_Com
Suite: stable
Codename: focal
Architectures: i386 amd64
Components: universe
Description: Une description du dépôt

```

4. Dans le dossier repo-cpe, générez l’arborescence du dépôt avec la commande 
``
reprepro -b . export 
``

5. Copiez le paquet **origine-commande.deb** créé précédemment dans le dossier packages du dépôt, puis, à la racine du dépôt, exécutez la commande 
``
reprepro -b . includedeb cosmic origine-commande.deb 
``
afin que votre paquet soit inscrit dans le dépôt. 

6. Il faut à présent indiquer à apt qu’il existe un nouveau dépôt dans lequel il peut trouver des logiciels. Pour cela, créez **(avec sudo)** dans le dossier **/etc/apt/sources.list.d** le fichier **repo-cpe.list** contenant : *
``
deb file:/home/VOTRE_NOM/repo-cpe cosmic multiverse 
``
(cette ligne reprend la configuration du dépôt, elle est à adapter au besoin) 
7. Lancez la commande **sudo apt update**. 
Féliciations ! Votre dépôt est désormais pris en compte ! ... Enfin, pas tout à fait... Si vous regardez la sortie **d’apt update**, il est précidé que le dépôt ne peut être pris en compte car il n’est pas signé. La signature permet de vérifier qu’un paquet provient bien du bon dépôt. On doit donc signer notre dépôt. 

### Signature du dépôt avec GPG 

GPG est la version GNU du protocole PGP (Pretty Good Privacy), qui permet d’échanger des données de manière sécurisée. Ce système repose sur la notion de clés de chiffrement asymétriques (une clé publique et une clé privée) 
1. Commencez par créer une nouvelle paire de clés avec la commande 
``
gpg --gen-key 
``
Attention ! N’oubliez pas votre passphrase !!! 

2. Ajoutez à la configuration du dépôt (fichier distributions la ligne suivante : 
``
SignWith: yes 
``
3. Ajoutez la clé à votre dépôt : 
``
reprepro --ask-passphrase -b . export 
``
Attention ! Cette méthode n’est pas sécurisée et obsolète ; dans un contexte professionnel, on utiliserait plutot un **gpg-agent**. 

4. Ajoutez votre clé publique à votre dépôt avec la commande
``
 gpg --export -a "auteur" > public.key 
 ``
 
5. Enfin, ajoutez cette clé à la liste des clés fiables connues de apt : 
``
sudo apt-key add public.key 
``

Félicitations ! La configuration est (enfin) terminée ! Vérifiez que vous pouvez installer votre paquet comme n’importe quel autre paquet.




> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyODYwMjI2MzMsNzY5NjExNzkwXX0=
-->