# TP 7 - Boot, services et processus / Tâches d’administration

## Exercice 1. Personnalisation de GRUB 

GRUB est considérablement paramétrable : résolution, langue, fond d’écran, thème, disposition du clavier…. 
> GRUB se configure via un fichier de paramètres (/etc/default/grub), mais aussi par des scripts situés dans /etc/grub.d ; ces scripts commencent tous par un numéro et sont traités dans l’ordre. 

> Evidemment, seuls les scripts exécutables sont pris en compte. 

> Sous Ubuntu Server, GRUB prend aussi en compte les fichiers d’extension .cfg présents dans /etc/default/grub.d. En particulier, sur les versions récentes, le fichier de configuration 50-curtin-settings.cfg donne à la variable GRUB_TERMINAL la valeur console, ce qui désactive tous les paramètres liés aux fonds d’écran, thèmes, certaines résolutions, etc. 

1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il est présent dans votre environnement (vous pouvez aussi commenter son contenu). 

Le fichier n'existe pas ds le dossier grub.d
Pour le créer:
touch
```bash
# Set the recordfail timeout
GRUB_RECORDFAIL_TIMEOUT=0

# Do not wait on grub prompt
GRUB_TIMEOUT=10

# Set the default commandline
GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0"

# Set the grub console type
GRUB_TERMINAL=console
```

2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ; passé ce délai, le premier OS du menu doit être lancé automatiquement. 

Commande:
>sudo nano grub

dans le fichier, changer les champs:
>GRUB_TIMEOUT_STYLE=menu
>GRUB_TIMEOUT=10

3. Lancez la commande update-grub 
> Cette commande fait appel au script grub-mkconfig qui construit le fichier de configuration ”final” de GRUB (/boot/grub/grub.cfg) à partir du fichier de paramètres et des scripts. 

```bash
  thererdfalto grubo
 e tdefault'
Sourcing file `/etc/default/grub.d/50-curtin-settings.cfg'
Sourcing file `/etc/default/grub.d/init-select.cfg'
Generating grub configuration ficndline
Gconole onlet0
 t e gru cone te
one
```

4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte 
> Pensez à lancer la commande update-grub après chaque modification de la configuration de GRUB ! 

On voit bien que grub fonctionne

5. On va augmenter la suin de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres à rajouter au fichier grub. 

Dans etc/default/grub
a c suivant:
>GRUB_GFXMODE=1024x768


6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splashimages (après installation, celles-ci sont disponibles dans /usr/share/images/grub). 

Commande: 
>sudo apt install grub2-splashimages

On rajoute le champ: 
>GRUB_BACKGROUND="/usr/share/images/grub/image_name"

ne pas oublier de faire un update-grub

7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*). Installez-en un. 

j'ai récupérer le thème fallout sur ce site :)

https://www.addictivetips.com/ubuntu-linux-tips/best-grub2-bootloader-themes-on-linux/

8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer. 

dans le fichier: etc/grub.d/40_custom

```
menuentry 'Arrêt du système' {​​​​​​​ 
	halt 
}​​​​​​​ 
menuentry 'Redémarrage du système' {​​​​​​​ 
	reboot 
}​​​​​​​```

```


10. Configurez GRUB pour que le clavier soit en français **( n’a plus l’air de fonctionner sur Ubuntu ≥ 19.04)**

## Exercice 2. Noyau 

Dans cet exercice, on va créer et installer un module pour le noyau. 

1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres). 

2. Créez un fichier hello.c contenant le code suivant :
```c
#include <linux/module.h>  
#include <linux/kernel.h>

MODULE_LICENSE("GPL");  
MODULE_AUTHOR("John Doe");  
MODULE_DESCRIPTION("Module hello world");  
MODULE_VERSION("Version 1.00");  
int init_module(void)  
{  
printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");  
return 0;  
}

void cleanup_module(void)  
{  
printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");  
}
```


3. Créez également un fichier Makefile :
```makefile
obj-m += hello.o

