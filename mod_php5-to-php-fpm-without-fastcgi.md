# Passer de mod_php5 à php5-fpm sans la librairie fastcgi sous debian8

En effet j'ai eu à passer un serveur sous debian8 qui fonctionnait avec apache2 et mod_php5 en php5-fpm pour pouvoir avoir
deux version de PHP en même temps sur le serveur.

La petite blague c'est que debian a retiré le paquet : *libapache2-mod-fastcgi* et du coup impossible d'installer comme lu
sur de nombreux tutoriels ma version de php5-fpm.

Pour se faire il faut au final, installer php5-fpm et activer mod_proxy_fgci (ou l'installer si pas présent, mais normalement il l'est).
Il faudra aussi désactiver le mod_php5

````bash
# installation
sudo apt-get install php5-fpm

# le temps de faire la configuration on coupe les services apache2 et php5-fpm
sudo service apache2 stop
sudo service php5-fpm stop

# on désactive mod_php5 et on active proxy_fcgi
sudo a2dismod php5
sudo a2enmod proxy_fcgi

```

Une fois que c'est fait pour que votre site Internet fonctionne avec php5-fpm il vous suffit d'ajouter ceci dans votre vhost : 

```
# Dans l'idée placé ceci après la déclaration de votre DocumentRoot
<FilesMatch "\.php$">
    # Note : la seule partie variable est /path/to/app.sock
    SetHandler  "proxy:unix:/var/run/php5-fpm.sock|fcgi://localhost/"
</FilesMatch>
```

Il faut maintenant redémarer les services : 

```bash
sudo service apache2 start
sudo service php5-fpm start
```

## Bonus : installer php7-fpm en plus

Pour l'installation pure et dure je vous renvoie à mon article : https://www.kanjian.fr/comment-installer-nginx-mysql-php7-debian.html

Une fois le *apt-get install* fait et les services / mods activé pour PHP7 il faudra dans vos Vhost ajouter ceci : 

```
# Dans l'idée placé ceci après la déclaration de votre DocumentRoot et il ne faut pas avoir le FilesMatch présent plus haut
ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/var/www/$1
```

Il vous faudra aussi modifier le fichier */etc/php/7.0/fpm/pool.d/www.conf* pour modifier la valeur de *listen* comme ceci : 

```
listen = 127.0.0.1:9000
```



