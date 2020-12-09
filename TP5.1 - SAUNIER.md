
# TP 5 - Gestion des disques / Tâches d’administration (1)

**SAUNIER Arthur**

Dans ce sixième TP, nous allons manipuler divers outils de gestion des disques et partitions, ainsi que le partitionnement LVM. Nous verrons comment créer des points de montage et monter des partitions. 

## Exercice 1. Disques et partitions 

1. Dans l’interface de configuration de votre VM, créez un second disque dur, de 5 Go dynamiquement alloués ; puis démarrez la VM 

2. Vérifiez que ce nouveau disque dur est bien détecté par le système 

Commande: 
>lsblk

Le nouveau disque est: sdb de 5giga.

```bash
07:24 - arthur@server:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0   55M  1 loop /snap/core18/1880
loop1                       7:1    0 55.4M  1 loop /snap/core18/1932
loop2                       7:2    0 70.6M  1 loop /snap/lxd/16922
loop3                       7:3    0 67.8M  1 loop /snap/lxd/18150
loop4                       7:4    0 29.9M  1 loop /snap/snapd/8542
loop5                       7:5    0   31M  1 loop /snap/snapd/9721
sda                         8:0    0   10G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0    9G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0    9G  0 lvm  /
sdb                         8:16   0    5G  0 disk
sr0                        11:0    1 1024M  0 rom

```

3. Partitionnez ce disque en utilisant fdisk : créez une première partition de 2 Go de type Linux (n°83), et une seconde partition de 3 Go en NTFS77 (n°7) 

Commande: 
>sudo fdisk /dev/sdb

suite de commande à utiliser:
n, p, [entré], +2G

```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.
```
Pour changer le type de partition:
>t dans fdsik, puis choix de la partition (2 ici) et code ici 7 pour NTFS77

```
08:43 - arthur@server:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0   55M  1 loop /snap/core18/1880
loop1                       7:1    0 55.4M  1 loop /snap/core18/1932
loop2                       7:2    0 70.6M  1 loop /snap/lxd/16922
loop3                       7:3    0 67.8M  1 loop /snap/lxd/18150
loop4                       7:4    0 29.9M  1 loop /snap/snapd/8542
loop5                       7:5    0   31M  1 loop /snap/snapd/9721
sda                         8:0    0   10G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0    9G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0    9G  0 lvm  /
sdb                         8:16   0    5G  0 disk
├─sdb1                      8:17   0    2G  0 part
└─sdb2                      8:18   0    3G  0 part
sr0                        11:0    1 1024M  0 rom
```
On voit bien la création des partitions sur le disk sdb

4. A ce stade, les partitions ont été créées, mais elles n’ont pas été formatées avec leur système de fichiers. A l’aide de la commande mkfs, formatez vos deux partitions (**Atttention** pensez à consulter le manuel !) 

Commande:
pour formater la partition 1 en ext4:
>sudo mkfs.ext4 /dev/sdb1

pour formater la partition 2 en ntfs:
> sudo mkfs.ntfs /dev/sdb2


5. Pourquoi la commande df -T, qui affiche le type de système de fichier des partitions, ne fonctionne-telle pas sur notre disque ? 

df donne l'occupation des différents espace disques

```
09:13 - arthur@server:~$ df -T
Filesystem                        Type     1K-blocks    Used Available Use% Mounted on
udev                              devtmpfs    458500       0    458500   0% /dev
tmpfs                             tmpfs       100488    1076     99412   2% /run
/dev/mapper/ubuntu--vg-ubuntu--lv ext4       9219412 5758576   2972800  66% /
tmpfs                             tmpfs       502424       0    502424   0% /dev/shm
tmpfs                             tmpfs         5120       0      5120   0% /run/lock
tmpfs                             tmpfs       502424       0    502424   0% /sys/fs/cgroup
/dev/sda2                         ext4        999320  201644    728864  22% /boot
/dev/loop0                        squashfs     56320   56320         0 100% /snap/core18/1880
/dev/loop1                        squashfs     56704   56704         0 100% /snap/core18/1932
/dev/loop2                        squashfs     72320   72320         0 100% /snap/lxd/16922
/dev/loop3                        squashfs     69376   69376         0 100% /snap/lxd/18150
/dev/loop4                        squashfs     30720   30720         0 100% /snap/snapd/8542
/dev/loop5                        squashfs     31744   31744         0 100% /snap/snapd/9721
tmpfs                             tmpfs       100484       0    100484   0% /run/user/1000
```