all:  
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:  
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean

install:  
	cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
```


4. Compilez le module à l’aide de la commande make, puis installez-le à l’aide de la commande make install. 
> Le module est installé dans le dossier spécifié à la ligne 10. 

```bash
17:26 - arthur@server:~$ make
make -C /lib/modules/5.4.0-56-generic/build M=/home/arthur modules
make[1]: Entering directory '/usr/src/linux-headers-5.4.0-56-generic'
  CC [M]  /home/arthur/hello.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC [M]  /home/arthur/hello.mod.o
  LD [M]  /home/arthur/hello.ko
make[1]: Leaving directory '/usr/src/linux-headers-5.4.0-56-generic'
```

```
17:28 - arthur@server:~$ sudo make install
[sudo] password for arthur:
cp ./hello.ko /lib/modules/5.4.0-56-generic/kernel/drivers/misc
```

5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod. 

pour charger le module utiliser insmod:
```bash
18:21 - arthur@server:/lib/modules/5.4.0-56-generic/kernel/drivers/misc$ sudo insmod hello.ko
18:21 - arthur@server:/lib/modules/5.4.0-56-generic/kernel/drivers/misc$ lsmod | grep hello
hello                  16384  0
```

6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez notamment voir les informations figurant dans le fichier C. 

On retrouve effectivement les infos données dans le .c

```bash
18:23 - arthur@server:/lib/modules/5.4.0-56-generic/kernel/drivers/misc$ modinfo hello.ko
filename:       /lib/modules/5.4.0-56-generic/kernel/drivers/misc/hello.ko
version:        Version 1.00
description:    Module hello world
author:         John Doe
license:        GPL
srcversion:     4398A2271F215E3A6F58078
depends:
retpoline:      Y
name:           hello
vermagic:       5.4.0-56-generic SMP mod_unload

```


7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module() est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande lsmod. 

On peut utiliser la commande modprobe (qui ne fonctionne pas ds notre cas.

```
18:25 - arthur@server:/lib/modules/5.4.0-56-generic/kernel/drivers/misc$ sudo rmmod hello.ko
18:25 - arthur@server:/lib/modules/5.4.0-56-generic/kernel/drivers/misc$ lsmod | grep hello
18:26 - arthur@server:/lib/modules/5.4.0-56-generic/kernel/drivers/misc$

```

8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.


## Exercice 3. Interception de signaux 

La commande interne trap permet de redéfinir des gestionnaires pour les signaux reçus par un processus. Un cas d’utilisation typique est la suppression des fichiers temporaires créés par un script lorsque celui-ci est interrompu. 

1. Commencez par écrire un script qui recopie dans un fichier tmp.txt chaque ligne saisie au clavier par l’utilisateur 

```bash
#!/bin/bash

while true
do
 read line
 echo $line >> tmp.txt
done
exit 0
```

2. Lancez votre script et appuyez sur CTRL+Z. Que se passe-t-il ? Comment faire pour que le script poursuive son exécution ? 

CTRL+Z interrompt le script:
```bash
22:11 - arthur@server:~/script$ ./recop.sh
hello
^Z
[1]+  Stopped                 ./recop.sh
22:15 - arthur@server:~/script$ bg
[1]+ ./recop.sh &
```
En réalité le script est passé en arrière plan, pour le ramener, il faut:
```bash
22:15 - arthur@server:~/script$ fg 1
./recop.sh
hello
^C
```

Avec ma commande fg plus le numéro associé, on peut ramener un processus au premier plan.

3. Toujours pendant l’exécution du script, appuyez sur CTRL+C. Que se passe-t-il ? 

CTRL+C force l'arrêt du script. Si on veux le relancer, on doit le faire depuis le début. Il n'est pas mis en pause.

4. Modifiez votre script pour redéfinir les actions à effectuer quand le script reçoit les signaux SIGTSTP (= CTRL+Z) et SIGINT (= CTRL+C) : dans le premier cas, il doit afficher ”Impossible de me placer en arrière-plan”, et dans le second cas, il doit afficher ”OK, je fais un peu de ménage avant” avant de supprimer le fichier temporaire et terminer le script. 

```bash
#!/bin/bash


