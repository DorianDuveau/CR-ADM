# TP 1 - Installation d’Ubuntu Server et prise en main du shell

**SAUNIER Arthur**

# Exercice 1
Clonage VM
**Ne pas oublier le fichier ISO server.**
**Pas de souci pour le démarrage, attention peut avoir besoin d'entrer dans le bios pour autoriser la virtualisation et faire fonctionner VM.**

>ssh arthur@localhost -p 2222 se connecter en ssh sur le server depuis mobaxterm.

Chemin absolu, je pars de la racine (commence par un /)
# Exercice 2

## Manuel

 1. A l’aide du manuel, identifiez le rôle de la commande which
utiliser la commande **"man which"** pour obtenir les infos sur la commande which.
>man permet d'avoir les infos du manuel
- which filename 
retourne le path du fichier spécifier qui serait exécuter dans l'environnement actuel

 2. Quand on consulte une page du manuel, comment peut-on rechercher un terme (par exemple, chercher le terme option dans la page de manuel de which ?
>On peut utiliser la commande **"/name"**
ici, pour option, on marque **/option**

 3. Comment quitte-t-on le manuel ?
>en appuyant sur q

 4. Chaque section du manuel a une première page, qui présente le contenu de la section. Afficher la première page de la section 6 ; de quoi parle cette section ?
>la commande pour ouvrir la section est: **"man 6 intro"**
La section 6 décrit les jeux et petits programme dispo sur le système

## Navigation dans l’arborescence des fichiers

 1. allez dans le dossier /var/log
>commande: **cd /var/** puis **cd log**
 2. remontez dans le dossier parent (/var) en utilisant un chemin relatif
>commande **"cd .."**
 3. retournez dans le dossier personnel
>commande **"cd"**
 4. revenez au dossier précédent (/var) sans utiliser de chemin
>commande **"cd -"**
 5. essayez d’accéder au dossier /root ; que se passe-t-il ?
commande **"cd /root"**
>la commande n'est pas autorisée
 6. essayez la commande sudo cd /root ; que se passe-t-il ? Expliquez
>commande **"sudo cd /root"**
La commande  me demande mon mdp. Par contre la commande **"sudo: cd:"** n'est pas connu dc pas exécutée
sudo ne prend en argument que des commandes, cd est interne a bash, donc via le super utilisateur, on ne peut pas utiliser sudo car il n'existe pas pour lui.

 7. à partir de votre dossier personnel, créez l’arborescence suivante :
>pour créer un dossier: **"mkdir"**
pour supprimer un dossier: **"rmdir"**
pour créer un fichier: **"> file1.txt"** l'extension du fichier peut être absente. 

 8. . revenez dans votre dossier personnel ; à l’aide de la commande rm, essayez de supprimer Fichier1, puis Dossier1 ; que se passe-t-il ?
>Fichier1 n'est pas dans ce directory, on ne peut pas le modifier depuis le dossier perso, on doit entrer dans le dossier où il est stocké pour le supprimer.
**rm** permet de supprimer des fichiers, pas des dossier c'est pour ça que ça ne fonctionne pas sur dossier1

 9. quelle commande permet de supprimer un dossier ?
> **"rmdir"** 
 10. que se passe-t-il quand on applique cette commande à Dossier2 ? 
>Ne fonctionne pas car dossier2 n'est pas vide
 11. comment supprimer en une seule commande Dossier2 et son contenu ?
> utiliser la commande rm récursive: **"rm -r"**

 ## Commandes importantes

 1. Quelle commande permet d’afficher l’heure ? A quoi sert la commande time ?
>**"date"** permet d'afficher la date et l'heure
**"time cmd"** permet de savoir combien de temps une cmd spécifique met-elle pour s'exécuter

 2. Dans votre dossier personnel, tapez successivement les commandes ls puis la ; que peut-on en déduire sur les fichiers commençant par un point ?
>les fichier commençant pas un point sont masqué

 3. Où se situe le programme ls ?
```
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

 4. . Essayez la commande ll. Existe-t-il une entrée de manuel pour cette commande ? Utilisez les commandes alias ou alias pour en savoir plus sur la nature de cette commande.
>**"ll"** permet de lister les fichiers avec leurs auteurs, dates de création, taille ainsi que les droit d'accès en ecriture, lecture, etc desdits fichiers
il n'y a pas d'entrée du manuel pour ll car c'est un alias de ls, **"man ls"** permet d'obtenir des infos.

 5. Quelle commande permet d’afficher les fichiers contenus dans le dossier /bin ?
>**"ls"** fonctionne pour afficher les fichier sous /bin

 6. Que fait la commande ls .. ? 
>**"ls .."** liste les dossier parents du dossier actuels

 7. Quelle commande donne le chemin complet du dossier courant ? 
>**pwd**
 8. Que fait la commande **"echo 'yo' > plop"** exécutée 2 fois ? 
>on écrit yo ds le fichier plop.
si on le fait 2 fois, on ecrase le fichier plop a chaque fois

 9.  Que fait la commande echo 'yo' >> plop exécutée 2 fois ? 
> on écrit yo ds le fichier plop 2 fois, **>>** permet d'écrire a la fin d'un fichier sans écraser celui ci. Si le fichier n'existe pas, il est créer.

 10. Interprétez le comportement de la commande **sleep 10 | echo 'toto' **? 
>La commande sleep ne produit rien, du coup on a tout de suite toto qui est affiché et la pause de 10 seconde qui ont lieu.

11. A quoi sert la commande file ? Essayez la sur des fichiers de types différents. 

>**"file filename"** permet de donner le type de fichier, si on les test sur plop par exemple, on nous donne que plop est de type **ASCII text**

12. Créez un fichier toto qui contient la chaîne **"Hello Toto !"** ; créer ensuite un lien **titi** vers ce fichier avec la commande **"ln toto titi"**. Modifiez à présent le contenu de toto et affichez le contenu de titi : qu’observe-t-on ? Supprimez le fichier toto ; quelle conséquence cela a-t-il sur titi ? 
>commande: **"echo 'hello toto!' > toto"**
ensuite: **"ln toto titi"**
pour afficher le contenu de titi: **cat** ou **less** ou **more**
**cat** affiche tout le fichier d'un coup, **less** le fait page par page et **more** aussi avec des personnalisation différentes.
on supprime toto, si on  regarde de nouveau titi, on a encore les données qui sont présente. Titi a donc copier le contenu de toto.

13. Créez à présent un lien symbolique tutu sur titi avec la commande **ln -s titi tutu**. Modifiez le contenu de titi ; quelle conséquence pour tutu ? Et inversement ? Supprimez le fichier titi ; quelle conséquence cela a-t-il sur tutu ? 
>commande: **"ln -s titi tutu"**
si on modifie titi, tutu  est modifié, si on modifie tutu, titi est modifié. Par contre, si titi est supprimé, on perd les données même avec tutu.

14. Affichez à l’écran le fichier **/var/log/syslog**. Quels raccourcis clavier permettent d’interrompre et reprendre le défilement à l’écran ? 
>commande pour afficher syslog: **"more syslog"**
pour défiler page par page: **spacebar**
pour revenir en arrière page par page: **b**
affiche les 11 lignes suivantes: **d**
ligne par ligne: **entrée**
pour quitter: **q**

15. Affichez les 5 premières lignes du fichier /var/log/syslog, puis les 15 dernières, puis seulement les lignes 10 à 20. 
>pour les 5 première ligne du fichier: **head 5 syslog**
>pour les 15 dernières ligne du fichier: **tail 15 syslog**

16. Que fait la commande **dmesg | less** ? 
>pipeline de demsg, qui affiche le kernel de linux.
>ici on utilise un pipeline pour afficher dmesg a l'aide de la meethode less donc page par page

17. Affichez à l’écran le fichier /etc/passwd ; que contient-il ? Quelle commande permet d’afficher la page de manuel de ce fichier ? 
>**etc/passwd** contient les infos de login, user name, localisation de dossier, etc...
**man passwd** pour afficher la page du manuel

18. Affichez seulement la première colonne triée par ordre alphabétique inverse 
>**sort -r** pour ordre alpha inverse
> **cat /etc/passwd** pour ouvrir le fichier
> **cut -d ":" -f1**, coupe la réponse jusqu'au délimiteur *":"* et prend la première instance *-f1*
>commande complète: 
```
cat /etc/passwd | sort -r | cut -d ":" -f1
```

19. Quelle commande nous donne le nombre d’utilisateurs ayant un compte sur cette machine (pas seulement les utilisateurs connectés) ? 
>commande: **cat /etc/passwd | wc -l**
>**wc** permet de compter quelque chose, **-l** permet de compter les lignes
>il y a 33 users


20. Combien de pages de manuel comportent le mot-clé conversion dans leur description ? 
>commande: utiliser **grep**

21. A l’aide de la commande find, recherchez tous les fichiers se nommant passwd présents sur la machine 


22. Modifiez la commande précédente pour que la liste des fichiers trouvés soit enregistrée dans le fichier ~/list_passwd_files.txt et que les erreurs soient redirigées vers le fichier spécial /dev/null 

23. Dans votre dossier personnel, utilisez la commande grep pour chercher où est défini l’alias ll vu précédemment 
>l'alias "ll" est spécidié ds le fichier .bashrc

24. Utilisez la commande locate pour trouver le fichier history.log. 
>commande: **locate history.log**
>réponse: /var/log/apt/history.log

25. Créer un fichier dans votre dossier personnel puis utilisez locate pour le trouver. Apparaît-il ? Pourquoi ?
>Il n'apparait pas, car nous somme déjà ds le directory où le fichier se trouve.


# exercice 3

 ## Familiarisation avec Nano

 1. Pour copier le fichier:
```
cp /var/log/syslog /home/arthur
```



# exercice 4

Pour ajouter l'heure en rose, il suffit d'ajouter ce champ:
>\[\033[1;95m\]\A\[033[00m\] -

dans la variable PS1, dans le fichier .bashrc

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3OTAyMzAwNjEsLTEwNjY1NDU5Nl19
-->