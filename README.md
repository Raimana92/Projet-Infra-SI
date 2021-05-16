# Projet-Infra-SI

## A propos

Mise en place d’une architecture permettant de déployer des applications web redondées et sécurisées :

<img src="Images\Schéma.jpg" width=600px>


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

> Utilisation de ce tutoriel : [Reverse Proxy](https://www.it-connect.fr/reverse-proxy-https-avec-pfsense/)

## Loadbalancing



## Contact

François BONNIN - francois.bonnin@ynov.com / Raimana SUN - raimana.sun@ynov.com

Project Link: [https://github.com/Raimana92/Projet-Infra-SI](https://github.com/Raimana92/Projet-Infra-SI)
