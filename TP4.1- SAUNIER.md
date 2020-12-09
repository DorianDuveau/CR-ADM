# TP 4 - Utilisateurs, groupes et permissions

**SAUNIER Arthur**

## Exercice 1 - Gestion des utilisateurs et des groupes

1. Utilisez la commande groupadd pour créer deux groupes dev et infra 

Commande:
>sudo groupadd dev
>sudo groupadd infra

. Créez ensuite 4 utilisateurs alice, bob, charlie, dave avec la commande useradd, en demandant la création de leur dossier personnel et avec bash pour shell 

Commande:
>sudo useradd -m "username"

. Ajoutez les utilisateurs dans les groupes créés : - alice, bob, dave dans dev - bob, charlie, dave dans infra 

Commande:
>sudo usermod username -g  groupe

Si une personne a déja un groupe, pour ajouter a un groupe secondaire:
>usermod -a -G groupe username


4. Donnez deux moyens d’afficher les membres de infra 

installer le package members avec
>sudo apt install members

ensuite:
>members infra
>members dev

Pour afficher membres primaire **-p**, membres secondaire **-s**

Autre solution:
>getent group infra/dev


5. Faites de dev le groupe propriétaire des répertoires /home/alice et /home/bob et de infra le groupe propriétaire de /home/charlie et /home/dave 

Commande:
>sudo chgrp dev /home/alice
>sudo chgrp dev /home/bob
> sudo chgrp infra /home/charlie
> sudo chgrp infra /home/dave

Pour vérifier que c'est bon:
>cd /home
>ls -l
```bash
09:31 - arthur@server:/home$ ls -l
total 20
drwxr-xr-x 2 alice   dev    4096 Nov 17 08:57 alice
drwxr-xr-x 6 arthur  arthur 4096 Nov 17 08:23 arthur
drwxr-xr-x 2 bob     dev    4096 Nov 17 08:57 bob
drwxr-xr-x 2 charlie infra  4096 Nov 17 08:57 charlie
drwxr-xr-x 2 dave    infra  4096 Nov 17 08:57 dave
```

6. Remplacez le groupe primaire des utilisateurs : - dev pour alice et bob - infra pour charlie et dave 

Commande:
>sudo usermod username -g groupe

7. Créez deux répertoires /home/dev et /home/infra pour le contenu commun aux membres de chaque groupe, et mettez en place les permissions leur permettant d’écrire dans ces dossiers. 

Pour créer les dossiers:
>09:42 - arthur@server:/home$ sudo mkdir dev
>09:42 - arthur@server:/home$ sudo mkdir infra

Pour changer les accès et le groupes:
>sudo chgrp dev /home/dev
>sudo chgrp infra /home/infra

Pour changer les droit:
>09:47 - arthur@server:/home$ sudo chmod g+w dev
>09:47 - arthur@server:/home$ sudo chmod g+w infra

8. Comment faire pour que, dans ces dossiers, seul le propriétaire d’un fichier ait le droit de renommer ou supprimer ce fichier ? 

On doit utiliser le sticky bit **-t** pour mettre en place ce fonctionnement.

9.  Pouvez-vous ouvrir une session en tant que alice ? Pourquoi ?  

Non ne peut pas ouvrir de session car tout utilisateur créer sans mdp est inactif.

10. Activez le compte de l’utilisateur alice et vérifiez que vous pouvez désormais vous connecter avec son compte. 

Pour activer le compte, création du mdp:
>sudo passwd alice

Ensuite on change d'utilisateurs pour voir:
>su alice

Pour ne plus être alice:
>exit

11. Comment obtenir l’uid et le gid de alice ? 

Commande:
>id alice

12. Quelle commande permet de retrouver l’utilisateur dont l’uid est 1003 ? 

Commande:
>id 1003

13. Quel est l’id du groupe dev ? 

 Commande:
 >getent group dev

Il a pour gid 1001

14. Quel groupe a pour gid 1002 ? ( Rien n’empêche d’avoir un groupe dont le nom serait 1002...) 

C'est le groupe infra, 

Commande:
>grep x:1002 /etc/group

15. Retirez l’utilisateur charlie du groupe infra. Que se passe-t-il ? Expliquez. 

La commande pour retirer charlie du groupe infra:
>sudo gpasswd -d charlie infra

Si on retire Charlie, il n'a plus de groupe (ce qui n'est pas possible) donc un groupe a son nom ds lequel il va être placé va être créé.

16. Modifiez le compte de dave de sorte que : 
```
— il expire au 1 er juin 2021 
— il faut changer de mot de passe avant 90 jours 
— il faut attendre 5 jours pour modifier un mot de passe 
— l’utilisateur est averti 14 jours avant l’expiration de son mot de passe 
— le compte sera bloqué 30 jours après expiration du mot de passe 
```

