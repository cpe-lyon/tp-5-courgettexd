## TP5 Service réseau

### Exercice 2 : Préparation de l'environnement

Commandes : 

* *ip a* => affiche l'ensemble des modules et addresses ip associés présents sur la machine
* *ifconfig* => équivalent de *ip a* (nécessite paquet spécifique **net-tools**) 

Le résultat affichée par cette même commande est :

1. *lo* => loopback => interface virtuel connecté au système afin de pouvoir réaliser un ensemble de test de connectivité utile

2. *enp0s3* => interface physique (carte ethernet) présente sur la machine

### Exercice 3 : Installation du serveur DHCP

Commandes :

* *sudo apt install isc-dhcp-server*  => installer le paquet pour le serveur DHCP
* *systemctl status isc-dhcp-server*  => observer le statut du serveur DHCP
* *sudo ip addr add @IP dev moninterface*  => ajoute une adresse ip à l'interface désigné
* *sudo ip addr flush moninterface* => efface l'adresse ip d'une interface

La configuration d'un serveur DHCP se fait au travers du fichier **/etc/dhcp/dhcpd.conf**.

Exemple de configuration :

    default-lease-time 120; #lease-time = temps qu'une machine en réseau 
                            #peut utiliser une IP du réseau
    max-lease-time 600;     #max= valeur max de temps que la machine gardera l'adresse IP
                            #ici 10 min max (600 secondes = 10 mins)
    authoritative;#DHCP officiel pour notre réseau
    option broadcast-address 192.168.100.255;          #informe les clients de l'adresse de 
                                                       #broadcast
    option domain-name "tpadmin.local";                #tous les hôtes qui se connectent au
                                                       #réseau auront ce nom de domaine
    subnet 192.168.100.0 netmask 255.255.255.0 {      #configuration du sous-réseau 192.168.100.0
    range 192.168.100.100 192.168.100.240;            #pool d'adresses IP attribuables
    option routers 192.168.100.1;                     #le serveur sert de passerelle par défaut
    option domain-name-servers 192.168.100.1;         #le serveur sert aussi de serveur DNS
    }

**NB** : Les valeurs indiquées sur les deux premières lignes sont faibles, afin que l’on puisse voir constituer quelqueslogs durant ce TP. Dans un environnement de production, elles sont beaucoup plus élevées!

Il faut par la suite éditer le fichier/etc/default/isc-dhcp-serverafin de spécifier l’interface sur laquelle le serveur doit écouter.

Pour valider la configuration, il faut utiliser la commande *dhcpd -t*. Par la suite, il faut redémarrer le serveur avec la commande *systemctl restart isc-dhcp-server*.

Commande :

* *hostnamectl set-hostname client* => modifie le nom de la machine pour "client"

**NB** (extrait du sujet) : Dans les versions récentes, Ubuntu installe d’oﬀice le paquet *cloud-init* lors de la configurationdu système. Ce paquet permet la configuration de machines via un script dans le cloud, et a parfois des effets de bord fâcheux; en particulier, il supprimera le nom qu’on vient de donner à notre **VM** au prochain redémarrage pour lui redonner son ancien nom. Pour éviter cela, créez le fichier */etc/cloud/cloud.cfg.d/99_hostname.cfg* dans lequel vous ajouterez simplement **preserve_hostname:true**

* *tail -f /var/log/syslog* => aﬀiche de manière continue les dernières lignes du fichier de log du système

7- Messages :

* *DHCPDISCOVER* : localise les serveurs DHCP disponibles et demande une 1ere configuration
* *DHCPOFFER* : en réponse à *DHCPDISCOVER* qui contient le 1er paramètre de configuration
=> offre envoyer au client pour donné une IP (sur port 68) identifié par MAC addresse, IP serveur, IP/mask.
* *DHCPREQUEST* : diffuser par le client. Comporte IP serveur et celle donnée par le client
=> demande au serveur choisi, l'assignation de l'addresse, des paramètres et d'informer les autres serveurs de l'acceptation
* *DHCPPACK* :
=> donne accusé de réception
=> assigne au client l'IP et mask
    => + durée du bail de l'adresse IP
    => + IP passerelle par défaut
    => + IP serveur DNS
    => + IP NBNS serveur

8- Notes :

* dhcp.leases : base de données persistante des données persisitnate des concessions attribuées
* *dhcp-lease-list* : affiche les DHCP leases actives
* *ping* : vérification de la connexion entre deux machines

10- Notes :

Modifiez la configuration du serveur pour que l’interface réseau du client reçoive l’IP statique : 

        192.168.100.20 :deny unknown-clients;     #empêche l'attribution d'une adresse IP à une
                                                  #station dont l'adresse MAC est inconnue du serveur

        host client1 {hardware ethernet XX:XX:XX:XX:XX:XX;   #remplacer par l'adresse MAC 
                                                             #de la machine client
        fixed-address 192.168.100.20;}
        
 **NB** :  pour forcer le renouvellement du bail DHCP, ou utilisez la commande *dhclient -v*

### Exercice 4 : Donner un accès à Internet au client

**Extrait du sujet** :

