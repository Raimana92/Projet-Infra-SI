# Projet-Infra-SI

## Introduction : 

Tout au long de l'année, nous avons vu divers outils pour la mise en place d'infrastructure sécuriser via des systèmes de virtualisation (GNS3 ou VMWare) à travers des technologies telles que Debian, Alpine Linux, OpenWRT, Pfsense et Windows Server.
Le projet choisi a été choisi en duo, avec Raimana (qui a dû, pour raison personnelle quitter l'école) est **une architecture permettant de déployer des applications web redondées et sécurisées**. C'est un sujet en lien avec la spécialisation de l'année prochaine (développement web), il était donc judicieux de se lancer là-dedans.


## Schéma
Afin d'avoir une sécurité optimale j'ai choisi de mettre en place en serveur PfSense qui possèdera le reverse proxy, et le loadbalancer.
Deux serveurs web me permettront d'avoir des applications web redondées, liés entre eux par HeartBeat afin d'avoir de la Haute Disponibilité.
Ainsi qu'une base de données sous MariaDB.

Nous obtenons le schéma d'infrastructure suivant :  qui possèdera le reverse proxy, et le loadbalancer.
Deux serveurs web me permettront d'avoir des applications web redondées, liés entre elles par HeartBeat afin d'avoir de la Haute Disponibilité.
Ainsi qu'une base de données sous MariaDB.

Nous obtenons le schéma d'infrastructure suivant :

<img src="https://cdn.discordapp.com/attachments/522143202426224654/846506548494073927/unknown.png" width=800px>


## Outils utilisés

Voici les différents outils utilisés au cours de ce TP :

* **VMware Workstation 16 Pro** va nous permettre d'exécuter des systèmes d'exploitation(PfSense et Debian) en tant que machines virtuelles afin de simuler dans de réelles conditions ce projet.

* **Debian** est un des systèmes Linux les plus fiables et les plus stables. Il a également un avantage économique car c'est système d'exploitation open-source donc gratuit. Nos serveurs web et notre base de données seront donc mis en place sur Debian.

* Relativement pratique avec son interface graphique et sa gratuité, **Pfsense**, basé sur FreeBSD, est un routeur/pare-feu dont l'ajout de packages peut le rendre multifonctionnel. Nous utiliserons donc Pfsense, avec le package Squid pour la mise en place du reverse proxy.

* Le choix de serveur HTTP se porte vers **Nginx** car il possède une importante stabilité, de hautes performances, une faible consommation de ressources, et surtout il est gratuit.

* **MariaDB** est un système de gestion de base de données gratuit et performant basé sur MySQL. Il propose un large choix de moteurs de base de données et un optimisateur de requête SQL efficace. 

* **Wordpress** est un CMS, gratuit et libre qui permet de créer et de gérer différents types de sites internet.

* **Wekan** est un outil collaboratif de gestion de tâches, gratuit et open source.

* **HeartBeat** va nous permettre de surveiller la disponibilité des serveurs et lorsque l'un d'eux sera défaillant, les services seront automatiquement basculés sur l'autre serveur. Un programme simple et gratuit nous permettant d'avoir de la Haute Disponibilité sur nos serveurs web.


## Informations - Plan IP
Afin de préparer au mieux la mise en place et la configuration de nos machines virtuelles, il est important de schématiser à travers un tableau les différentes IP, masques de sous-réseaux et passerelles (Gateway) utilisées.

|    | pfSense WAN | pfSense LAN | Cluster HeartBeat | Serveur Web1 : Debian| Serveur Web2 : Debian| Base de donnée: Debian|
|:--:|:-------:|:---:|:---:| :--------------------:|:--------------------:|:---------------------:|
| **IP**      | WAN : 192.168.5.50 | LAN : 192.168.2.50 | 192.168.2.40 | 192.168.2.3 | 192.168.2.4 | 192.168.2.5 
| **Netmask** | 255.255.255.0 | 255.255.255.0 | 255.255.255.0 | 255.255.255.0 | 255.255.255.0 | 255.255.255.0 |
| **Gateway** | 192.168.5.2 | | 192.168.2.50 | 192.168.2.50 | 192.168.2.50 | 192.168.2.50 |


## Rules - PfSense

Afin de pouvoir accéder à l'interface et aux serveurs depuis l'extérieur (WAN = Wide Area Network) nous ajouterons des règles de redirection de port (_Firewall/NAT/Port Forward_). Ainsi nous ajouterons des règles pour le protocole SSH (port 22) et pour le protocole HTTP (port 80):

<img src="https://cdn.discordapp.com/attachments/522143202426224654/846509750770466846/unknown.png" border=2px> <br>

Ainsi pour le ssh nous entrerons la commande dans une invite de commande et pour http dans un navigateur.

* Cluster HeartBeat :
  - ssh : **user@pfSenseWANAddress -p 2222**
  - http : **pfSenseWANAddress:3222**
<br>

* Serveur Web1 :
  - ssh : **user@pfSenseWANAddress -p 2223**
  - http : **pfSenseWANAddress:3223**
<br>

* Serveur Web2 :
  - ssh : **user@pfSenseWANAddress -p 2224**
  - http : **pfSenseWANAddress:3224**
<br>

* Base de donnée:
  - ssh : **user@pfSenseWANAddress -p 2225**
<br>

* Wekan :
  - http : **pfSenseWANAddress:4222**

Ensuite, nous allons ajouter différentes règles WAN afin d'autoriser le flux des port ci dessus afin que PfSense ne bloque pas leur bloque pas l'accès:
![](https://cdn.discordapp.com/attachments/522143202426224654/846545776506830858/unknown.png)


## Reverse Proxy - Squid

Un reverse-proxui va nous ajouter de la sécurité en publiant de façon sécurisé plusieurs sites web (Wordpress, Wekan...) qui sont eux mêmes hébergés par plusieurs serveurs web à travers notre PfSense.
Pour la mise en place de celui-ci, nous allons nous diriger vers le paquet Squid.

On appelle reverse-proxy car comme son nom l'indique, il permet de faire l'inverse du proxy : 
- **Proxy :** accéder à Internet depuis le réseau local.
- **Reverse-proxy :** accéder au réseau local depuis internet.

En terme de sécurité, le reverse proxy possède plusieurs avantages :
- Anonymise les serveurs web lors des requêtes du client.
- Optimise le temps de réponse des requêtes
- Répartis la charge entre plusieurs serveurs
- Analyse les flux en amont (afin d'éviter les erreurs)

1. Installer les packages Squid dans _System/Package Manager/Available Packages_. Rechercher _squid_ et cliquer sur **Install**.

2. Créer la règle "System Tunables" dans _System/Advanced/System Tunables_, cliquer sur **New** et ajoutez les valeurs si dessous :
![](https://cdn.discordapp.com/attachments/522143202426224654/846539879465877504/unknown.png)
<br>

3. Configurer Squid Reverse Proxy dans _Services/Squid Reverse Proxy_ et ajouter un élément dans la section **Web Servers**. Ainsi on va pouvoir ajouter un serveur web.
![](https://cdn.discordapp.com/attachments/522143202426224654/846543054864121866/unknown.png)

<br>

4.Pour associer à un nom de domaine, il faut se rendre dans la section **Mappings**, ajouter avec **Add** et sélectionner le serveur qui hébergent le site. Puis entrer le nom de domaine dans la zone **URI**
![](https://cdn.discordapp.com/attachments/522143202426224654/846544670154293298/unknown.png)
<br>

5. Vérifier que le flux HTTPS est autorisé sur le firewall dans les règles WAN, provenant de n'importe quelle source et à destination du port 443.
<br>

6. Afin de sécuriser les connexions entre le serveur web et le navigateur du client, il est nécessaire d'avoir un certificat SSL. Il permettent principalement de chiffrer les informations confidentielles du client et de sécuriser les données entre les serveurs.
Pour cela on se rend dans _System/Cert. Manager_ puis dans la section CAs, cliquer sur **Add**.
Ajouter un nom, et la method ci-dessous :
![](https://cdn.discordapp.com/attachments/522143202426224654/846549640261533716/unknown.png)
Ensuite passer dans la section Certificates puis créer un certificat. 
![](https://cdn.discordapp.com/attachments/522143202426224654/846551083417075722/unknown.png)
<br>

7. Activation et ajout du certificat SSL : Dans _Services/Squid Reverse Proxy_, activer **Enabled HTTPS Reverse Proxy** et au niveau du champ **Reverse SSL Certificate**, sélectionner le certificat nouvellement créé.
![](https://cdn.discordapp.com/attachments/522143202426224654/843594325929361418/unknown.png)
<br>

4. Pour activez complètement **Squid**, il est nécessaire de se rendre dans _Services/Squid Proxy_, et d'actiber **Enable Squid Proxy**.
<br>

## Serveur Nginx
Afin d'installer correctement nos applications web, nous avons choisi Nginx commer serveur HTTP. Pour mettre en place celui, munissez-vous tout d'abord d'une machine virtuel avec Debian d'installer.

1. Dans un premier temps, installer Nginx avec `apt-get install nginx` puis arrêter Apache s'il était déjà installer avec `systemctl stop apache2.service`.

2. Ensuite, décommenter la ligne `server_tokens off;` dans le fichier  _/etc/nginx/nginx.conf_ pour plus de sécurité, afin de ne pas envoyer les informations telles que le numéro de version de Nginx.

3. Pour passer en IP static, modifier le fichier _/etc/network/interfaces_ comme :
```
auto lo
iface lo inet loopback

auto ens33
iface ens33 inet static
        address 192.168.2.3
        netmask 255.255.255.0
        gateway 192.168.2.50
```
<br>

### MariaDB

MariaDB est l'un des système de gestion de base de données le plus utilisé sous Debian. A présent nous allons l'installer notre système de base de données et créer nous créer une **database** (_ynov_) ainsi qu'un **utilisateur** (_admin_) avec **un mot de passe** (_Passw0rd_) et lui accorder les **droits d'accès** à cette base.

1. Installation des paquets avec `apt-get install mariadb-server`.

2. Configuration de celle ci avec `mysql_secure_installation`
![](https://cdn.discordapp.com/attachments/522143202426224654/845450380368543813/unknown.png)

3. Se connecter avec `mysql -u root -p` puis :
  - créer une base de donnée _ynov_ : `CREATE DATABASE blog;`
  - créer un utilisateur _admin_ : `CREATE USER "admin"@"localhost";`
  - accorder lui un mot de passe : `SET password FOR "admin"@"localhost" = password('Passw0rd');`
  - accorder lui les droits : `GRANT ALL ON ynov.* to 'admin'@'%' IDENTIFIED BY 'Passw0rd' WITH GRANT OPTION;` puis `FLUSH PRIVILEGES;`.

4. Par défaut MariaDB n'accorde l'accès uniquement à localhost. 
On doit donc modifier le fichier `/etc/mysql/mariadb.conf.d/50-server.cnf` et passer la ligne ``bind-address` à **0.0.0.0**.



### Wordpress
Nous allons maintenant, installer Wordpress

1. **Installation :**
  - se placer dans le répertoire _/var/www_ 
  - télécharger la dernière version de Wordpress avec `wget http://fr.wordpress.org/latest-fr_FR.tar.gz`
  - décompresser l'archive `tar -xzvf latest-fr_FR.tar.gz`
  - supprimer l'archive `rm latest-fr_FR.tar.gz`

2. **Configuration :**:
  * Installation de modules php avec `apt-get -y install php-cli php-mysql php-curl php-gd php-intl`:
    - PHP PDO est un connecteur pour MariaDB
    - PHP CURL est nécessaire pour certains plugins
    - PHP GD permettra d'opérer sur les images
    - PHP INTL est un support de l’internationalisation
<br>
  * Vérification de la version de php avec `php -v`

  ![](https://cdn.discordapp.com/attachments/522143202426224654/845604558214529024/unknown.png)

  * **PHP FPM :**
    - Installation du module php-fpm, il permet la communication entre le serveur Nginx et PHP. On utilise `apt-get install -y php-fpm`
    - Création du pool wordpress : `/etc/php/7.3/fpm/pool.d/wordpress.conf`
    ```    
    [wordpress]
    listen = /var/run/wordpress.sock

    listen.owner = wordpress
    listen.group = www-data

    user = wordpress
    group = www-data

    pm = ondemand
    pm.max_children = 10
    pm.process_idle_timeout = 60s
    pm.max_requests = 500
    ```

    > `listen` est l'interface d'écoute des requêtes. On utilisera donc un socket qui va nous permettre d'interfacer des processus entre eux sans passer par la couche réseau du système.

    - Création du fichier _/etc/nginx/sites-available/wordpress.conf_
    ```
    upstream php-wp {
    server            unix:/var/run/wordpress.sock;
    }

    server {
        listen            80;
        listen            [::]:80;
        server_name       projet-infra.fr;

        root              /var/www/wordpress;

        index             index.php;

        location / {
            try_files     $uri $uri/ /index.php?$args;
        }

        location = /favicon.ico {
            log_not_found off;
            access_log    off;
        }

        location = /robots.txt {
            allow                    all;
            log_not_found off;
            access_log    off;
        }

        location ~ .php$ {
            include       fastcgi.conf;
            fastcgi_pass  php-wp;
        }

        location ~* .(js|css|png|jpg|jpeg|gif|ico)$ {
            expires       max;
            log_not_found off;
        }
    }
    ```

    3. **Activation :**
    Redémarrer le service nginx : `systemctl restart nginx`
    <br>
    > si une erreur apparait de type `nginx.service: Failed to read PID from file /run/nginx.pid: Invalid argument` lancer les commandes suivantes:
    >- `mkdir /etc/systemd/system/nginx.service.d`
    >- `printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/system/nginx. service.d/override.conf`
    >- `systemctl daemon-reload`
    >- `systemctl restart nginx`

    4. Mise en route de Wordpress :
    - Si vous avez mis en place votre reverse proxy, rendez vous sur votre navigateur (hors serveur) à l'adresse http://www.projet-infra.fr.
    - Soit vous activez une redirection depuis un port non utilisé de pfSense (ex : 3222) vers le port 80 de votre cluster Heartbeat. Ainsi vous pouvez, depuis votre navigateur vous rendre sur wordpresse avec 192.168.5.50:3222.

    Vous arrivez sur cette page :
    ![](https://cdn.discordapp.com/attachments/522143202426224654/846018468708745266/unknown.png)

    > Si lorsque vous cliquez sur **Envoyer** il se produit une erreur, vérifier que vous avez bien configurer MariaDB.

    <br>

### Wekan


```
[Unit]
Description=Wekan Server
After=syslog.target
After=network.target

[Service]
Type=simple
Restart=always
StandardOutput=syslog
SyslogIdentifier=Wekan
User=wekan
Group=wekan
Environment=MARIADB_URL=mariadb://192.168.2.5/wekan
Environment=ROOT_URL=https://wekan.com
Environment=PORT=8000
WorkingDirectory=/bundle
ExecStart=/usr/bin/node /bundle/main.js

[Install]
WantedBy=multi-user.target
```

> Ressources : https://websetnet.net/fr/how-to-install-wekan-on-debian-9/
  


## Haute disponibilité
Nous utilisons Heartbeat pour la mise en place de la haute disponibilité. En effet nous allons créé un clustering avec nos deux serveurs web qui partageront la même ip.

Les étapes ci-dessous devront être réalisés sur les deux serveurs.

1. Configuration de Heartbeat : 
`apt-get install heartbeat`
<br>

2. Seulement 3 fichiers de configurations sont nécessaire à la mise en place d'un cluster de serveur, ils se situe dans le dossier _/etc/heartbeat/_:

- **ha.cf** :
```bash
# Indication du fichier de log
logfile /var/log/heartbeat.log

# Les logs heartbeat seront gérés par syslog, dans la catégorie daemon
logfacility daemon

# On liste tous les membres de notre cluster heartbeat (par les noms de préférences)
node SrvWeb1
node SrvWeb2

# On défini la périodicité de controle des noeuds entre eux (en seconde)
keepalive 1

# Au bout de combien de seconde un noeud sera considéré comme "mort"
deadtime 10

# Quelle carte résau utiliser pour les broadcasts Heartbeat (eth1 dans mon cas)
bcast eth1

# Adresse du routeur pour vérifier la connexion au net
ping 192.168.2.50

# Rebascule-t-on automatiquement sur le primaire si celui-ci redevient vivant
auto_failback yes
```

- **authkeys** dont on sécurise les droits avec `chmod 600 /etc/heartbeat/authkeys` :
```bash
auth 1
1 sha1 SecretKey
```

- **haresources**, va permettre de définir l'action à effectuer lorsqu'un serveur passera de passif à actif:
`SrvWeb1 192.168.2.40`

3. Démarrage du serveur avec `systemctl start heartbeat`

> Il est possible de vérifier la connexion dans `/var/log/heartbeat.log`
![](https://cdn.discordapp.com/attachments/522143202426224654/845343578535886918/unknown.png)

4. Notre ip c'est correctement changer en 192.168.2.40 sur le ServWeb1 :
![](https://cdn.discordapp.com/attachments/522143202426224654/845346496156532781/unknown.png)

5. Si on éteint le ServWeb1, et que l'on regarde les logs du côté du ServWeb2,  on observe que l'ip 192.168.2.40 à bien était transférer sur le ServWeb2: 
![](https://cdn.discordapp.com/attachments/522143202426224654/845346287293431818/unknown.png)

> Réplication des données : Il est possible avec de répliquer ses données de façon synchroniser entre ses deux serveurs web avec [DRBD](https://www.informatiweb-pro.net/admin-systeme/linux/debian-clustering-et-balancement-de-services-avec-2-serveurs.html) 


## Loadbalancing
Nous pouvons utiliser HAProxy 




## Contact

François BONNIN - francois.bonnin@ynov.com / Raimana SUN - raimana.sun@ynov.com

Project Link: [https://github.com/Raimana92/Projet-Infra-SI](https://github.com/Raimana92/Projet-Infra-SI)


----

# Document d'exploitation

- Accès à distance pour le télétravail  à PfSense et aux différents serveurs web.
- Installation de Wordpress et accès au site.
192.168.5.50:2222 (addresse du serveur pfsense) sur le port 2222.

- SSH pour installer une nouvelle appli:
  - install paquet apt-get
  - ajout à virtual host / site-available

- Si un serveur tombe un autre prend le relais avec heartbeat