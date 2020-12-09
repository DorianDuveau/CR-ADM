# TP 2 - Bash

**SAUNIER Arthur**

### Exercice 1
1. Dans quels dossiers bash trouve-t-il les commandes tapées par l’utilisateur ? 


2. Quelle variable d’environnement permet à la commande cd tapée sans argument de vous ramener dans votre répertoire personnel ? 


3. Explicitez le rôle des variables LANG, PWD, OLDPWD, SHELL et _. 
>Utiliser $VAR ou "echo $VAR"
- LANG : permet de spécifier la langue des programme
- PWD : stocke le path du directory dans lequel on se trouve
- OLDPWD : stocke le path du directory dans lequel on se trouvait précédemment
- SHELL: stocke le path du shell en cours d'exécution
- _ : stocke le path du shell également

4. Créez une variable locale MY_VAR (le contenu n’a pas d’importance). Vérifiez que la variable existe. 

Pour créer une variable locale, commande: **VAR="HO!"** par exemple.
Pour vérifier qu'elle existe: **echo $VAR**

5. Tapez ensuite la commande bash. Que fait-elle ? La variable MY_VAR existe-t-elle ? Expliquez. A la fin de cette question, tapez la commande exit pour revenir dans votre session initiale. 

Ici la commande bash ne fait rien. La variable MY_VAR n'est pas affiché lorsqu'on l'appelle. La variable est locale, elle ne peut donc être utilisé que par le shell courant ou elle a été déclarée.


6. Transformez MY_VAR en une variable d’environnement et recommencez la question précédente. Expliquez.

Maintenant, on peut avoir la variable MY_VAR depuis bash. MY_VAR est maintenant une variable d'environnement, elle peut être utilisé par différents programme pour modifier des comportements.

7. Créer la variable d’environnement NOMS ayant pour contenu vos noms de binômes séparés par un espace. Afficher la valeur de NOMS pour vérifier que l’affectation est correcte. 
```
10:52 - arthur@server:~$ export NOMS="SAUNIER BRUENDET"
10:54 - arthur@server:~$ printenv NOMS
SAUNIER BRUENDET
```

9. Ecrivez une commande qui affiche ”Bonjour à vous deux, binôme1 binôme2 !” (où binôme1 et binôme2 sont vos deux noms) en utilisant la variable NOMS. 
```
10:58 - arthur@server:~$ echo ""Bonjour a vous deux, $NOMS""
Bonjour a vous deux, SAUNIER BRUENDET
```

10. Quelle différence y a-t-il entre donner une valeur vide à une variable et l’utilisation de la commande unset ? 

Unset supprime complètement la variable, et free l'espace mémoire qu'elle utilise. La valeur vide fait que la variable ne vaut rien, par contre elle occupe toujours un espace mémoire.
 
11. Utilisez la commande echo pour écrire exactement la phrase : $HOME = chemin (où chemin est votre dossier personnel d’après bash)
commande: 

echo '$HOME' = $HOME


## Programmation Bash

Vous enregistrerez vos scripts dans un dossier script que vous créerez dans votre répertoire personnel. Tous les scripts sont bien entendu à tester. Ajoutez le chemin vers script à votre PATH de manière permanente.

dans .bashrc, ne pas oublier de changer la variable PATH:
>PATH=$PATH:~/mondossier

Ne pas oublier de rendre un script exécutable:
>chmod u+x script.sh


### Exercice 2

Écrivez un script testpwd.sh qui demande de saisir un mot de passe et vérifie s’il correspond ou non au contenu d’une variable PASSWORD dont le contenu est codé en dur dans le script. Le mot de passe saisi par l’utilisateur ne doit pas s’afficher.
```bash
#!/bin/bash
echo "saisissez le mot de passe"
read -s mdp

if [ $mdp = 'lol' ]; then
        echo "mot de passe correct"
else
        echo "mot de passe erroné"
fi
```

### Exercice 3

Ecrivez un script qui prend un paramètre et utilise la fonction suivante pour vérifier que ce paramètre est un nombre réel : 
```bash
function is_number() 
{ 
	re='^[+-]?[0-9]+([.][0-9]+)?$' 
	if ! [[ $1 =~ $re ]] ; then 
	  return 1 
	else
	  return 0 
	fi 
} 
```
Il affichera un message d’erreur dans le cas contraire.

```bash
#!/bin/bash
function is_number() 
{ 
	re='^[+-]?[0-9]+([.][0-9]+)?$' 
	if ! [[ $1 =~ $re ]] ; then 
	  return 1 
	else
	  return 0 
	fi 
} 

is_number $1

if ! [[ $? = 0 ]] ; then
    echo 'erreur : non réel'
fi
```


### Exercice 4

Écrivez un script qui vérifie l’existence d’un utilisateur dont le nom est donné en paramètre du script. Si le script est appelé sans nom d’utilisateur, il affiche le message : ”Utilisation : nom_du_script nom_utilisateur”, où nom_du_script est le nom de votre script récupéré automatiquement (si vous changez le nom de votre script, le message doit changer automatiquement)

```bash
#!/bin/bash
if [ $# -ne 1 ]; then
 echo "Utilisation : $0 nom_d'utilisateur"
fi
```

### Exercice 5

Écrivez un programme qui calcule la factorielle d’un entier naturel passé en paramètre (on supposera que l’utilisateur saisit toujours un entier naturel).

```bash
#!\bin/bash

val=$1
result=1

while [ $val -ne 0 ]; do
 let result=$result*$val
 let val=$val-1
done
echo "$result"
```

### Exercice 6

Écrivez un script qui génère un nombre aléatoire entre 1 et 1000 et demande à l’utilisateur de le deviner. Le programme écrira ”C’est plus !”, ”C’est moins !” ou ”Gagné !” selon les cas (vous utiliserez $RANDOM).

```bash
 #!\bin/bash
 
val=1024

random=$RANDOM
let "random %= 1000"

while [ $val -ne $random ]; do
	if [ $val -gt $random ]; then
		echo "C'est moins !"
	elif [ $val -lt $random ]; then
		echo "C'est plus !"
	else
		echo "C'est gagné !"
	fi
done 
```



### Exercice 7

1. Écrivez un script qui prend en paramètres trois entiers (entre -100 et +100) et affiche le min, le max et la moyenne. Vous pouvez réutiliser la fonction de l’exercice 3 pour vous assurer que les paramètres sont bien des entiers. 
2. Généralisez le programme à un nombre quelconque de paramètres (pensez à SHIFT) 
3. Modifiez votre programme pour que les notes ne soient plus données en paramètres, mais saisies et stockées au fur et à mesure dans un tableau.

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDIxNzI3NDYyLDc3NTMxMzM4MCwxNDA0MD
U0Mjg1XX0=
-->