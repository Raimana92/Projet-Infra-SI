# Projet-Infra-SI

## A propos

Mise en place d’une architecture permettant de déployer des applications web redondées et sécurisées :

<img src="Images\Schéma.png" width=600px>


## Outils utilisés

* <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/Vmware_workstation_16_icon.svg/1200px-Vmware_workstation_16_icon.svg.png" width="20px"> VMware Workstation Pro
* <img src="https://img1.freepng.fr/20180504/fqe/kisspng-debian-apt-linux-distribution-raspbian-blue-logo-5aec8fd5de5470.3000639315254527579107.jpg" height="20px"> Debian
* <img src="https://image.flaticon.com/icons/png/512/59/59137.png" height="20px"> Wordpress
* <img src="https://img.icons8.com/color/452/nginx.png" height="20px"> Nginx
* <img src="https://colibri.unistra.fr/application/assets/images/courses/sql_icone.png" height="20px"> Base de donnée SQL

## Informations - Plan IP


|    | pfSense WAN | pfSense LAN | Serveur Web1 : Debian| Serveur Web2 : Debian| Base de donnée: Debian|
|:--:|:-------:|:---:|:--------------------:|:--------------------:|:---------------------:|
| **IP**      | WAN : 192.168.5.50 | LAN : 192.168.2.50 | 192.168.2.3 | 192.168.2.4 | 192.168.2.5 
| **Netmask** | 255.255.255.0 | 255.255.255.0 | 255.255.255.0 | 255.255.255.0 | 255.255.255.0 
| **Gateway** | 192.168.5.2 | | 192.168.2.50 | 192.168.2.50 | 192.168.2.50 |


## Rules NAT - PfSense

Ajouter du Port-Forwarding (redirection de port) pour le SSH et pour le protocole HTTP : 
* Serveur Web1 :
  -  ssh : **user@pfSenseWANAddress -p 2223**

* Serveur Web2 :
  -  ssh : **user@pfSenseWANAddress -p 2224**

* Base de donnée:
  -  ssh : **user@pfSenseWANAddress -p 2225**


<img src="Images\Rules - Port Forwarding.png" border=2px>

## Reverse Proxy - Squid

1. Ajout des serveurs web et association à un nom de domaine `www.projet-infra.fr`

![](https://cdn.discordapp.com/attachments/522143202426224654/843577952905723984/unknown.png)
<br>

2. Autorisation du flux HTTPS dans le firewall l'interface WAN provenant de n'importe quelle source et à destination du port 443.
<br>

3. Création et Ajout du certificat SSL
![](https://cdn.discordapp.com/attachments/522143202426224654/843594325929361418/unknown.png)
<br>

4. Activation du **Server Squid**

> Reverse proxy source : https://www.it-connect.fr/reverse-proxy-https-avec-pfsense/

## Serveur Nginx
Nous utilisons un serveur HTTP Nginx pour une question de performances. Nginx est reconnu pour ses hautes performances, sa stabilité, son ensemble de fonctionnalités, sa configuration simple ainsi que sa faible consommation en ressources.

1. Installer Nginx `apt-get install nginx` puis arrêter Apache s'il était déjà installer avec `systemctl stop apache2.service`

2. Décommenter la ligne `server_tokens off;` dans le fichier  _/etc/nginx/nginx.conf_ pour plus de sécurité, afin de ne pas envoyez les informations telles que le numéro de version de Nginx.



### Wordpress
https://howto.wared.fr/installation-wordpress-ubuntu-nginx/

1. Installation : 
  - se placer dans le répertoire _/var/www_ 
  - télécharger la dernière version de Wordpress avec `wget http://fr.wordpress.org/latest-fr_FR.tar.gz`
  - décompressez l'archive `tar -xzvf latest-fr_FR.tar.gz`
  - supprimer l'archive `rm latest-fr_FR.tar.gz`

2. Configuration :
  * Installation de modules php avec `apt-get -y install php-cli php-mysql php-curl php-gd php-intl`:
    - PHP PDO est un connecteur pour MariaDB
    - PHP CURL est nécessaire pour certains plugins
    - PHP GD permettra d'opérer sur les images
    - PHP INTL est un support de l’internationalisation
<br>
  * Vérification de la version de php avec `php -v`

  ![](https://cdn.discordapp.com/attachments/522143202426224654/845604558214529024/unknown.png)

  * PHP FPM :
    - Installation du module php-fpm, il permet la communcation entre le serveur Nginx et PHP `apt-get install -y php-fpm`
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

    3. Activation :
    Redémarrer le service nginx : `systemctl restart nginx`
    <br>
    > si une erreur apparait de type `nginx.service: Failed to read PID from file /run/nginx.pid: Invalid argument` lancer les commandes suivantes:
    >- `mkdir /etc/systemd/system/nginx.service.d`
    >- `printf "[Service]\nExecStartPost=/bin/sleep 0.1\n" > /etc/systemd/system/nginx. service.d/override.conf`
    >- `systemctl daemon-reload`
    >- `systemctl restart nginx`

    4. Mise en route de Wordpress : 

  
### MariaDB
Nous utilisons MariaDB comme serveur de base de données. 
1. Installation des paquets avec `apt-get install mariadb-server`.

2. Configuration de celle ci avec `mysql_secure_installation`
![](https://cdn.discordapp.com/attachments/522143202426224654/845450380368543813/unknown.png)

3. Se connecter avec `mysql -u root -p` puis :
  - créer une base de donnée _ynov_ : `CREATE DATABASE blog;`
  - créer un utilisateur _admin_ : `CREATE USER "admin"@"localhost";`
  - accorder lui un mot de passe : `SET password FOR "admin"@"localhost" = password('admin');`
  - accorder lui les droits : `GRANT ALL PRIVILEGES ON ynov.* TO "admin"@"localhost" IDENTIFIED BY "admin";` puis `FLUSH PRIVILEGES;`.

> Ressource : https://www.geek17.com/fr/content/debian-10-buster-installer-et-configurer-mariadb-107


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

- PfSense
- Installation de Wordpress
- SSH pour installer une nouvelle appli:
  - install paquet apt-get
  - ajout à virtual host / site-available
- Si un serveur tombe un autre prend le relais avec heartbeat