********************************
Configuration basique de Nginx
********************************

Dans ce chapitre, nous verrons comment créer une configuration minimal pour un serveur web.
Nous allons donc voir la syntaxe utilisée dans le fichier de configuration.
Nous allons également essayé de comprendre les différentes directives qui vont permettre d'optimiser votre serveur web pour le trafique et l'installation du hardware.
On va créer des pages de test pour vérifier que la configuration est bonne.

	* Présentation de la syntaxe
	* Configuration basique
	* Établir une configuration approprié
	* Tester un site web
	* Tester et maintenir le serveur

Fichier de configuration.
----------------------------

Le fichier de configuration se trouve à ``/usr/local/nginx/conf/nginx.conf``

Voici le fichier de configuration par défaut:

.. code-block:: bash

	#user  nobody;
	worker_processes  1;

	#error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;

	#pid        logs/nginx.pid;


	events {
	    worker_connections  1024;
	}


	http {
		include       mime.types;
		default_type  application/octet-stream;

		#log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	    #                  '$status $body_bytes_sent "$http_referer" '
		#                  '"$http_user_agent" "$http_x_forwarded_for"';

		#access_log  logs/access.log  main;

		sendfile        on;
		#tcp_nopush     on;

		#keepalive_timeout  0;
		keepalive_timeout  65;

		#gzip  on;

		server {
			listen       80;
			server_name  localhost;

			#charset koi8-r;

			#access_log  logs/host.access.log  main;

			location / {
				root   html;
				index  index.html index.htm;
			}

			#error_page  404              /404.html;

			# redirect server error pages to the static page /50x.html
			#
			error_page   500 502 503 504  /50x.html;
			location = /50x.html {
				root   html;
			}

			# proxy the PHP scripts to Apache listening on 127.0.0.1:80
			#
			#location ~ \.php$ {
			#    proxy_pass   http://127.0.0.1;
			#}

			# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
			#
			#location ~ \.php$ {
			#    root           html;
			#    fastcgi_pass   127.0.0.1:9000;
			#    fastcgi_index  index.php;
			#    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
			#    include        fastcgi_params;
			#}

			# deny access to .htaccess files, if Apache's document root
			# concurs with nginx's one
			#
			#location ~ /\.ht {
			#    deny  all;
			#}
		}

		# another virtual host using mix of IP-, name-, and port-based configuration
		#
		#server {
		#    listen       8000;
		#    listen       somename:8080;
		#    server_name  somename  alias  another.alias;
	
		#    location / {
		#        root   html;
		#        index  index.html index.htm;
		#    }
		#}

		# HTTPS server
		#
		#server {
		#    listen       443 ssl;
		#    server_name  localhost;
		
		#    ssl_certificate      cert.pem;
		#    ssl_certificate_key  cert.key;

		#    ssl_session_cache    shared:SSL:1m;
		#    ssl_session_timeout  5m;

		#    ssl_ciphers  HIGH:!aNULL:!MD5;
		#    ssl_prefer_server_ciphers  on;

		#    location / {
		#        root   html;
		#        index  index.html index.htm;
		#    }
		#}

	}


La seconde ligne ``worker_processes`` est un directive, la valeur 1 indique que Nginx travail un seul ``worker`` processus. On voit que la directive finit par un ``;``

Organisation et inclusion.
---------------------------

Ligne 16, on peut voir la directive ``include``.

.. code-block:: bash

	include mime.types;

Comme son nom l'indique, cette directive va permettre d'inclure un fichier spécifique.
Voici un petit exemple pour comprendre:

.. code-block:: bash

	nginx.conf:
		user nginx nginx;
		worker_processes 4;
		include other_settings.conf;

	other_settings.conf:
		error_log logs/error.log;
		pid logs/nginx.pid;

Ce que Nginx va interpréter:

.. code-block:: bash

	user nginx nginx;
	worker_processes 4;
	error_log logs/error.log;
	pid logs/nginx.pid;


Par défaut, l'insertion de sites ce fait de cette manière. On va utiliser la directive ``<mon_site>.conf`` ou:

