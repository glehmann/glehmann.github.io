---
title: Utiliser un VPN avec transmission, FranceTV Pluzz et Arte+7
tags: code france
---

De nombreux vendeurs de VPN fournissent des solutions efficaces contre la surveillance
systématique façon [Hadopi], en changeant l'origine apparente de l'accès internet.
Un accès hors de France n'intéresse plus Hadopi.

Malheureusement, si il est simple de configurer son linux pour utiliser un VPN,
il est moins simple de faire cohabiter différents outils sur la même machine.

Les services de replay de télévision par exemple, utilisent l'origine géographique
de la connexion internet pour autoriser ou refuser l'accès. Si la connexion parait
venir hors de France, l'accès sera impossible à [FranceTV pluzz]. Arte étant
une chaîne franco-allemande, [Arte+7] autorise aussi l'accès depuis l'Allemagne.

Pour pouvoir accéder à [FranceTV pluzz] et [Arte+7], le réseau est configuré de façon
à faire passer tout le trafic vers l'extérieur par le VPN jusqu'en Allemagne, à
l'exception des accès aux services de replay [FranceTV pluzz].
De cette façon, par défaut, toutes les communications seront anonymisées par le VPN,
y compris tout ce à quoi on a pas pensé au moment de la configuration du VPN.
Seules certaines adresses correspondant à [FranceTV pluzz] peuvent passer par la
connexion hors VPN.

Et pour éviter les oublis, [transmission] est démarré et arrêté en même temps
que le VPN.

## Configurer l'accès par VPN

Les exemples listés ici utilisent [hidemyass] et [OpenVPN]. Plusieurs serveurs
sont disponibles en Allemagne, dans plusieurs villes. Les pings varient sensiblement
entre les différentes villes disponibles. Pour moi, celle donnant le ping le
plus faible est Francfort.

### Télécharger la configuration de base

On télécharge d'abord un fichier de configuration correspondant à la ville désirée.
Par exemple,
[https://www.hidemyass.com/vpn-config/UDP/Germany.Hesse.Frankfurt_LOC1S1.UDP.ovpn](https://www.hidemyass.com/vpn-config/UDP/Germany.Hesse.Frankfurt_LOC1S1.UDP.ovpn),
et on le sauve dans `/etc/openvpn/hma.conf`.

### Utiliser plusieurs serveurs

Pour utiliser tous les serveurs
disponibles dans cette ville, on ajoute des commandes `remote` correspondant
à chaque serveur et la commande `remote-random` qui permet de choisir un des
serveur au hasard. De cette façon, OpenVPN essaiera différents serveurs en cas
de problème de connexion. Pour Francfort, on obtient :

~~~
remote-random
remote 62.113.202.130 53
remote 62.113.251.2 53
remote 62.113.206.2 53
remote 62.113.206.130 53
remote 62.113.254.130 53
remote 62.113.251.130 53
remote 212.83.40.2 53
remote 212.83.59.130 53
~~~

### Authentification automatique

Pour que l'authentification se fasse sans demander de mot de passe, on ajoute
un fichier `/etc/openvpn/hmauser.pass` contenant le login et le mot de passe
séparé par un retour à la ligne :

~~~~
monLogin
monMotDePasse
~~~~

Il vaut mieux s'assurer que le fichier n'est accessible que par l'utilisateur `root` :

~~~bash
chown root /etc/openvpn/hmauser.pass
chmod 600 /etc/openvpn/hmauser.pass
~~~

Ensuite, dans le fichier `/etc/openvpn/hma.conf`, on remplace `auth-user-pass`
par `auth-user-pass /etc/openvpn/hmauser.pass`

### Démarrage de la connexion VPN

Cette étape dépend de la distribution linux utilisée. Sur Ubuntu, c'est

~~~bash
sudo service openvpn start
~~~

À ce stade, l'ensemble des connexions passent par le VPN, ce qu'on peut vérifier
avec la commande `traceroute` ou en accédant à un service permettant de
localiser l'adresse IP comme [celui-ci](http://www.mon-ip.com/info-adresse-ip.php).
Avec la configuration précédente, la connexion doit apparaitre comme venant d'Allemagne.

À ce stade, il est normalement impossible d'accéder au contenu de FranceTV pluzz
puisque la connexion semble venir d'Allemagne.

## Accéder à FranceTV pluzz

Pour accéder à FranceTV pluzz, on force certaines adresses IP à ne pas passer
par le VPN. Pour ça on crée deux nouveaux fichiers :

* `/etc/openvpn/hma-up.sh` :

~~~bash
#!/bin/bash

for ip in 77.67.21.223 77.67.11.114 5.178.43.9 ; do
  /sbin/ip route add $ip/32 via 192.168.1.1
done
~~~

* `/etc/openvpn/hma-down.sh` :

~~~bash
#!/bin/bash

for ip in 77.67.21.223 77.67.11.114 5.178.43.9 ; do
  /sbin/ip route del $ip/32 via 192.168.1.1
done
~~~

Il faut s'assurer que ces fichiers sont exécutables :

~~~bash
chmod a+x /etc/openvpn/hma-*.sh
~~~

Enfin on ajoute ces lignes au fichier `/etc/openvpn/hma.conf` :

~~~
script-security 2
up /etc/openvpn/transmission-up.sh
down /etc/openvpn/transmission-down.sh
~~~

enfin on ajoute ces lignes à `/etc/hosts`, pour forcer la résolution des noms
utilisée par FranceTV pluzz à une seule adresse IP :

~~~
77.67.21.223    webservices.francetelevisions.fr
77.67.11.114    medias2.francetv.fr
5.178.43.9      ftvodhdsecz-f.akamaihd.net
~~~

À ce stade, FranceTV pluzz doit à nouveau être accessible.

## Démarrer transmission en même temps que la connexion VPN

Pour démarrer [transmission] en même temps que le VPN, on ajoute tout simplement

~~~bash
/etc/init.d/transmission-daemon start
~~~

dans `/etc/openvpn/hma-up.sh`, et

~~~bash
/etc/init.d/transmission-daemon stop
~~~

dans `/etc/openvpn/hma-down.sh`.

Il faut également s'assurer que le service qui démarre transmission est désactivé.
Toujours sur ubuntu :

~~~bash
sudo update-rc.d transmission-daemon remove
~~~


[Hadopi]: http://www.hadopi.fr/
[FranceTV pluzz]: http://pluzz.francetv.fr/
[Arte+7]: http://www.arte.tv/guide/fr/plus7
[transmission]: https://www.transmissionbt.com/
[hidemyass]: https://www.hidemyass.com/
[OpenVPN]: https://openvpn.net/index.php/open-source.html