1-La première chose à faire est d’autoriser **l’IP  forwarding** sur le serveur (désactivé par défaut, étantdonné que la plupart des utilisateurs n’en ont pas besoin). Pour cela, il suﬀit de décommenter la ligne **net.ipv4.ip_forward=1** dans le fichier **/etc/sysctl.conf**. Pour que les changements soient pris encompte immédiatement, il faut saisir la commande *sudo sysctl -p /etc/sysctl.conf*.

Vérifiez avec la commande *sysctl net.ipv4.ip_forward* que la nouvelle valeur a bien été prise en compte.

2-Ensuite, il faut autoriser la traduction d’adresse source (masquerading) en ajoutant la règle iptables suivante : *sudo iptables --table nat --append POSTROUTING --out-interface enp0s3 -j MASQUERAD*

Le client à dorénavant accès à internet.

### Exercice 5 : Installation du serveur DNS

**Extrait du sujet** :

Il est plus simple de mémoriser le nom d’un hôte sur un réseau (par exemple www.cpe.fr) plutôt que son adresse IP (178.237.111.223).

Dans les premiers réseaux, cette correspondance, appelée résolution de nom, se faisait via un fichier nommé **hosts** (présent dans /etc sous Linux ). L’inconvénient de cette méthode est que lorsqu’un nom ou une adresse IP change, il faut modifier les fichiers hosts de toutes les machines!

Par conséquent, avec l’avénement des réseaux à grande échelle, ce système n’était plus viable, et une autre solution, automatisée et centralisée cette fois, a été mise au point : **DNS (Domain Name Server)**. Généralement, le serveur DNS utilisé est soit celui mis à disposition par le fournisseur d’accès à Internet, soit un DNS public (comme celui de Google : 8.8.8.8, ou celui de Cloudflare : 1.1.1.1).

Il existe de nombreux serveurs DNS, mais le plus commun sous UNIX est Bind9 (Berkeley Internet Name Daemon v.9).

1.	Sur le serveur, commencez par installer Bind9, puis assurez-vous que le service est bien actif.

Commande :

* *service bind9 start/stop/restart* => start, stop ou restart **bind9**

2.	A ce stade, Bind n’est pas configuré et ne fait donc pas grand chose. L’une des manières les simples de le configurer est d’en faire un serveur cache : il ne fait rien à part mettre en cache les réponses de serveurs externes à qui il transmet la requête de résolution de nom.

Le binaire (= programme) installé avec le paquet bind9 ne s’appelle ni bind ni bind9 mais **named**

Nous allons donc modifier son fichier de configuration : **/etc/bind/named.conf.options**.

Dans ce fichier, décommentez la partie forwarders, et à la place de 0.0.0.0, renseignez les IP de DNS publics comme 1.1.1.1 et 8.8.8.8 (en terminant à chaque fois par un point virgule). Redémarrez le serveur bind9.

3.	Sur le client, installez le navigateur en mode texte **lynx** et essayez de surfer sur fr.wikipedia.org (sudo apt-get install lynx)

### Exercice 6 : Configuration du serveur DNS pour la zone tpadmin.local

**Extrait du sujet** :

L’intérêt d’un serveur DNS privé est principalement de pouvoir résoudre les noms des machines du réseau local. Pour l’instant, il est impossible de pinguer client depuis serveur et inversement.

1.	Modifiez le fichier /etc/bind/named.conf.local et ajoutez les lignes suivantes :

        zone "tpadmin.local" {
            type master;	// c'est un serveur maître
        file "/etc/bind/db.tpadmin.local"; // lien vers le fichier de définition de zone };

2.Ce fichier (**db.tpadmin.local**) est un fichier configuration typique de DNS, constitué d’enregistrements DNS (cf. cours). Commencez par remplacer **localhost** par **tpadmin.local**, et l’adresse 127.0.0.1 par l’adresse IP du serveur.

La ligne root.tpadmin.local. indique en fait une adresse mail du responsable technique de cette zone, où le symbole @ est remplacé par un point. Attention également à ne pas oublier le point final, qui représente la racine DNS; on ne le met pas dans les navigateurs, mais il est indispensable dans les fichiers de configuration DNS!

Le champ serial doit être incrémenté à chaque modification du fichier. Généralement, on lui donne pour valeur la date suivie d’un numéro sur deux chiffres, par exemple 2019031401.

3.	Maintenant que nous avons configuré notre fichier de zone, il reste à configurer le fichier de zone inverse, qui permet de convertir une adresse IP en nom.

Commencez par rajouter les lignes suivantes à la fin du fichier **named.conf.local** :

            zone "100.168.192.in-addr.arpa" { type master;
            file "/etc/bind/db.192.168.100";
            };
            
Créez ensuite le fichier db.192.168.100 à partir du fichier db.127, et modifiez le de la même manière que le fichier de zone. Sur la dernière ligne, faites correspondre l’adresse IP avec celle du serveur (Attention, il y a un petit piège!).

4.	Utilisez les utilitaires **named-checkconf** et named-checkzone pour valider vos fichiers de configuration :

        $ named-checkconf named.conf.local
        $ named-checkzone tpadmin.local /etc/bind/db.tpadmin.local
        $ named-checkzone 100.168.192.in-addr.arpa /etc/bind/db.192.168.100

5.	Redémarrer le serveur Bind9. Vous devriez maintenant être en mesure de ”pinguer” les différentes machines du réseau.





   





