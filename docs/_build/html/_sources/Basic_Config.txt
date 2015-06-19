********************************
Configuration basique de Nginx
********************************

Dans ce chapitre, nous verrons comment créer une configuration minimal pour un serveur web.
Nous allons donc voir la syntax utilisé dans le fichier de configuration.
Nous allons également essayé de comprendre les différentes directives qui vont permettre d'optimiser votre serveur web pour le traffique et ll'installation du hardware.
On va créer des pages de test pour vérifier que la configuration est bonne.

	* Presentation de la syntaxe
	* Configuration basique
	* Etablir une configuration approprié
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

Ce que Nginx va interpreter:

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

On peut voir également dans le fichier par défaut, la directive ``events``, celle-ce se présente en bloque:

.. code-block:: bash

	events {
		worker_connections 1024;
	}

Ce bloque est définie par le module ``events``. La directive 