.. code-block:: bash

	include sites/*.conf;

Les directives en bloque.
--------------------------

On peut voir également dans le fichier par défaut, la directive ``events``, celle-ci se présente en bloque:

.. code-block:: bash

	events {
		worker_connections 1024;
	}

Ce bloque est définie par le module ``events``. La directive ``worker_connections`` peut seulement être utilisé dans le bloque ``events``.

On peut également imbriquer des bloques les uns dans les autres:

.. code-block:: bash

	http {
		server {
			listen 80;
			server_name exemple.fr;
			acces_log /var/log/nginx/exemple.fr.log;
			location ^~ /admin/ {
				index index.php;
			}
		}
	}

Dans le bloque ``http``, on peut déclarer plusieurs bloque ``server``. Ces bloques ``server`` permettent de configurer des hôtes virtuelles.
Nous verrons au fur et à mesure l'utilisation des bloques.

Les directives des modules noyau.
-----------------------------------

Nous allons voir quelques modules qui vont permettre de paramétrer le noyau. Ces modules ne peuvent être utilisés qu'une seule fois, et doivent être placé dans la racine du fichier de configuration.

+-----------------------+---------------------------------------------------------------------------+
| Nom et contexte    	| Syntaxe et description                                                    |
+=======================+===========================================================================+
| daemon                | Valeurs acceptées: *on* ou *off*                                          |
|                       |                                                                           |
|                       | Syntaxe: *daemon on;*                                                     |
|                       |                                                                           |
|                       | Valeur par défaut *on;*                                                   |
|                       |                                                                           |  
|                       | Active ou désactive le mode *daemon*. Si on le désactive, le programme    | 
|                       | ne démarrera pas en arrière plan; Il restera au premier plan lorsqu'il    |
|                       | sera lancé depuis le shell. Peut-être utile pour débbuguer.               |
+-----------------------+---------------------------------------------------------------------------+
| debug_points          | Valeurs acceptés: *stop* ou *abort*                                       |
|                       |                                                                           |
|                       | Syntaxe: *debug_points stop;*                                             |
|                       |                                                                           |
|                       | Valeur par défaut: *None*                                                 |
|                       |                                                                           |
|                       | Active un point de débbuguage.Il suffit d'utiliser *stop* pour interrompre|
|                       | l'application. Utiliser *abort* pour arrêter le point de débbuguage et    |
|                       | fichier ``core dump``.                                                    |
|                       | Pour désactiver cette option, ne tout simplement pas utiliser cette       |
|                       | directive.                                                                |
+-----------------------+---------------------------------------------------------------------------+
| env                   | Syntaxe: *env MY_VARIABLE;*                                               |
|                       |                                                                           |
|                       |          *env MY_VARIABLE=my_value;*                                      |
|                       |                                                                           |
|                       | Permet de (re)définir une variable d'environnement.                       |
+-----------------------+---------------------------------------------------------------------------+
| error_log             | Syntaxe: *error_log /file/path level;*                                    |
|                       |                                                                           |
| Contexte: main, http, | Valeur par défaut: logs/error.log error.                                  |
| server, et location   |                                                                           |
|                       | Où ``level`` est une des valeurs suivantes: *debug*, *info*, *notice*,    |
|                       | *warn*, *error*, et *crit* (du plus au moins détaillé: *debug* fourni     |
|                       | les messages de logs fréquents, *crit* seulement les messages critiques). |
|                       |                                                                           |
|                       | Active les messages d'erreur sur plusieurs niveaux: Application, HTTP     |
|                       | serveur, hôte virtuel, et dossier de l'hôte virtuel.                      |
|                       |                                                                           |
|                       | En redirigeant la sortie des logs vers */dev/null/*, on peut désactiver   |
|                       | les logs d'erreurs. En utilisant la directive suivante depuis la racine   |
|                       | du fichier de configuration:                                              |
|                       |                                                                           |
|                       |           ``error_log /dev/null crit;``                                   |
+-----------------------+---------------------------------------------------------------------------+
| lock_file             | Syntaxe: *File Path*                                                      |
|                       |                                                                           |
|                       |        ``lock_file logs/nginx.lock;``                                     |
|                       |                                                                           |
|                       | Valeur par défaut: Définie au moment de la compilation.                   |
|                       |                                                                           |
|                       | Sur la plupart des OS cette directive est ignorée.                        |
+-----------------------+---------------------------------------------------------------------------+
| log_not_found         | Valeurs acceptées: *on* ou *off*                                          |
|                       |                                                                           |
| Contexte: main, http, |         ``log_not_found on;``                                             |
| server, et location   |                                                                           |
|                       | Valeur par défaut: *on*                                                   |
|                       |                                                                           |
|                       | Active ou désactive les logs pour les erreurs HTTP "404 not found".       |
|                       | Si les logs sont remplis avec des 404 à cause des *favicon.ico* ou        |
|                       | *robots.txt*, on peut mettre à *off*.                                     |
+-----------------------+---------------------------------------------------------------------------+
| master_process        | Valeurs acceptées: *on* ou *off*                                          |
|                       |                                                                           |
|                       |               ``master_process on;``                                      |
|                       |                                                                           |
|                       | Valeur par défaut: *on*                                                   |
|                       |                                                                           |
|                       | Si activé, Nginx va démarrer plusieurs processus: un principal (le        |
|                       | processus maître) et le processus *worker*. Si désactivé, Nginx travail   |
|                       | avec un processus unique.                                                 |
+-----------------------+---------------------------------------------------------------------------+
|                       | Reste page 48 à 51 de ``pcre_jit``  à ``worker_aio_requests``             |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
+-----------------------+---------------------------------------------------------------------------+


Les modules d'évenements.
-----------------------------

Les modules d'évènement vient avec des directives qui permet de configurer les mécanismes de réseau. Certains paramètres ont une impacte importante sur les performances.

Toutes les directives de la liste suivante, doivent être placé dans le bloque ``events``, qui doit être placé à la racine du fichier de configuration:

.. code-block:: bash

	user nginx nginx;
	master_process on;
	worker_processes 4;
	events {
		worker_connections 1024;
		use epoll;
	}
	[...]


+-----------------------+---------------------------------------------------------------------------+
| Nom et contexte    	| Syntaxe et description                                                    |
+=======================+===========================================================================+
|                       | Reste page 52 à 53 de ``accept_mutex``  à ``worker_connections``          |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
|                       |                                                                           |
+-----------------------+---------------------------------------------------------------------------+


Les modules de configuration.
--------------------------------

Le module de configuration permet l'insertion de fichier grâce à la directive ``include``.
Cette directive peut être inséré n'importe où dans le fichier de configuration et accepte un seul paramètre - le chemin du fichier.

.. code-block:: bash

	include /file/path.conf;
	include sites/*.conf;

.. note::

	Si aucun chemin absolue n'est défini,le chemin est relatif au dossier de configuration. Par défaut, ``include sites/exemple.conf`` va insérer le fichier: ``/usr/local/nginx/conf/sites/exemple.conf``.


Travailler sur la configuration par défaut.
---------------------------------------------

User
^^^^^^

Par défaut:

.. code-block:: bash

	#user nobody;

A changer par:

.. code-block:: bash

	user www-data www-data

Worker_processes
^^^^^^^^^^^^^^^^^

Par défaut:

.. code-block:: bash

	worker_processes 1;

A changer par:

.. code-block:: bash

	worker_processes 4;

Permet d'utiliser les 4 cœurs de notre machine. A adapter suivant notre machine.


Log_not_found
^^^^^^^^^^^^^^^

Permet de logger les erreurs 404 ou non. Il est fortement recommandé de le laisser à *on*

.. code-block:: bash

	log_not_found on;

Worker_connections
^^^^^^^^^^^^^^^^^^^

Cette directive permet de définir le nombre de connections admises par le serveur. Il est en relation avec le *worker_processes*, 1 worker permet 1024 connections.



Les tests de perforance.
------------------------------

Nous allons utiliser 3 outils pour tester les performances de notre serveur:

	* httperf
	* Autobench
	* OpenWebLoad

Ces outils vont permettre de générer de multiples requêtes HTTP pour étudier les résultats.

Httperf
^^^^^^^^

Httperf s'utilise en ligne de commande, on peut le trouver généralement dans les dépôts des distributions.
Voici un exemple de commande:

.. code-block:: bash

	$ httperf --server 192.168.56.101 --port 80 --uri /index.html --rate 300 --num-conn 30000 --num-call 1 --timeout 5

Voici à quoi correspond les différentes options:

	* --server: Le site que l'on veut tester
	* --uri: le chemin du fichier qui sera téléchargé
	* --rate: Combien de requêtes par seconde
	* --num-conn: le nombre de connections
	* --num-call:combien de requêtes envoyées par connections
	* --timeout nombre de seconde avant de considérer une requête perdu


