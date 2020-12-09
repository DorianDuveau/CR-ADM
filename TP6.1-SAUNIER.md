
# TP 5 - Services réseau

**SAUNIER Arthur**

Dans ce sixième TP, nous allons voir comment mettre en place différents services réseau (serveur DHCP, serveur DNS, serveur web…) sur notre serveur.

## Exercice 1. Adressage IP (rappels)

Vous administrez le réseau interne 172.16.0.0/23 d’une entreprise, et devez gérer un parc de 254 machines réparties en 7 sous-réseaux. La répartition des machines est la suivante : 
- Sous-réseau 1 : 38 machines 
- Sous-réseau 2 : 33 machines 
- Sous-réseau 3 : 52 machines 
- Sous-réseau 4 : 35 machines 
- Sous-réseau 5 : 34 machines 
- Sous-réseau 6 : 37 machines 
- Sous-réseau 7 : 25 machines 
Donnez, pour chaque sous-réseau, l’adresse de sous-réseau, l’adresse de broadcast (multidiffusion) ainsi que les adresses de la première et dernière machine configurées (précisez si vous utilisez du VLSM ou pas).

On utilise du VLSM:
SR1: 172.16.0.0/26
broadcast: 172.16.0.63
1ére adresse: 172.16.0.1
dernière adresse: 172.16.0.62

SR2: 172.16.0.64/26
broadcast: 172.16.0.127
1ére adresse: 172.16.0.65
dernière adresse: 172.16.0.126

SR3: 172.16.0.128/26
broadcast: 172.16.0.191
1ére adresse: 172.16.0.129
dernière adresse: 172.16.0.190

SR4: 172.16.0.192/26
broadcast: 172.16.0.255
1ére adresse: 172.16.0.193
dernière adresse: 172.16.0.254

SR5: 172.16.1.0/26
broadcast: 172.16.1.63
1ére adresse: 172.16.1.1
dernière adresse: 172.16.1.62

SR6: 172.16.1.64/26
broadcast: 172.16.1.127
1ére adresse: 172.16.1.65
dernière adresse: 172.16.1.126

SR7: 172.16.1.92/27
broadcast: 172.16.1.123
1ére adresse: 172.16.1.93
dernière adresse: 172.16.1.122

## Exercice 2. Préparation de l’environnement

Dans ce TP nous allons mettre en place un réseau rudimentaire constitué de seulement deux machines : un serveur et un client : 
- le serveur a une connexion Internet, notamment pour télécharger les paquets nécessaires à l’installation des serveurs, et sert de passerelle au client ; 
- les deux machines appartiennent à un réseau local ; 
- le client a accès à Internet uniquement via le serveur; il dispose d’une interface réseau qui recevra son adresse IP du serveur DHCP. 

1. VM éteintes, utilisez les outils de configuration de VirtualBox pour mettre en place l’environnement décrit ci-dessus 

On a 2 machines (**Serveur** et **Client**).

