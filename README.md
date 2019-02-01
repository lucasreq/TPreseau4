# TPreseau4

# TP 4 - Spéléologie réseau : descente dans les couches

# Sommaire
* [Préparation d'une VM "patron"](#préparation-dune-vm-patron)
* I. [Mise en place du lab](#i-mise-en-place-du-lab)
* II. [Spéléologie Réseau](#ii-spéléologie-réseau)
  * 1. [ARP](#1-arp)
  * 2. [Interception de trafic avec Wireshark](#2-wireshark)
    * [ARP et `ping`](#a-interception-darp-et-ping)
    * [netcat](#b-interception-dune-communication-netcat)
    * [Trafic Web (HTTP)](#c-interception-dun-trafic-http)
* [Annexe 1 : Installation d'une interface graphique sur CentOS 7](#annexe-1--installation-dun-client-graphique)

# I. Mise en place du lab
## 1. Création des réseaux

On va créer de nouveaux réseaux host-only. Pour rappel, la création d'un réseau host-only **ajoute une [carte réseau](../../cours/lexique.md#carte-réseau-ou-interface-réseau) sur votre PC**. Vous pouvez la voir avec un `ipconfig` bien sûr ! 

Créez les réseaux **host-only** suivants :
* le "réseau 1" ou `net1` : `10.1.0.0/24`
  * la carte réseau de l'hôte doit porter l'IP `10.1.0.1`
  * **PAS** de DHCP
* le "réseau 2" ou `net2` : `10.2.0.0/24`
  * la carte réseau de l'hôte doit porter l'IP `10.2.0.1`
  * **PAS** de DHCP

## 2. Création des VMs

**NB : Quand vous clonez, Virtualbox va vous poser des questions :**
* **clone intégral**
* **réinitialisation des adresses MAC : OUI**

**NB2 : PAS DE NAT DANS LES CLONES (ou alors vous la désactivez)**. Une interface de type NAT dans VirtualBox sert à accéder à internet. On en a **PAS** besoin (vous avez déjà fait les `yum install` dans le patron).

Créez les VMs suivantes (= clonez votre VM patron !) :
* **VM cliente** ou [`client1.tp4`](../../cours/procedures.md##changer-son-nom-de-domaine)
  * elle a une carte réseau dans `net1` (host-only) qui porte l'IP `10.1.0.10`
  * elle nous servira... de [client](../../cours/3.md#clientserveur) !
* **VM serveur** ou [`server1.tp4`](../../cours/procedures.md##changer-son-nom-de-domaine)
  * elle a une carte réseau dans `net2` (host-only) qui porte l'IP `10.2.0.10`
  * elle nous servira de [serveur](../../cours/3.md#clientserveur) :|
* **VM routeur** ou [`router1.tp4`](../../cours/procedures.md##changer-son-nom-de-domaine)
  * elle a une carte réseau dans `net1` (host-only) qui porte l'IP `10.1.0.254`
  * et une carte réseau dans `net2` (host-only) qui porte l'IP `10.2.0.254`
  * cette machine sera notre [routeur](../../cours/lexique.md#routeur). Ce sera la [passerelle](../../cours/lexique.md#passerelle-ou-gateway) de `client1` et `server1`

Je pense que vous avez compris le principe. Au cas où, je vous fais un schéma moche !
```
client  <--net1--> router <--net2--> server
```

---

**Checklist (à faire sur toutes les machines)** :
* [X] Désactiver SELinux
  * déja fait dans le patron
* [X] Installation de certains paquets réseau
  * déja fait dans le patron
* [X] **Désactivation de la carte NAT**
  * déja fait dans le patron
* [x] [Définition des IPs statiques](../../cours/procedures.md#définir-une-ip-statique)
* [x] La connexion SSH doit être fonctionnelle
  * une fois fait, vous avez vos trois fenêtres SSH ouvertes, une dans chaque machine
* [x] [Définition du nom de domaine](../../cours/procedures.md##changer-son-nom-de-domaine)
* [x] [Remplissage du fichier `/etc/hosts`](../../cours/procedures.md#editer-le-fichier-hosts)
* [x] `client1` ping `router1.tp4` sur l'IP `10.1.0.254`
* [x] `server1` ping `router1.tp4` sur l'IP `10.2.0.254`

##Definition IPs statiques

Définir une IP statique
**1. Repérer le nom de l'interface dont on veut changer l'IP**
```
ip a
```
**2. Modifier le fichier correspondant à l'interface**
* il se trouve dans `/etc/sysconfig/network-scripts`
* il porte le nom `ifcfg-<NOM_DE_L'INTERFACE>`
* on peut le créer s'il n'existe pas
* exemple de fichier minimaliste qui assigne `192.168.1.19/24` à l'interface `enp0s8`
  * c'est donc le fichier `/etc/sysconfig/network-scripts/ifcfg-enp0s8`
```
NAME=enp0s8
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=192.168.1.19
NETMASK=255.255.255.0
```
**3. Redémarrer l'interface**
```
sudo ifdown <INTERFACE_NAME>
sudo ifup <INTERFACE_NAME>
```

##La connection ssh doit être fonctionnelle

![Image connection ssh fonctionnelle](https://github.com/lucasreq/TPreseau4/blob/master/images/tp4_connection.JPG)

##Définition du nom de domaine

**1. Changer le FQDN immédiatement** (temporaire)
```
# commande hostname
sudo hostname <NEW_HOSTNAME>
# par exemple
sudo hostname vm1.tp3.b1
```
**2. Définir un FQDN quand la machine s'allume** (permanent)
* écriture du FQDN dans le fichier (avec `nano`) : `sudo nano etc/hostname`
* **OU** en une seule commande `echo 'vm1.tp1.b3' | sudo tee /etc/hostname`

**3. Pour consulter votre FQDN actuel**
```
hostname --fqdn
```
![nom de domaine](https://github.com/lucasreq/TPreseau4/blob/master/images/Nom_de_domaine.JPG)