trap message SIGTSTP
trap cleanup SIGINT

message()
{
 echo "impossible de placer le programme en arrière plan"
}

cleanup()
{
 echo "OK, je fais un peu de ménage avant";
 rm tmp.txt;
 exit
}

while true
do
 read line
 echo $line >> tmp.txt
done
exit 0

```

On utilise la commande trap pour récupérer les signaux.
La syntaxe d'utilisation est:
>trap commande signal_to_catch

5. Testez le nouveau comportement de votre script en utilisant d’une part les raccourcis clavier, d’autre part la commande kill 

commande kill:
>killall recop.sh

Stoppe le script sans aucun message spécifique. Alors que maintenant, CTRL+C stoppe le script avec le message qu'on lui a donné et supprime le fichier .txt
CTRL+Z ne met plus en pause le script, mais affiche juste un message.

6. Relancez votre script et faites immédiatement un CTRL+C : vous obtenez un message d’erreur vous indiquant que le fichier tmp.txt n’existe pas. A l’aide de la commande interne test, corrigez votre script pour que ce message n’apparaisse plus. 

```bash
#!/bin/bash


trap message SIGTSTP
trap cleanup SIGINT

message()
{
 echo "impossible de placer le programme en arrière plan"
}

cleanup()
{
 echo "OK, je fais un peu de ménage avant";
 if test -e "tmp.txt"; then
  rm tmp.txt;
 fi
 exit
}

while true
do
 read line
 echo $line >> tmp.txt
done
exit 0
```

## Exercice 4. Surveillance de l’activité du système 

1. Lancez la commande htop, puis ouvrez un second terminal (avec Alt + F2, ou Alt + F3...) et tapez la commande w dans tty2. Qu’affiche cette commande ? 

La commande w affiche qui est connecté sur la machine.
On peut voir que l'utilisateur arthur est connecté 2 fois sur la machine, sur tty1 et tty2.

2. Comment afficher l’historique des dernières connexions à la machine ? 

commande:
>last

```py
22:43 - arthur@server:~$ last
arthur   tty2                          Sun Dec  6 22:36   still logged in
arthur   pts/0        10.0.2.2         Sun Dec  6 18:30   still logged in
arthur   tty1                          Sun Dec  6 18:30   still logged in
reboot   system boot  5.4.0-56-generic Sun Dec  6 18:28   still running
arthur   pts/0        10.0.2.2         Sun Dec  6 15:57 - 18:28  (02:30)
arthur   tty1                          Sun Dec  6 15:57 - down   (02:31)
reboot   system boot  5.4.0-56-generic Sun Dec  6 15:56 - 18:28  (02:31)
arthur   pts/0        10.0.2.2         Sun Dec  6 15:54 - crash  (00:02)
arthur   tty1                          Sun Dec  6 15:54 - crash  (00:02)
reboot   system boot  5.4.0-56-generic Sun Dec  6 15:53 - 18:28  (02:34)
arthur   pts/0        10.0.2.2         Sat Dec  5 18:16 - crash  (21:37)
arthur   tty1                          Sat Dec  5 17:45 - crash  (22:08)
reboot   system boot  5.4.0-56-generic Sat Dec  5 17:42 - 18:28 (1+00:45)
arthur   pts/0        10.0.2.2         Thu Dec  3 10:27 - crash (2+07:15)
arthur   tty1                          Thu Dec  3 10:27 - crash (2+07:15)
reboot   system boot  5.4.0-56-generic Thu Dec  3 10:24 - 18:28 (3+08:03)
arthur   pts/0        10.0.2.2         Thu Dec  3 10:08 - 10:24  (00:15)
arthur   tty1                          Thu Dec  3 10:08 - down   (00:16)
reboot   system boot  5.4.0-56-generic Thu Dec  3 10:08 - 10:24  (00:16)
arthur   pts/0        10.0.2.2         Thu Dec  3 10:06 - 10:07  (00:01)
arthur   tty1                          Thu Dec  3 10:04 - down   (00:03)
reboot   system boot  5.4.0-56-generic Thu Dec  3 10:03 - 10:07  (00:04)
reboot   system boot  5.4.0-56-generic Thu Dec  3 09:17 - 10:07  (00:50)
arthur   tty1                          Tue Dec  1 10:44 - crash (1+22:32)
reboot   system boot  5.4.0-56-generic Tue Dec  1 10:22 - 10:07 (1+23:45)
arthur   tty1                          Tue Dec  1 10:19 - crash  (00:02)
reboot   system boot  5.4.0-56-generic Tue Dec  1 10:19 - 10:07 (1+23:48)
arthur   tty1                          Tue Dec  1 09:45 - crash  (00:33)
reboot   system boot  5.4.0-56-generic Tue Dec  1 09:44 - 10:07 (2+00:22)
arthur   tty1                          Tue Dec  1 09:36 - crash  (00:08)
reboot   system boot  5.4.0-56-generic Tue Dec  1 09:36 - 10:07 (2+00:31)
arthur   tty1                          Tue Dec  1 08:08 - crash  (01:27)
reboot   system boot  5.4.0-56-generic Tue Dec  1 08:08 - 10:07 (2+01:59)