On ne vois pas sdb. On obtient pas les infos que l'on souhaite car sdb n'a pas été monté.

6. Faites en sorte que les deux partitions créées soient montées automatiquement au démarrage de la machine, respectivement dans les points de montage /data et /win (vous pourrez vous passer des UUID en raison de l’impossibilité d’effectuer des copier-coller) 

Il faut modifier le fichier **/etc/fstab**

**Ne pas oublier de créer data et win**

Commande: 
>sudo nano /etc/fstab

Pour avoir la liste des partitions avec leurs uuid:
>sudo blkid

Ci-dessous le fichier fstab modifié:
```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-ywdMpOmlTGGJ6B2jX34w4gcKvtF5DUmEqiCzFpIlqYstVWe5Ewkc1G28VefMgqRW / ext4 defaults 0 0
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/874ea0bd-ab3f-4eb1-8443-57ac973f8b7d /boot ext4 defaults 0 0
/swap.img       none    swap    sw      0       0
# dev/sdb1
UUID=2f807f4d-7ac7-4f75-8fd1-76e559d43c7f /data ext4 defaults 0 0
# dev/sdb2
UUID=1FBCACF35EBA24E8 /win ntfs defaults 0 0

```

7. Utilisez la commande mount puis redémarrez votre VM pour valider la configuration 

Commande:
>sudo mount-a

mount -a permet de parcourir le fichier fstab pour monter les partitions.

8. Montez votre clé USB dans la VM 

9. Créez un dossier partagé entre votre VM et votre système hôte (rem. il peut être nécessaire d’installer les Additions invité de VirtualBox). 
 
## Exercice 2 Partitionnement LVM 
 
 Dans cet exercice, nous allons aborder le partitionnement LVM, beaucoup plus flexible pour manipuler les disques et les partitions. 

1. On va réutiliser le disque de 5 Gio de l’exercice précédent. Commencez par démonter les systèmes de fichiers montés dans /data et /win s’ils sont encore montés, et supprimez les lignes correspondantes du fichier /etc/fstab 

pour démonter les systèmes de fichier:
>sudo umount /data
>sudo umount /win

Ne pas oublier de supprimer les lignes de **/etc/fstab**.
 
 2. Supprimez les deux partitions du disque, et créez une partition unique de type LVM 
 ```
La création d’une partition LVM n’est pas indispensable, mais vivement recommandée 
quand on utilise LVM sur un disque entier.
En effet, elle permet d’indiquer à d’autres OS ou logiciels de gestion de 
disques (qui ne reconnaissent pas forcément le format LVM) qu’il y a des données
sur ce disque.  
Attention à ne pas supprimer la partition système ! 
 ```
commande:
pour supprimer les partitions:
>sudo fdisk /dev/sdb
puis d

Créer une nouvelle partition à l'aide de fdisk et lui donner le type 8e pour linux LVM.

3. A l’aide de la commande pvcreate, créez un volume physique LVM. Validez qu’il est bien créé, en utilisant la commande pvdisplay. 
 **Toutes les commandes concernant les volumes physiques commencent par pv. Celles concernant les groupes de volumes commencent par vg, et celles concernant les volumes logiques par lv.**

```bash
10:52 - arthur@server:~$ sudo pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
```

```bash
10:53 - arthur@server:~$ sudo pvdisplay /dev/sdb1
  "/dev/sdb1" is a new physical volume of "<5.00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               <5.00 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               9HCKDN-a9qO-btdQ-lirB-IDk4-2yyR-9THlU5
```

  
 4. A l’aide de la commande vgcreate, créez un groupe de volumes, qui pour l’instant ne contiendra que le volume physique créé à l’étape précédente. Vérifiez à l’aide de la commande vgdisplay.  **Par convention, on nomme généralement les groupes de volumes vgxx (où xx représente l’indice du groupe de volume, en commençant par 00, puis 01...)**

