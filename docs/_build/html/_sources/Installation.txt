***********************
Installation
***********************

Dans ce chapitre, nous allons voir comment installer le serveur web Nginx sur une distribution Debian 8.
Nous verrons comment installer celui-ci depuis son code source et non depuis le paquet présent sur les dépots, et ce, pour disposer de la dernière version disponible.

Pour faire l'installation depuis les sources, nous aurons besoin de certains modules.

GCC - Compilateur.
-------------------

Pour vérifier que celui-ci est présent sur notre distribution, nous pouvons lancer la commande suivante:

.. note::

	GCC va permettre de compiler plusieurs langages, tel que: C, C++, Java, Ada, FORTRAN, ...

.. code-block:: bash
	
	$ gcc

Si nous récupérons la sortie suivante, le compilateur est bien installé:

.. code-block:: bash

	gcc: fatal error: no input files
	compilation terminated.

Par contre, si nous récupérons la sortie suivante, il faudra installer le compilateur:

.. code-block:: bash

	~bash: gcc: comand not found

Voici les comandes à lancer pour l'installer:

.. code-block:: bash

	# yum groupinstall "Development Tools"

ou

.. code-block:: bash

	# apt-get install build-essentials

PCRE - Perl Compatible Regular Expression.
-------------------------------------------

Cette librairie est requise pour l'installation de Nginx.
Les modules "Rewrite" et "HTTP Core" utilisent cette librairie pour la syntax des expressions regulières.
2 paquets sont requis:

	#. pcre
	#. pcre-devel

On va installer ces 2 paquets:

.. code-block:: bash
	
	# yum install pcre pcre-devel

	# apt-get install libpcre3 libpcre3-dev

Zlib - Compression.
---------------------

Ce paquet est requis pour l'utilisation de gzip.
2 paquets sont requis:

	#. zlib1g
	#. zlib1g-dev

.. code-block:: bash

	# yum install zlib zlib-devel

	# apt-get install zlib1g zlib1g-dev


OpenSSL
---------

Cette librairie permet de servir des pages web securisées.
2 paquets sont requis:

	#. openssl
	#. openssl-dev

.. code-block:: bash

	# yum install openssl openssl-devel

	# apt-get install openssl opensll-dev

.. note::

	Lors de l'installation sur Debian 8, openssl-dev n'etait pas présent.

Nous pouvons maintenant installer Nginx depuis les sources.


Télécharger Nginx.
------------------------

Tout d'abord, on vérifie la version stable actuelle depuis le site `Nginx`_.

.. _Nginx: http://www.nginx.org

Au moment où nous écrivons ce tutoriel, la version stable est la 1.8.0, pour la télécharger, il suffit de lancer la commande suivante:

.. code-block:: bash

	$ mkdir src && cd src
	$ wget http://nginx.org/download/nginx-1.8.0.tar.gz
	--2015-06-17 15:00:46--  http://nginx.org/download/nginx-1.8.0.tar.gz
	Résolution de nginx.org (nginx.org)… 206.251.255.63, 2606:7100:1:69::3f
	Connexion à nginx.org (nginx.org)|206.251.255.63|:80… connecté.
	requête HTTP transmise, en attente de la réponse… 200 OK
	Taille : 832104 (813K) [application/octet-stream]
	Sauvegarde en : « nginx-1.8.0.tar.gz »

	nginx-1.8.0.tar.gz                       100%[=============================>] 812,60K   376KB/s   ds 2,2s

	2015-06-17 15:00:48 (376 KB/s) — « nginx-1.8.0.tar.gz » sauvegardé [832104/832104]

Ensuite, nous allons extraire l'archive dans le dossier courant:

.. code-block:: bash

	$ tar zxvf nginx-1.8.0.tar.gz

Nous allons à présent, configurer le processus de compilation pour que le binaire soit parfaitement compatible avec notre OS.

Les options de configuration.
-------------------------------

