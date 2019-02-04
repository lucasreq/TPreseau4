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

## Definition IPs statiques

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

## La connection ssh doit être fonctionnelle


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

# II. Spéléologie réseau

## 1. ARP
### **A. Manip 1**

* [ ] 1. vider la table ARP de **toutes** vos machines ```ip neigh flush all```

* [ ] 2. sur `client1`
    * vous DEVEZ utiliser SSH
    * afficher la table ARP ```ip neigh```
    * **expliquer la seule ligne visible** (hint : vous êtes connecté à cette VM, non ?... ;) )
    [Manip 1](https://github.com/lucasreq/TPreseau4/blob/master/images/Manip1_ARPclient1.JPG)
    Cette ligne représente, l'adresse MAC de la carte réseau du client.

* [ ] 3. sur `server1`
    * vous DEVEZ utiliser SSH
    * afficher la table ARP ```ip neigh```
    * **expliquer la seule ligne visible** (hint : celle-la aussi non ?)
    [Manip 2](https://github.com/lucasreq/TPreseau4/blob/master/images/Manip1_ARPserv1.JPG)
    Cette ligne représente, l'adresse MAC de la carte réseau du client.


* [ ] 4. sur `client1`
    * ping `server1`
    * afficher la table ARP ```ip neigh```
    * **expliquer le changement**
* [ ] 5. sur `server1`
    * afficher la table ARP ```ip neigh```
    * **expliquer le changement**

### **B. Manip 2**

* [ ] 1. vider la table ARP de **toutes** vos machines
* [ ] 2. sur `router1`
    * afficher la table ARP
    * **expliquer le(s) ligne(s)**
* [ ] 3. sur `client1`
    * ping `server1`
* [ ] 4. sur `router1`
    * afficher la table ARP
    * **expliquer le(s) changement(s)**

### **C. Manip 3**

* [ ] 1. vider la table ARP de **toutes** vos machines
* [ ] 2. sur l'hôte (votre PC)
    * afficher la table ARP
    * vider la table ARP 
    * afficher de nouveau la table ARP
    * attendre un peu
    * afficher encore la table ARP
    * **expliquer le(s) changement(s)** (c'est lié à votre [passerelle](../../cours/lexique.md#passerelle-ou-gateway))

### **D. Manip 4**

* [ ] 1. vider la table ARP de **toutes** vos machines
* [ ] 2. sur `client1`
    * afficher la table ARP
    * activer la carte NAT
    * joindre internet (`curl google.com` par exemple)
    * afficher la table ARP
    * **expliquer le(s) changement(s)**
      * expliquer quelle machine porte l'IP qui vient de pop dans cette table ARP
    * **n'oubliez pas de re-désactiver/débrancher la carte NAT**  


## 2. Wireshark

* [ ] 1. Installer Wireshark
* [ ] 2. On va capturer le trafic qui passe par `router1` :
    * on doit dire à Wireshark d'intercepter et noter tout ce qui passe par une interface spécifique
    * actuellement, votre PC est connecté en SSH à `router1`
    * pour ce faire, vous avez choisi l'une de ses deux IPs pour vous connecter 
        * `10.1.0.254` ou `10.2.0.254`
    * vu que vous êtes connecté en SSH, vous envoyez des trames en permanence sur l'IP choisie
    * **on va donc capturer le trafic de l'interface à laquelle vous n'êtes PAS connecté** pour éviter le bruit généré par SSH
* [ ] 3. On va capturer le trafic qui passe par `router1` :
    * on doit dire à Wireshark d'intercepter et noter tout ce qui passe par une interface spécifique
    * actuellement, votre PC est connecté en SSH à `router1`
    * pour ce faire, vous avez choisi l'une de ses deux IPs pour vous connecter 
        * `10.1.0.254` ou `10.2.0.254`
    * vu que vous êtes connecté en SSH, vous envoyez des trames en permanence sur l'IP choisie
    * **on va donc capturer le trafic de l'interface à laquelle vous n'êtes PAS connecté** pour éviter le bruit généré par SSH

## A. Interception d'ARP et `ping`

* [ ] 1. sur `router1`
    * lancer Wireshark pour enregistrer le trafic qui passer par l'interface choisie et enregistrer le trafic dans un fichier `ping.pcap` :
        * `sudo tcpdump -i enp0s9 -w ping.pcap`
* [ ] 2. sur `client1`
    * vider la table ARP
    * envoyer 4 pings à `server1`
    * `ping -c 4 server1`
* [ ] 3. sur `router1`
    * quitter la capture (CTRL + C)
    * vérifier la présence du fichier `ping.pcap` avec un `ls`
    * envoyer le fichier `ping.pcap` sur votre hôte
        * si vous savez pas comment, ou si vous voulez des conseils sur des moyens rapides et/ou secure de le faire, appelez-moi !
* [ ] 4. sur l'hôte (votre PC) :
    * ouvrir le fichier `ping.pcap` dans Wireshark
    * essayez de comprendre un peu toutes les lignes (il devrait y en avoir une dizaine tout au plus !)
    * vous devriez voir :
        * la question pour connaître la MAC de la destination
            * protocole ARP
            * "Who has .... ? Tell ...." 
            * envoyée en broadcast
        * la réponse
            * protocole ARP
            * "... is at ..."
            * envoyée à celui qui a posé la question
        * les pings aller 
            * "ping !"
            * protocole ICMP
            * message `ECHO request`
        * les ping retour 
            * "pong !" 
            * protocole ICMP
            * message `ECHO reply`
    * **Important**
        * notez que ARP **n'est pas** encapsulé dans IP, c'est un paquet ARP dans une trame ethernet
        * notez que ICMP est encapsulé dans IP, c'est un datagramme ICMP, dans un paquet IP, dans une trame Ethernet !
        * **NOTEZ BIEN "QUI DISCUTE AVEC QUI" REELLEMENT** : au niveau des MAC
            * vous devriez en déduire qu'il vous manque la moitié des trames concernant cette communication
            * expliquez pourquoi 

## B. Interception d'une communication `netcat`

* [ ] 1. intercepter le trafic 
    * depuis `router1`
        * pendant que `client1` se connecte au serveur `netcat` de `server1`
        * oubliez pas d'ouvrir le port firewall sur `server1`
        * videz les tables ARP de tout le monde, comme ça on verra encore les messages ARP dans la capture
        * échangez quelques messages pour avoir de la matière à étudier :)
        * nommez la capture `netcat_ok.pcap`

* [ ] 2. Envoyez le fichier `netcat_ok.pcap` sur votre hôte, puis ouvrez le avec Wireshark. Mettez en évidence : 
    * l'établissement de la connexion TCP
        * c'est le "3-way handshake" :
        * le client envoie `SYN` : demande de synchronisation
        * le serveur répond `SYN,ACK` : il accepte la synchronisation
        * le client répond `ACK` : "ok frer, on est bien connectés, on peut échanger de la donnée maintenant !"
    * vos messages qui circulent

* [ ] 3. fermer le port firewall du `server1`
    * refaire la même chose 
        * se faire jeter la connexion parce que le port est fermé
        * intercepter le trafic dans un fichier `netcat_ko.pcap`
    * exporter le fichier sur l'hôte
        * mettre en évidence les lignes qui correspondent au firewall qui dit "nop frer"