```bash
10:54 - arthur@server:~$ sudo vgcreate gp1 /dev/sdb1
  Volume group "gp1" successfully created
10:56 - arthur@server:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               gp1
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <5.00 GiB
  PE Size               4.00 MiB
  Total PE              1279
  Alloc PE / Size       0 / 0
  Free  PE / Size       1279 / <5.00 GiB
  VG UUID               MQ2wdm-GBjt-b2Gc-fwPn-ryhF-RYWq-9uSqdq

  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <9.00 GiB
  PE Size               4.00 MiB
  Total PE              2303
  Alloc PE / Size       2303 / <9.00 GiB
  Free  PE / Size       0 / 0
  VG UUID               ywdMpO-mlTG-GJ6B-2jX3-4w4g-cKvt-F5DUmE
```
 
5. Créez un volume logique appelé lvData occupant l’intégralité de l’espace disque disponible.  **On peut renseigner la taille d’un volume logique soit de manière absolue avec l’option -L (par exemple -L 10G pour créer un volume de 10 Gio), soit de manière relative avec l’option -l : -l 60%VG pour utiliser 60% de l’espace total du groupe de volumes, ou encore -l 100%FREE pour utiliser la totalité de l’espace libre.**

Commande:
```bash
10:56 - arthur@server:~$ sudo lvcreate -l 100%FREE -n lvData gp1
  Logical volume "lvData" created.
```
  
 6. Dans ce volume logique, créez une partition que vous formaterez en ext4, puis procédez comme dans l’exercice 1 pour qu’elle soit montée automatiquement, au démarrage de la machine, dans /data.  A ce stade, l’utilité de LVM peut paraître limitée. Il trouve tout son intérêt quand on veut par exemple agrandir une partition à l’aide d’un nouveau disque.

Commande:
Pour voir le volume logique
```bash
15:41 - arthur@server:/dev/gp1$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/gp1/lvData
  LV Name                lvData
  VG Name                gp1
  LV UUID                PXSQr3-Ceo1-u4TT-Kbot-qm61-GTF8-QlgfWJ
  LV Write Access        read/write
  LV Creation host, time server, 2020-11-30 11:02:35 +0000
  LV Status              available
  # open                 0
  LV Size                <5.00 GiB
  Current LE             1279
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1

  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                35mVCF-h5Yf-F7A8-ZWn9-IRZi-BbBy-EO17So
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2020-12-01 07:53:48 +0000
  LV Status              available
  # open                 1
  LV Size                <9.00 GiB
  Current LE             2303
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0


```
Pour créer la partition en ext4:
```bash
15:50 - arthur@server:~$ sudo mkfs.ext4 /dev/gp1/lvData
mke2fs 1.45.5 (07-Jan-2020)
Found a dos partition table in /dev/gp1/lvData
Proceed anyway? (y,N) y
Creating filesystem with 1309696 4k blocks and 327680 inodes
Filesystem UUID: 383e58a0-3b16-4d60-963d-47e619a48312
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

Ensuite modifier le fichier /etc/fstab pour monter la partition au démarrage de la VM.

enfin, faire:
>sudo mount -a

Puis redémarrer la VM pour vérifier que cela fonctionne bien.

si on fait la commande **mount** après le redémarrage, on peut voir cette ligne qui montre que cela a fonctionné:
```bash
/dev/mapper/gp1-lvData on /home/arthur/data type ext4 (rw,relatime)
```

 7. Eteignez la VM pour ajouter un second disque (peu importe la taille pour cet exercice). Redémarrez la VM, vérifiez que le disque est bien présent. Puis, répétez les questions 2 et 3 sur ce nouveau disque. 

Pour créer la nouvelle partition:
```
15:57 - arthur@server:~$ sudo fdisk /dev/sdc
[sudo] password for arthur:

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x6dd93e10.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759):

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'.
```

Ensuite pour créer un volume lvm:
```bash
16:00 - arthur@server:~$ sudo pvcreate /dev/sdc1
  Physical volume "/dev/sdc1" successfully created.
```

8. Utilisez la commande vgextend pour ajouter le nouveau disque au groupe de volumes 

la commande:
```bash
sudo vgextend gp1 /dev/sdc1

16:10 - arthur@server:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               gp1
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               9.99 GiB
  PE Size               4.00 MiB
  Total PE              2558
  Alloc PE / Size       1279 / <5.00 GiB
  Free  PE / Size       1279 / <5.00 GiB
  VG UUID               MQ2wdm-GBjt-b2Gc-fwPn-ryhF-RYWq-9uSqdq
