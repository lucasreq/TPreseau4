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

**Installation et configuration de la VM "patron"** :
* créer une VM
  * 512 Mo RAM
  * 1 CPU
  * Réseau
    * une carte NAT
  * Stockage
    * disque de 8Go 
    * `.iso` de CentOS 7 (sur le "contrôleur IDE")
* installation
  * se référer au [TP précédent](../3/README.md#i-création-et-utilisation-simples-dune-vm-centos) (n'hésitez pas à m'appeler en cas de doute)
* wait for installation process to finish
* redémarrer la VM
  * vous pouvez enlever le `.iso` du lecteur CD si ce n'est pas fait automatiquement :)
* configuration VM
  * se logger avec votre utilisateur
  * exécutez :
```bash
# Désactivation de SELinux
sudo setenforce 0 # temporaire
sudo sed -i 's/enforcing/permissive/g' /etc/selinux/config # permanent

# Mise à jour des dépôts
sudo yum update -y

# Installation de dépôts additionels
sudo yum install -y epel-release

# Installation de plusieurs paquets réseau dont on se sert souvent
sudo yum install -y traceroute bind-utils tcpdump nc nano

# Désactivation de la carte NAT au reboot
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
# mettre ONBOOT à NO

# Eteindre la machine
sudo shutdown now
```

# I. Mise en place du lab

Le "lab", c'est juste l'environnement nécessaire à notre TP. Vous allez créer des VMs quoi !

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