Il y a habituellement 3 étapes lorsque l'on installe une application depuis les sources:

	* La configuration
	* La compilation
	* L'installation

L'étape de configuration permet la séléction d'options qui ne seront plus éditables après l'installation. Il est donc fortement recommandé de suivre les étapes si nous ne voulons pas avoir des surprises plus tard, comme des modules inexistants ou des fichiers au mauvais endroit.

Pour une installation par défaut
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

	$ cd nginx-1.8.0/

	# ./configure
	# make
	# make install

Le dossier d'installation sera, par défaut, ``/usr/local/nginx``.
Pour avoir un aperçu des options possible lors de la configuration, on peut lancer la commande suivante:

.. code-block:: bash

	$ ./configure --help

L'option --prefix.
^^^^^^^^^^^^^^^^^^^

Durant la configuration, une attention particulière doit être porté à l'option ``--prefix``.
C'est l'emplacement où va être installé Nginx, par défaut, ``/usr/local/nginx``.
Le problème avec cet emplacement, sera lors d'un upgrade de Nginx, la nouvelle installation va écraser les anciens fichiers de configuration.
Il est donc recommandé d'utiliser un prefix pour chaque version:

.. code-block:: bash

	./configure --prefix=/usr/local/nginx-1.8.0

En complément, pour pouvoir faire des changements simple lors d'un upgrade de version, est de créer un lien symbolique ``/usr/local/nginx`` qui va pointer sur ``/usr/local/nginx-1.8.0``. 
Une fois l'upgrade effectué, il suffit de modifier le lien pour le faire pointer sur ``/usr/local/nginx-newer.verion``.
Ce qui va permettre au script ``init`` d'utiliser à chaque fois la dernière version de Nginx.

Erreurs lors de la configuration
---------------------------------

Dans certains cas, la comande de configuration peut échouer. Dans la plupart des cas, ces erreurs peuvent provenir d'un chemin non spécifié ou manquant, ou de modules/paquets requis manquant.

Dans ce cas, il faudra procéder attentivement à certaines vérifications pour être sûr que l'application pourra être compilé.
On peut également consulter le fichier ``objs/autoconf.err`` pour plus de détails sur les erreurs de compilation.
Ce fichier est généré pendant le processus de configuration et fera apparaître l'endroit où le processus a échoué.

S'assurer de l'installation des pre-requis.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Il y a au minimum 4 pre-requis:

	* GCC
	* PCRE
	* zlib
	* OpenSSL

Les 3 derniers sont fournis en 2 paquets distinct: la librairie et les sources de developement.

Un autre erreurs peut venir du fait que le script de configuration ne peut trouver les fichiers de ces modules.

Lors du lancement de la configuration, on peut ajouter le chemin:

.. code-block:: bash

	./configure [...] --with-openssl=/usr/lib64

On peut également s'assurer des droits d'accées aux dossier/fichiers. 

Lancer la configuration
-------------------------


Nous allons lancer la configuration par défaut avec juste un prefix différent:

.. code-block:: bash

	./configure --prefix=/usr/local/nginx-1.8.0

Normallement, la configuration doit se passer sans encombre, si non, voir les chapitres précédent.

Compilation et installation.
------------------------------

Après le succès de la configuration on peut lancer la compilation.
Pour lancer cette compilation, il faut lancer la commande ``make`` dans le dossier source du projet:

.. code-block:: bash

	$ make

Sur la Debian 8, ``make`` n'est pas installé par défaut. Il suffit de lancer le gestionnaire de paquet:

.. code-block:: bash

	# apt-get install make

.. note::

	Après lancement de ``make`` une erreur de permissions apparaît sur les fichiers contenus dans le dossier ``objs/``. Il suffit de faire un ``chown -R <user>:<group> objs/``

Si la compilation a réussi, on devrait avoir la sortie suivante:

.. code-block:: bash

	make[1]: Leaving directory '[path]/src/nginx-1.8.0'