```
on voit bien que notre groupe fait 9.99GiB donc on a doublé sa taille en ajoutant sdc1.

 9. Utilisez la commande lvresize (ou lvextend) pour agrandir le volume logique. Enfin, il ne faut pas oublier de redimensionner le système de fichiers à l’aide de la commande resize2fs. 
 > Il est possible d’aller beaucoup plus loin avec LVM, par exemple en créant des volumes par bandes (l’équivalent du RAID 0) ou du mirroring (RAID 1). Le but de cet exercice n’était que de présenter les fonctionnalités de base.

Commande pour étendre le volume logique:
```bash
16:15 - arthur@server:~$ sudo lvextend -L+4.99G /dev/gp1/lvData
  Rounding size to boundary between physical extents: 4.99 GiB.
  Size of logical volume gp1/lvData changed from 5.00 GiB (1280 extents) to 9.99 GiB (2558 extents).
  Logical volume gp1/lvData successfully resized.
```

Pour augmenter la taille du système de fichier:
```bash
16:16 - arthur@server:~$ sudo resize2fs /dev/gp1/lvData
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/gp1/lvData is mounted on /home/arthur/data; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/gp1/lvData is now 2619392 (4k) blocks long.
```

##  Exercice 3. Exécution de commandes en différé : at et cron 

**crontab.guru** utilise pour tester des programmation de cron.

1. Programmez une tâche qui affiche un rappel pour la réunion qui aura lieu dans 3 minutes. Vérifiez entre temps que la tâche est bien programmée. 

Pour programmer une tache:
```bash
16:29 - arthur@server:~$ at now + 3 minutes
warning: commands will be executed using /bin/sh
at> echo reunion
at> <EOT>
job 1 at Sun Dec  6 17:32:00 2020
```

utiliser ^D pour fermer le at et sauvegarder.

2. Est-ce que le message s’est affiché ? Si la réponse est non, essayez de trouver la cause du problème (par exemple en vous aidant des logs, du manuel...) 

Non, le message ne s'affiche pas.
Si on va ds **/var/log** et qu'on ouvre le fichier **syslog** on peut voir:
```bash
Dec  6 16:36:00 server atd[1361]: Exec failed for mail command: No such file or directory
```

La commande at envoie la commande par mail ce qui n'est pas setup pour nous.

3. Pour tester le fonctionnement de cron, commencez par programmer l’exécution d’une tâche simple, l’affichage de “Il faut réviser pour l’examen !”, toutes les 3 minutes. 

Commande pour éditer une nouvelle tache via crontab:
```bash
16:23 - arthur@server:~$ crontab -e
no crontab for arthur - using an empty one

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/vim.basic
  3. /usr/bin/vim.tiny
  4. /bin/ed

Choose 1-4 [1]: 1
No modification made
```

4. Programmez l’exécution d’une commande tous les jours, toute l’année, tous les quarts d’heure 

Commande tout les jours:
> 0 0 * * * echo hello

Commande toute las années:
> 0 0 1 1 * echo hello

Commande toute les 15minutes:
>*/15 * * * * echo hello

5. Programmez l’exécution d’une commande toutes les cinq minutes à partir de 2 (2, 7, 12, etc.) à 18 heures les 1er et 15 du mois : 

>2/5 18 1,25 * * echo hello

6. Programmez l’exécution d’une commande du lundi au vendredi à 17 heures 

>0 */17 * * 1-5 echo hello

7. Modifiez votre crontab pour que les messages ne soient plus envoyés par mail, mais redirigés dans un fichier de log situé dans votre dossier personnel 

il faut modifier les commande cron:
```
* * * * * echo hello >> /home/arthur/cron.log 2>&1
```

8. Videz votre crontab

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTk4MjA5NDEyLDI4OTY2NDIyMyw5NTg5Mz
czMDQsLTI0MDY0OTU2OCwtMzQ3Nzk4MDcyLDIwNzQ2Mzk0ODYs
MTQ2NDE5OTk2MCwtMTUxNjQ2MDg0MiwtOTU1NzM4MjI2LC0xMT
U0NjIxNTkxLDExODEzMzAzMTIsMTY2Njk1Mjg0MSwtMTIyMDkw
MDE2Nyw2NjkyMzUzNDEsLTExMTkwNjExMDRdfQ==
-->