wtmp begins Tue Dec  1 08:08:29 2020

```

3. Quelle commande permet d’obtenir la version du noyau ? 

```
22:43 - arthur@server:~$ cat /proc/version
Linux version 5.4.0-56-generic (buildd@lgw01-amd64-025) (gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)) #62-Ubuntu SMP Mon Nov 23 19:20:19 UTC 2020
```

4. Comment récupérer toutes les informations sur le processeur, au format JSON ? 

```JSON
22:45 - arthur@server:~$ sudo lshw -class processor -json
[sudo] password for arthur:
[
  {
    "id" : "cpu",
    "class" : "processor",
    "claimed" : true,
    "product" : "Intel(R) Core(TM) i7-8750H CPU @ 2.20GHz",
    "vendor" : "Intel Corp.",
    "physid" : "2",
    "businfo" : "cpu@0",
    "width" : 64,
    "capabilities" : {
      "fpu" : "mathematical co-processor",
      "fpu_exception" : "FPU exceptions reporting",
      "wp" : true,
      "vme" : "virtual mode extensions",
      "de" : "debugging extensions",
      "pse" : "page size extensions",
      "tsc" : "time stamp counter",
      "msr" : "model-specific registers",
      "pae" : "4GB+ memory addressing (Physical Address Extension)",
      "mce" : "machine check exceptions",
      "cx8" : "compare and exchange 8-byte",
      "apic" : "on-chip advanced programmable interrupt controller (APIC)",
      "sep" : "fast system calls",
      "mtrr" : "memory type range registers",
      "pge" : "page global enable",
      "mca" : "machine check architecture",
      "cmov" : "conditional move instruction",
      "pat" : "page attribute table",
      "pse36" : "36-bit page size extensions",
      "clflush" : true,
      "mmx" : "multimedia extensions (MMX)",
      "fxsr" : "fast floating point save/restore",
      "sse" : "streaming SIMD extensions (SSE)",
      "sse2" : "streaming SIMD extensions (SSE2)",
      "ht" : "HyperThreading",
      "syscall" : "fast system calls",
      "nx" : "no-execute bit (NX)",
      "rdtscp" : true,
      "x86-64" : "64bits extensions (x86-64)",
      "constant_tsc" : true,
      "rep_good" : true,
      "nopl" : true,
      "xtopology" : true,
      "nonstop_tsc" : true,
      "cpuid" : true,
      "tsc_known_freq" : true,
      "pni" : true,
      "pclmulqdq" : true,
      "monitor" : true,
      "ssse3" : true,
      "cx16" : true,
      "pcid" : true,
      "sse4_1" : true,
      "sse4_2" : true,
      "x2apic" : true,
      "movbe" : true,
      "popcnt" : true,
      "aes" : true,
      "xsave" : true,
      "avx" : true,
      "rdrand" : true,
      "hypervisor" : true,
      "lahf_lm" : true,
      "abm" : true,
      "3dnowprefetch" : true,
      "invpcid_single" : true,
      "pti" : true,
      "fsgsbase" : true,
      "avx2" : true,
      "invpcid" : true,
      "rdseed" : true,
      "clflushopt" : true,
      "flush_l1d" : true
    }
  }
]
```


5. Comment obtenir la liste des derniers démarrages de la machine avec la commande journalctl ? Comment afficher tout ce qu’il s’est passé sur la machine lors de l’avant-dernier boot ? 

**journalctl -b** pour les logs de démarrage.
```
22:46 - arthur@server:~$ journalctl -b
-- Logs begin at Tue 2020-12-01 08:08:30 UTC, end at Sun 2020-12-06 22:46:06 UTC. --
Dec 06 18:28:18 server kernel: Linux version 5.4.0-56-generic (buildd@lgw01-amd64-025) (gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)) #62-Ubuntu SMP Mon >
Dec 06 18:28:18 server kernel: Command line: BOOT_IMAGE=/vmlinuz-5.4.0-56-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro maybe-ubiquity
Dec 06 18:28:18 server kernel: KERNEL supported cpus:
Dec 06 18:28:18 server kernel:   Intel GenuineIntel
Dec 06 18:28:18 server kernel:   AMD AuthenticAMD
[....]
```

**journalctl -b -1** pour afficher les logs de l'avant dernier boot.

```
22:53 - arthur@server:~$ journalctl -b -1
-- Logs begin at Tue 2020-12-01 08:08:30 UTC, end at Sun 2020-12-06 22:46:06 UTC. --
Dec 06 15:56:39 server kernel: Linux version 5.4.0-56-generic (buildd@lgw01-amd64-025) (gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)) #62-Ubuntu SMP Mon >
Dec 06 15:56:39 server kernel: Command line: BOOT_IMAGE=/vmlinuz-5.4.0-56-generic root=/dev/mapper/ubuntu--vg-ubuntu--lv ro maybe-ubiquity
Dec 06 15:56:39 server kernel: KERNEL supported cpus:
Dec 06 15:56:39 server kernel:   Intel GenuineIntel
Dec 06 15:56:39 server kernel:   AMD AuthenticAMD
Dec 06 15:56:39 server kernel:   Hygon HygonGenuine
[.....]
```

6. Faites en sortes que lors d’une connexion à la machine, les utilisateurs soient prévenus par un message à l’écran d’une maintenance le 26 mars à minuit. 

Il faut aller écrire dans le fichier: **/etc/motd**

Si on écrit dans ce fichier, les message apparaissent lors de la connexion. 

7. Ecrivez un script bash qui permet de calculer le k-ième nombre de Fibonacci : Fk = Fk−1 + Fk−2, avec F0 = F1 = 1. Lancez le calcul de F100 puis lancez la commande tload depuis un autre terminal virtuel. Que constatez-vous ? Interrompez ensuite le calcul avec CTRL+C et observez la conséquence sur l’affichage de tload.
```bash

```

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTcxNDQ5MTQ4LC0xOTA5NTYyOTA0LDkxOT
E1NDAzMCwtMjEzODA5ODE0NywxODAyODgzNDc0LDEzNTAzODA4
MTksLTIwNzc0Mjg3NzksMTQ3NTk0NDUzNiwtNjcyNDA0MjMyLC
00NTkyNjkxNjIsMTg2ODUyNzQ1LC0xOTc4NDk4ODIyXX0=
-->