On peut maintenant lancer l'installation de Nginx.
On doit lancer cette commande avec les privilèges ``root``:

.. code-block:: bash

	# make install

Il ne devrait pas y avoir d'erreur durant l'installation.

Création du lien symbolique.
-----------------------------

On va créer un lien symbolique comme dit plus haut, voici la commande:

.. code-block:: bash

	ln -s /usr/local/nginx-1.8.0 /usr/local/nginx

Donne:

.. code-block:: bash

	# cd /usr/local
	# ls -al
	...
	lrwxrwxrwx  1 root staff   22 juin  17 18:19 nginx -> /usr/local/nginx-1.8.0
	drwxr-sr-x  6 root staff 4096 juin  17 18:18 nginx-1.8.0
	...


Démarrer et arreter le daemon.
-------------------------------

Pour démarrer Nginx, il suffit de lancer la commande suivante:

.. code-block:: bash

	/usr/local/nginx/sbin/nginx

Si le daemon est déjà en cours, un message d'erreur apparait:

.. code-block:: bash

	....
	nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
	nginx: [emerg] still could not bind()

On peut aussi vérifier que le daemon est en cours en lançant la commande suivante:

.. code-block:: bash

	netstat -laputen | grep 80
	tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      0          83055       10609/nginx

Lorsque le daemon est en cours, on peut lancer les commandes suivantes:

+-----------------+------------------------------+
| Commande        | Description                  |
+=================+==============================+
| nginx -s stop   | Stop le daemon immédiatement |
+-----------------+------------------------------+
| nginx -s quit   | Stop le daemon élégament     |
+-----------------+------------------------------+
| nginx -s reopen | Re-ouvre les fichiers de log |
+-----------------+------------------------------+
| nginx -s reload | Relance la configuration     |
+-----------------+------------------------------+

Lorsque l'on démarre, arrête, ou n'importe quel des commandes précédentes, le daemon va vérifier la configuration. Si la configuration n'est pas bonne la commande va échouer, même si nous voulons arreter le daemon.

La manière forte pour arrêter la processus est d'utiliser ``kill`` ou ``killall``:

.. code-block:: bash

	# killall nginx

Tester la configuration.
--------------------------

Le script ``nginx`` permet avec l'option ``-t`` de tester la configuration en cours. Ce qui permet après un changement de la configuration de voir si une erreur a été insérer avant de relancer le daemon.

.. code-block:: bash

	$ /usr/local/nginx/sbin/nginx -t
	nginx: the configuration file /usr/local/nginx-1.8.0/conf/nginx.conf syntax is ok
	nginx: configuration file /usr/local/nginx-1.8.0/conf/nginx.conf test is successful

Par contre, il est fortement recomandé de ne pas travailler directement sur le fichier de configuration en production.
Il est préférable de créer un fichier de configuration test situé dans un dossier séparé, comme notre dossier perso, ``/home/<mon_login>/test.conf``. On peut, de ce fait, tester notre copie de configuration au lieu du fichier de configuration lui-même:

.. code-block:: bash

	$ .nginx -t -c /home/<mon_login>/test.conf

On peut donc, après avoir validé que la configuration est bonne, remplacer la configuration actuelle par la nouvalle:

.. code-block:: bash

	$ cp -i /home/<mon_login>/test.conf /usr/local/nginx/conf/nginx.conf

	cp: erase 'nginx.conf' ? yes
	$ ./nginx -s reload

Ajouter Nginx en tant que service système.
--------------------------------------------

Pour plus de facilité, nous allons rendre controllable Nginx comme une commande standard et l'ajouter au demarrage sytème ainsi qu'à l'arrêt.

Tout d'abord, il faut arrêter le script:

.. code-block:: bash

	# kill `cat /usr/local/nginx/logs/nginx.pid`

Création du script init pour sysVinit.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

	# vim /etc/init.d/nginx

Voici ce qu'il faut écrire:

.. code-block:: bash

	#!/bin/bash

	### BEGIN INIT INFO
	# Provides:				nginx
	# Required-Start:		$all
	# Require-Stop:			$all
	# Default-Start:		2 3 4 5
	# Default-Stop:			0 1 6
	# Short-Description:	starts the nginx web server
	# Description:			starts nginx using start-stop-daemon
	### END INIT INFO

	PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
	DAEMON=/usr/local/nginx/sbin/nginx
	NAME=nginx
	DESC=nginx

	test -x $DAEMON || exit 0

	# Include nginx defaults if available
	if [ -f /etc/defaults/nginx ] ; then
		. /etc/defaults/nginx
	fi

	set -e

	. /lib/lsb/init-functions

	case "$1" in
		start)
			echo -n "Starting $DESC: "
			start-stop-daemon --start --quiet --pidfile /usr/local/nginx/logs/$NAME.pid --exec $DAEMON -- $DAEMON_OPTS || true
			echo "$NAME."
			;;
		stop)
			echo -n "Stopping $DESC: "
			start-stop-daemon --stop --quiet --pidfile /usr/local/nginx/logs/$NAME.pid --exec $DAEMON || true
			echo "$NAME."
			;;
		restart|force-reload)
			echo -n "Restarting $DESC: "
			start-stop-daemon --stop --quiet --pidfile /usr/local/nginx/logs/$NAME.pid --exec $DAEMON || true
			sleep 1
			start-stop-daemon --start --quiet --pidfile /usr/local/nginxlogs/$NAME.pid --exec $DAEMON -- $DAEMON_OPTS || true
			echo "$NAME."
			;;
		reload)
			echo -n "Reloading $DESC configuration: "
			start-stop-daemon --stop --signal HUP --quiet --pidfile /usr/local/nginx/logs/$NAME.pid --exec $DAEMON || true
			echo "$NAME."
			;;
		status)
			status_of_proc -p /usr/local/nginx/logs/$NAME.pid "$DAEMON" nginx && exit 0 || exit $?
			;;
		*)
			N=/etc/init.d/$NAME
			echo "Usage: $N {start|stop|restart|reload|force-reload|status}" >&2
			exit 1
			;;
	esac

	exit 0

Création du script de démarrage pour systemd.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash
	
	# vim /lib/systemd/system/nginx.service

	[Unit]
	Description=A high performance web server and a reverse proxy server
	After=network.target

	[Service]
	Type=forking
	PIDFile=/usr/local/nginx/logs/nginx.pid
	ExecStartPre=/usr/local/nginx/sbin/nginx -t -q -g 'daemon on; master_process on;'
	ExecStart=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;'
	ExecReload=/usr/local/nginx/sbin/nginx -g 'daemon on; master_process on;' -s reload
	ExecStop=/usr/local/nginx/sbin/nginx -s quit

	[Install]
	WantedBy=multi-user.target

On vérifie le bon fonctionnement de notre script:

.. code-block:: bash

	# systemctl status nginx
	● nginx.service - A high performance web server and a reverse proxy server
	   Loaded: loaded (/lib/systemd/system/nginx.service; disabled)
	   Active: inactive (dead)

On demarre notre service:

.. code-block:: bash

	# systemctl start nginx

On vérifie:

.. code-block:: bash

	# systemctl status nginx
	● nginx.service - A high performance web server and a reverse proxy server
	   Loaded: loaded (/lib/systemd/system/nginx.service; disabled)
	   Active: active (running) since jeu. 2015-06-18 01:49:54 CEST; 43s ago
	  Process: 11505 ExecStart=/usr/local/nginx/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
	  Process: 11504 ExecStartPre=/usr/local/nginx/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
	Main PID: 11508 (nginx)
	   CGroup: /system.slice/nginx.service
			   ├─11508 nginx: master process /usr/local/nginx/sbin/nginx -g daemon on; master_process on;
			   └─11509 nginx: worker process