Pour afficher les infos sur le mdp de Dave:
```bash
10:22 - arthur@server:/home$ sudo chage -l dave
[sudo] password for arthur:
Last password change                                    : Nov 17, 2020
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```
Pour que la date d'expiration soit au 1er juin 2021
>sudo chage -E 2021-06-01 dave

Pour changer le mdp au bout de 90 jours:
>sudo chage -M 90 dave

Pour qu'il faille attendre 5 jours a chaque fois que le mdp est changé:
>sudo chage -m 5 dave

Pour avertir l'utilisateur 14 jours avant:
>sudo chage -W 14 dave

Pour bloquer le compte après 30 jours:
>sudo chage -I 30 dave


Le résultat:
```bash
Last password change                                    : Nov 17, 2020
Password expires                                        : Feb 15, 2021
Password inactive                                       : Mar 17, 2021
Account expires                                         : Jun 01, 2021
Minimum number of days between password change          : 5
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 14

```


17. Quel est l’interpréteur de commandes (Shell) de l’utilisateur root ? 

commande:
>grep root /etc/passwd

réponse:
>root:x:0:0:root:/root:/bin/bash

C'est donc /bin/bash

18. Si vous regardez la liste des comptes présents sur la machine, vous verrez qu’il en existe un nommé nobody. A quoi correspond-il ? 

C'est un utilisateur qui n'a aucun accès, permet au programme daemon de discuter et de faire des recherches ds d'autres programmes.

19. Par défaut, combien de temps la commande sudo conserve-t-elle votre mot de passe en mémoire ? Quelle commande permet de forcer sudo à oublier votre mot de passe ?

Le mdp expire au bout de 5min (par défaut).
pour forcer sudo a oublier directement, on utilise la commande **exit** ou bien **^D**


## Exercice 2. Gestion des permissions

1. Dans votre $HOME, créez un dossier test, et dans ce dossier un fichier fichier contenant quelques lignes de texte. Quels sont les droits sur test et fichier ? 

pour avoir les infos on utilise:
>ls -l

et on obtien en réponse:
>-rw-rw-r-- 1 arthur arthur 15 Nov 17 11:05 fichier
pour le fichier
>drwxrwxr-x 2 arthur arthur   4096 Nov 17 11:05 test
pour le dossier test

2. Retirez tous les droits sur ce fichier (même pour vous), puis essayez de le modifier et de l’afficher en tant que root. Conclusion ? 

Pour retirer tout les droit:
>chmod 000 fichier

le résultat:
>---------- 1 arthur arthur 15 Nov 17 11:05 fichier

On peut encore le modifier et l'afficher en tant que root.
Root a bien des accès qui supplante tout le monde.

3. Redonnez vous les droits en écriture et exécution sur fichier puis exécutez la commande echo "echo Hello" > fichier. On a vu lors des TP précédents que cette commande remplace le contenu d’un fichier s’il existe déjà. Que peut-on dire au sujet des droits ? 

Commande:
>chmod 300 fichier


4. Essayez d’exécuter le fichier. Est-ce que cela fonctionne ? Et avec sudo ? Expliquez. 

Commande:
>11:14 - arthur@server:~/test$ ./fichier
>bash: ./fichier: Permission denied

avec sudo, on peut l'executer:
>11:15 - arthur@server:~/test$ sudo ./fichier
>Hello

On ne peut pas éxécuter le fichier car on n'a pas les droits en lecture sur le fichier.

5. Placez-vous dans le répertoire test, et retirez-vous le droit en lecture pour ce répertoire. Listez le contenu du répertoire, puis exécutez ou affichez le contenu du fichier fichier. Qu’en déduisez-vous ? Rétablissez le droit en lecture sur test. 

commande:
>chmod u-w test

Cela ne change rien, je peux toujours accéder au fichier et les exécuter. C'est parce que j'ai encore les accès via mon groupe donc je peux encore accéder aux fichier et les lires/exécuter.

6. Créez dans test un fichier nouveau ainsi qu’un répertoire sstest. Retirez au fichier nouveau et au répertoire test le droit en écriture. Tentez de modifier le fichier nouveau. Rétablissez ensuite le droit en écriture au répertoire test. Tentez de modifier le fichier nouveau, puis de le supprimer. Que pouvezvous déduire de toutes ces manipulations ? 

commande:
>chmod a-w test
>chmod a-w nouveau

résultat:
>14:26 - arthur@server:~/test$ echo "echo Hello" > nouveau
>-bash: nouveau: Permission denied
>14:32 - arthur@server:~/test$ rm nouveau
>rm: remove write-protected regular empty file 'nouveau'? y
>rm: cannot remove 'nouveau': Permission denied

Dans le cas du rm, on me demande l'autorisation de supprimer ou non le fichier.

Une fois le droit en écriture rétablit sur test, on a toujours pas la possibilité de modifier nouveau, on peut le supprimer et on nous demande de nouveau si on peut le faire.