-   La machine **Serveur** sert de serveur DHCP et serveur DNS pour **Client** ; elle a **deux** cartes réseau (une en NAT, reliée à Internet ; l'autre en réseau interne, reliée à **Client**) ; elle sert aussi de _passerelle_ à **Client** pour se connecter au net
-   La machine **Client** n'a qu'une seule interface réseau, en réseau interne, reliée à **Serveur**

2. Démarrez le serveur et vérifiez que les interfaces réseau sont bien présentes. A quoi correspond l’interface appelée lo ?

lo est l'adresse de loopback qui permet de communiquer sur sa propre machine.

## Exercice 3. Installation du serveur DHCP

Un serveur DHCP permet aux ordinateurs clients d’obtenir automatiquement une configuration réseau (adresse IP, serveur DNS, passerelle par défaut…), pour une durée déterminée. Ainsi, dans notre cas, l’interfaces réseau de **client** doit être configurée automatiquement par **serveur**. Les deux machines seront sur le réseau local **tpadmin.local** ayant pour adresse **192.168.100.0/24** (on aurait pu choisir une autre adresse; attention, 192.168.1.0/24 est souvent réservée, par exemple par votre FAI).

1. Sur le serveur, installez le paquet **isc-dhcp-server**. La commande **systemctl status isc-dhcp-server** devrait vous indiquer que le serveur n’a pas réussi à démarrer, ce qui est normal puisqu’il n’est pas encore configuré (en particulier, il n’a pas encore d’adresses IP à distribuer). 

commande: 
>sudo apt install isc-dhcp-server

Ici, on voit bien que la config n'est pas encore faite:
```bash
08:18 - arthur@server:~$ systemctl status isc-dhcp-server
● isc-dhcp-server.service - ISC DHCP IPv4 server
     Loaded: loaded (/lib/systemd/system/isc-dhcp-server.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Tue 2020-12-01 08:18:06 UTC; 1min 7s ago
       Docs: man:dhcpd(8)
   Main PID: 2519 (code=exited, status=1/FAILURE)

Dec 01 08:18:06 server dhcpd[2519]:
Dec 01 08:18:06 server dhcpd[2519]: If you think you have received this message due to a bug rather
Dec 01 08:18:06 server dhcpd[2519]: than a configuration issue please read the section on submitting
Dec 01 08:18:06 server dhcpd[2519]: bugs on either our web page at www.isc.org or in the README file
Dec 01 08:18:06 server dhcpd[2519]: before submitting a bug.  These pages explain the proper
Dec 01 08:18:06 server dhcpd[2519]: process and the information we find helpful for debugging.
Dec 01 08:18:06 server dhcpd[2519]:
Dec 01 08:18:06 server dhcpd[2519]: exiting.
Dec 01 08:18:06 server systemd[1]: isc-dhcp-server.service: Main process exited, code=exited, status=1/FAILURE
Dec 01 08:18:06 server systemd[1]: isc-dhcp-server.service: Failed with result 'exit-code'.
```

2. Un serveur DHCP a besoin d’une IP statique. Attribuez de manière **permanente** l’adresse IP 192.168.100.1 à l’interface réseau du réseau interne. Vérifiez que la configuration est correcte. 

Commande:
>cd /run/systemd/network
>sudo netplan generate
>sudo nano /etc/netplan/*.yaml

Dans le fichier .yaml mettre:
```bash
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8 :
      addresses :
        - 192.168.100.1/24
  version: 2
```

Et ensuite pour appliquer et vérifier tout ça:
```bash
09:07 - arthur@server:/run/systemd/network$ sudo netplan apply
09:07 - arthur@server:/run/systemd/network$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:0d:68:c1 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86397sec preferred_lft 86397sec
    inet6 fe80::a00:27ff:fe0d:68c1/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b9:24:f0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb9:24f0/64 scope link
       valid_lft forever preferred_lft forever
```

et on voit bien que l'IP est setup pour enp0s8.


3. La configuration du serveur DHCP se fait via le fichier **/etc/dhcp/dhcpd.conf**. Faites une sauvegarde du fichier existant sous le nom **dhcpd.conf.bak** puis éditez le fichier **dhcpd.conf** avec les informations suivantes : 
```bash
default-lease-time 120; 
max-lease-time 600; 
authoritative; #DHCP officiel pour notre réseau 
option broadcast-address 192.168.100.255; #informe les clients de l'adresse de broadcast 
option domain-name "tpadmin.local"; #tous les hôtes qui se connectent au #réseau auront ce nom de domaine 

subnet 192.168.100.0 netmask 255.255.255.0 { #configuration du sous-réseau 192.168.100.0  
	range 192.168.100.100 192.168.100.240; #pool d'adresses IP attribuables 
	option routers 192.168.100.1; #le serveur sert de passerelle par défaut 
	option domain-name-servers 192.168.100.1; #le serveur sert aussi de serveur DNS 
	} 
```
Commandes:
>sudo cp dhcpd.conf dhcpd.conf.bak


A quoi correspondent les deux premières lignes ? 

Les premières ligne spécifie en seconde le temps pendant une adresse ip peut être attribué via une demande au dhcp. Elles sont très courtes!

** Les valeurs indiquées sur ces deux lignes sont faibles, afin que l’on puisse voir constituer quelques logs durant ce TP. Dans un environnement de production, elles sont beaucoup plus élevées! **

4. Editez le fichier /etc/default/isc-dhcp-server afin de spécifier l’interface sur laquelle le serveur doit écouter. 

Modifier le fichier pour qu'il ressemble à ça:
```bash
# Defaults for isc-dhcp-server (sourced by /etc/init.d/isc-dhcp-server)

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
#DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPDv4_PID=/var/run/dhcpd.pid
#DHCPDv6_PID=/var/run/dhcpd6.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="enp0s8"
INTERFACESv6=""
```

5. Validez votre fichier de configuration avec la commande dhcpd -t puis redémarrez le serveur DHCP (avec la commande systemctl restart isc-dhcp-server) et vérifiez qu’il est actif. 

```bash
09:27 - arthur@server:/etc/default$ dhcpd -t
Internet Systems Consortium DHCP Server 4.4.1
Copyright 2004-2018 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/
Config file: /etc/dhcp/dhcpd.conf
Database file: /var/lib/dhcp/dhcpd.leases
PID file: /var/run/dhcpd.pid
```
Commande pour redémarrer dhcp:
```bash
09:27 - arthur@server:/etc/default$ systemctl restart isc-dhcp-server
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to restart 'isc-dhcp-server.service'.
Authenticating as: arthur
Password:
==== AUTHENTICATION COMPLETE ===
```

Pour vérifier que le dhcp est bien actif:
```bash
09:29 - arthur@server:~$ systemctl status isc-dhcp-server
● isc-dhcp-server.service - ISC DHCP IPv4 server
     Loaded: loaded (/lib/systemd/system/isc-dhcp-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-12-01 09:29:08 UTC; 30s ago
       Docs: man:dhcpd(8)
   Main PID: 3073 (dhcpd)
      Tasks: 4 (limit: 1074)
     Memory: 4.5M
     CGroup: /system.slice/isc-dhcp-server.service
             └─3073 dhcpd -user dhcpd -group dhcpd -f -4 -pf /run/dhcp-server/dhcpd.pid -cf /etc/dhcp/dhcpd.conf

Dec 01 09:29:08 server dhcpd[3073]: Sending on   LPF/enp0s8/08:00:27:b9:24:f0/192.168.100.0/24
Dec 01 09:29:08 server dhcpd[3073]:
Dec 01 09:29:08 server dhcpd[3073]: No subnet declaration for enp0s3 (10.0.2.15).
Dec 01 09:29:08 server dhcpd[3073]: ** Ignoring requests on enp0s3.  If this is not what
Dec 01 09:29:08 server dhcpd[3073]:    you want, please write a subnet declaration
Dec 01 09:29:08 server dhcpd[3073]:    in your dhcpd.conf file for the network segment
Dec 01 09:29:08 server dhcpd[3073]:    to which interface enp0s3 is attached. **
Dec 01 09:29:08 server dhcpd[3073]:
Dec 01 09:29:08 server dhcpd[3073]: Sending on   Socket/fallback/fallback-net
Dec 01 09:29:08 server dhcpd[3073]: Server starting service.
```


6. Passons au client. Si vous avez suivi le sujet du TP 1, le client a été créé en clonant la machine virtuelle du serveur. Par conséquent, son nom d’hôte est toujours serveur. Nous allons remédier à cela. Pour l’instant, vérifiez que la carte réseau du client est désactivée, puis démarrez le client. Pour modifier le nom de la machine, saisissez la commande hostnamectl set-hostname client. 

> Dans les versions récentes, Ubuntu installe d’office le paquet cloud-init lors de la configuration du système. Ce paquet permet la configuration de machines via un script dans le cloud, et a parfois des effets de bord fâcheux; en particulier, il supprimera le nom qu’on vient de donner à notre VM au prochain redémarrage pour lui redonner son ancien nom. Pour éviter cela, créez le fichier /etc/cloud/cloud.cfg.d/99_hostname.cfg dans lequel vous ajouterez simplement preserve_hostname: true. 

7. La commande **tail -f /var/log/syslog** affiche de manière continue les dernières lignes du fichier de log du système (dès qu’une nouvelle ligne est écrite à la fin du fichier, elle est affichée à l’écran). Lancez cette commande sur le **serveur**, puis activez la carte réseau du **client** et observez les logs sur le serveur. Expliquez à quoi correspondent les messages DHCPDISCOVER, DHCPOFFER, DHCPREQUEST, DHCPACK. Vérifiez que le client reçoit bien une adresse IP de la plage spécifiée précédemment. 

```bash
09:29 - arthur@server:~$ tail -f /var/log/syslog
Dec  1 09:29:08 server dhcpd[3073]:    you want, please write a subnet declaration
Dec  1 09:29:08 server dhcpd[3073]:    in your dhcpd.conf file for the network segment
Dec  1 09:29:08 server dhcpd[3073]:    to which interface enp0s3 is attached. **
Dec  1 09:29:08 server dhcpd[3073]:
Dec  1 09:29:08 server dhcpd[3073]: Sending on   Socket/fallback/fallback-net
Dec  1 09:29:08 server dhcpd[3073]: Server starting service.
Dec  1 09:36:09 server dhcpd[3073]: DHCPDISCOVER from 08:00:27:e8:81:49 via enp0s8
Dec  1 09:36:10 server dhcpd[3073]: DHCPOFFER on 192.168.100.100 to 08:00:27:e8:81:49 (arthurclient) via enp0s8
Dec  1 09:36:10 server dhcpd[3073]: DHCPREQUEST for 192.168.100.100 (192.168.100.1) from 08:00:27:e8:81:49 (arthurclient) via enp0s8
Dec  1 09:36:10 server dhcpd[3073]: DHCPACK on 192.168.100.100 to 08:00:27:e8:81:49 (arthurclient) via enp0s8
```

On observe bien les message de client en discussion avec dhcp. 
DHCPDISCOVER, première demande du client au serveur dhcp pour avoir une adresse
DHCPOFFER, le serveur dhcp propose une adresse ip a l'adresse mac qui demande.
DHCPREQUEST, le client sélectionne l'adresse IP proposé et demande son utilisation
DHCPACK, le serveur dhcp accuse de réception et accorde l'utilisation

8. Que contient le fichier **/var/lib/dhcp/dhcpd.leases** sur le serveur, et qu’affiche la commande **dhcp-lease-list** ? 

Voici ce que contient le fichier **/var/lib/dhcp/dhcpd.leases**:
Il liste les demandes de bail dhcp qu'il reçoit.
```
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\001'X\311d\010\000'\271$\360";

lease 192.168.100.100 {
  starts 2 2020/12/01 09:36:10;
  ends 2 2020/12/01 09:41:10;
  cltt 2 2020/12/01 09:36:10;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:e8:81:49;
  uid "\377\3424?>\000\002\000\000\253\021\357\314U~\236\340\334\221";
  client-hostname "arthurclient";
}
```
**dhcp-lease-list** affiche les interfaces qui ont demandé une adresse via le serveur dhcp et qui ont actuellement une adresse attribuée:
```
10:02 - arthur@server:~$ dhcp-lease-list
To get manufacturer names please download http://standards.ieee.org/regauth/oui/oui.txt to /usr/local/etc/oui.txt
Reading leases from /var/lib/dhcp/dhcpd.leases
MAC                IP              hostname       valid until         manufacturer
===============================================================================================
08:00:27:e8:81:49  192.168.100.100 arthurclient   2020-12-01 10:02:28 -NA-
```

9. Vérifiez que les deux machines se « voient » via leur adresse IP, à l’aide de la commande ping. 

```bash
10:02 - arthur@server:~$ ping 192.168.100.100
PING 192.168.100.100 (192.168.100.100) 56(84) bytes of data.
64 bytes from 192.168.100.100: icmp_seq=1 ttl=64 time=0.338 ms
64 bytes from 192.168.100.100: icmp_seq=2 ttl=64 time=0.453 ms
64 bytes from 192.168.100.100: icmp_seq=3 ttl=64 time=0.441 ms
64 bytes from 192.168.100.100: icmp_seq=4 ttl=64 time=0.391 ms
64 bytes from 192.168.100.100: icmp_seq=5 ttl=64 time=0.403 ms
64 bytes from 192.168.100.100: icmp_seq=6 ttl=64 time=0.377 ms
--- 192.168.100.100 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5081ms
rtt min/avg/max/mdev = 0.338/0.400/0.453/0.038 ms
```


10. Modifiez la configuration du serveur pour que l’interface réseau du client reçoive l’IP statique 192.168.100.20 : 
```bash
deny unknown-clients; #empêche l'attribution d'une adresse IP à une 
#station dont l'adresse MAC est inconnue du serveur 
host client1 { 
	hardware ethernet XX:XX:XX:XX:XX:XX; #remplacer par l'adresse MAC 
	fixed-address 192.168.100.20; 
	} 
```
Vérifiez que la nouvelle configuration a bien été appliquée sur le client (éventuellement, désactivez puis réactivez l’interface réseau pour forcer le renouvellement du bail DHCP, ou utilisez la commande dhclient -v).

Rajouter la config ci-dessus dans le fichier **/etc/dhcp/dhcpd.conf**

puis relancer les requêtes dhcp avec la commande: **sudo dhclient -v"**
Attention il peut y avoir des bugs (corrupt lease file par exemple)
ne pas trop se fier a **dhcp-lease-list** aussi, il bug a des moment. 

Faire une vérif sur le client avec ip a est plus fiable.

## Exercice 4. Donner un accès à Internet au client

A ce stade, le client est juste une machine sur notre réseau local, et n’a aucun accès à Internet. Pour remédier à cette situation, on va se servir de la machine serveur (qui, elle, a un accès à Internet via son autre carte réseau) comme d’une passerelle. 

1. La première chose à faire est d’autoriser l’IP forwarding sur le serveur (désactivé par défaut, étant donné que la plupart des utilisateurs n’en ont pas besoin). 
Pour cela, il suffit de décommenter la ligne **net.ipv4.ip_forward=1** dans le fichier **/etc/sysctl.conf**. Pour que les changements soient pris en compte immédiatement, il faut saisir la commande **sudo sysctl -p /etc/sysctl.conf**.
**  Vérifiez avec la commande sysctl net.ipv4.ip_forward que la nouvelle valeur a bien été prise en compte. **

2. Ensuite, il faut autoriser la traduction d’adresse source (masquerading) en ajoutant la règle iptables suivante : 
**sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERADE** 
Vérifiez à présent que vous arrivez à « pinguer » une adresse IP (par exemple 1.1.1.1 depuis le client. A ce stade, le client a désormais accès à Internet, mais il sera difficile de surfer : par exemple, il est même impossible de pinguer www.google.com. C’est parce que nous n’avons pas encore configuré de serveur DNS pour le client.
```bash
10:51 - arthur@server:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=55 time=10.7 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=55 time=9.83 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=55 time=9.45 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=55 time=9.55 ms
--- 1.1.1.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 9.449/9.879/10.691/0.489 ms
```


## Exercice 5. Installation du serveur DNS

De la même façon qu’il est plus facile de retenir le nom d’un contact plutôt que son numéro de téléphone, il est plus simple de mémoriser le nom d’un hôte sur un réseau (par exemple www.cpe.fr) plutôt que son adresse IP (178.237.111.223). 

Dans les premiers réseaux, cette correspondance, appelée résolution de nom, se faisait via un fichier nommé hosts (présent dans /etc sous Linux 1 ). L’inconvénient de cette méthode est que lorsqu’un nom ou une adresse IP change, il faut modifier les fichiers hosts de toutes les machines! 

Par conséquent, avec l’avénement des réseaux à grande échelle, ce système n’était plus viable, et une autre solution, automatisée et centralisée cette fois, a été mise au point : DNS (Domain Name Server). Généralement, le serveur DNS utilisé est soit celui mis à disposition par le fournisseur d’accès à Internet, soit un DNS public (comme celui de Google : 8.8.8.8, ou celui de Cloudflare : 1.1.1.1). 

Il est aussi très commun d’utiliser un serveur DNS privé, interne à l’organisation, afin de pouvoir résoudre les noms des machines locales. Pour les requêtes extérieures, le serveur DNS privé passe alors la main à un DNS externe. 

Il existe de nombreux serveurs DNS, mais le plus commun sous UNIX est Bind9 (Berkeley Internet Name Daemon v.9). 

1. Sur le serveur, commencez par installer Bind9, puis assurez-vous que le service est bien actif. 

Commande:
>sudo apt install bind9

2. A ce stade, Bind n’est pas configuré et ne fait donc pas grand chose. L’une des manières les simples de le configurer est d’en faire un serveur cache : il ne fait rien à part mettre en cache les réponses de serveurs externes à qui il transmet la requête de résolution de nom. 

> Le binaire (= programme) installé avec le paquet bind9 ne s’appelle ni bind ni bind9 mais named... 

Nous allons donc modifier son fichier de configuration : **/etc/bind/named.conf.options**. 
Dans ce fichier, décommentez la partie forwarders, et à la place de 0.0.0.0, renseignez les IP de DNS publics comme 1.1.1.1 et 8.8.8.8 (en terminant à chaque fois par un point virgule). Redémarrez le serveur bind9. 

Commande:
>sudo service bind9 restart

4. Sur le client, retentez un ping sur www.google.fr. Cette fois ça devrait marcher! On valide ainsi la configuration du DHCP effectuée précédemment, puisque c’est grâce à elle que le client a trouvé son serveur DNS. 4. Sur le client, installez le navigateur en mode texte lynx et essayez de surfer sur fr.wikipedia.org (bienvenue dans le passé...)

## Exercice 6. Configuration du serveur DNS pour la zone tpadmin.local

L’intérêt d’un serveur DNS privé est principalement de pouvoir résoudre les noms des machines du réseau local. Pour l’instant, il est impossible de pinguer client depuis serveur et inversement. 

1. Modifiez le fichier /etc/bind/named.conf.local et ajoutez les lignes suivantes : 
```bash
zone "tpadmin.local" { 
type master; // c'est un serveur maître 
file "/etc/bind/db.tpadmin.local"; // lien vers le fichier de définition de zone }; 
```
2. Créez une copie appelée db.tpadmin.local du fichier db.local. Ce fichier est un fichier configuration typique de DNS, constitué d’enregistrements DNS (cf. cours). Commencez par remplacer localhost par tpadmin.local, et l’adresse 127.0.0.1 par l’adresse IP du serveur. 

> La ligne root.tpadmin.local. indique en fait une adresse mail du responsable technique de cette zone, où le symbole @ est remplacé par un point. Attention également à ne pas oublier le point final, qui représente la racine DNS ; on ne le met pas dans les navigateurs, mais il est indispensable dans les fichiers de configuration DNS ! 

> Le champ serial doit être incrémenté à chaque modification du fichier. Généralement, on lui donne pour valeur la date suivie d’un numéro sur deux chiffres, par exemple 2019031401. 

3. Maintenant que nous avons configuré notre fichier de zone, il reste à configurer le fichier de zone inverse, qui permet de convertir une adresse IP en nom. 
Commencez par rajouter les lignes suivantes à la fin du fichier named.conf.local : 
```bash
zone "100.168.192.in-addr.arpa" { 
type master; 
file "/etc/bind/db.192.168.100"; 
}; 
```
Créez ensuite le fichier db.192.168.100 à partir du fichier db.127, et modifiez le de la même manière que le fichier de zone. Sur la dernière ligne, faites correspondre l’adresse IP avec celle du serveur (Attention, il y a un petit piège !). 

Juste mettre le 1 sur la dernière ligne

4. Utilisez les utilitaires named-checkconf et named-checkzone pour valider vos fichiers de configuration :
```bash
$ named-checkconf named.conf.local 
$ named-checkzone tpadmin.local /etc/bind/db.tpadmin.local 
$ named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100 
```

6. Redémarrer le serveur Bind9. Vous devriez maintenant être en mesure de ”pinguer” les différentes machines du réseau

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjE4NDE0MTE0LDE0NTM3NDkwMDNdfQ==
-->