7. Positionnez vous dans votre répertoire personnel, puis retirez le droit en exécution du répertoire test. Tentez de créer, supprimer, ou modifier un fichier dans le répertoire test, de vous y déplacer, d’en lister le contenu, etc…Qu’en déduisez vous quant au sens du droit en exécution pour les répertoires ? 

Commande:
>chmod a-x test

On ne peut plus accéder au dossier lorsque on enlève le droit d'exécution.
>14:35 - arthur@server:~$ cd test
>-bash: cd: test: Permission denied

le droit d'exécution sur les répertoire conditionne donc l'accès a ceux ci ainsi que l'accès en lecture. On doit avoir a la fois r et x pour accéder a un répertoire.

8. Rétablissez le droit en exécution du répertoire test. Positionnez vous dans ce répertoire et retirez lui à nouveau le droit d’exécution. Essayez de créer, supprimer et modifier un fichier dans le répertoire test, de vous déplacer dans ssrep, de lister son contenu. Qu’en concluez-vous quant à l’influence des droits que l’on possède sur le répertoire courant ? Peut-on retourner dans le répertoire parent avec ”cd ..” ? Pouvez-vous donner une explication ? 



9. Rétablissez le droit en exécution du répertoire test. Attribuez au fichier fichier les droits suffisants pour qu’une autre personne de votre groupe puisse y accéder en lecture, mais pas en écriture. 

Commande:
>chmod 640 fichier 

résultat:
>-rw-r----- 1 arthur arthur   11 Nov 17 11:14 fichier
 
 10. Définissez un umask très restrictif qui interdit à quiconque à part vous l’accès en lecture ou en écriture, ainsi que la traversée de vos répertoires. Testez sur un nouveau fichier et un nouveau répertoire. 

Commande:
>14:51 - arthur@server:~/test$ umask 077
>14:52 - arthur@server:~/test$ umask -S
>u=rwx,g=,o=

en créant un nouveau fichier/dossier:
>drwx------ 2 arthur arthur 4096 Nov 17 14:53 newdir
>-rw------- 1 arthur arthur    0 Nov 17 14:53 newfichier

On voit bien que seul l'utilisateur a les accès sur ce dossier et ce fichier.

11. Définissez un umask très permissif qui autorise tout le monde à lire vos fichiers et traverser vos répertoires, mais n’autorise que vous à écrire. Testez sur un nouveau fichier et un nouveau répertoire. 

Commande:
>14:54 - arthur@server:~/test$ umask 022
>14:56 - arthur@server:~/test$ umask -S
>u=rwx,g=rx,o=rx

en créant un nouveau fichier/dossier:
>drwxr-xr-x 2 arthur arthur 4096 Nov 17 14:57 newdir
>-rw-r--r-- 1 arthur arthur    0 Nov 17 14:57 newfichier


12. Définissez un umask équilibré qui vous autorise un accès complet et autorise un accès en lecture aux membres de votre groupe. Testez sur un nouveau fichier et un nouveau répertoire. 

Commande:
>14:59 - arthur@server:~/test$ umask 033
>14:59 - arthur@server:~/test$ umask -S
>u=rwx,g=r,o=r

en créant un nouveau fichier/dossier:
>drwxr--r-- 2 arthur arthur 4096 Nov 17 15:00 newdir
>-rw-r--r-- 1 arthur arthur    0 Nov 17 15:00 newfichier


13. Transcrivez les commandes suivantes de la notation classique à la notation octale ou vice-versa (vous pourrez vous aider de la commande stat pour valider vos réponses) :

 >- chmod u=rx,g=wx,o=r fic 
> --->chmod 534 fic

 >- chmod uo+w,g-rx fic en sachant que les droits initiaux de fic sont r--r-x--- 
> --->chmod 602 fic

> - chmod 653 fic en sachant que les droits initiaux de fic sont 711 
> --->chmod u-x,g+r,o+w fic

 >- chmod u+x,g=w,o-r fic en sachant que les droits initiaux de fic sont r--r-x--- 
> ---> chmod 540 fic 

14. Affichez les droits sur le programme passwd. Que remarquez-vous ? En affichant les droits du fichier /etc/passwd, pouvez-vous justifier les permissions sur le programme passwd ? 

Commande:
>15:09 - arthur@server:~$ ll /etc/passwd
>-rw-r--r-- 1 root root 1914 Nov 17 09:39 /etc/passwd

Les accès de root sont en lecture-écriture et pour le reste des users seulement en lecture.
C'est logique que les utilisateurs n'ai pas accès en ecriture sur des fichier de config important comme passwd. Il est logique que root puisse modifier le fichier car il peut créer de nouveaux utilisateurs.

Pour les plus rapides : 

15. Access Control Lists (ACL) : suivez le tutoriel de cette page : https://doc.ubuntu-fr.org/acl. 
16. Quotas disques : suivez le tutoriel de cette page : https://doc.ubuntu-fr.org/quota




> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzU0NTA1MzUzLDgzNjUzNzM0NF19
-->