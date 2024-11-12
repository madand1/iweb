h1. Práctica (2 / 3): Instalación/migración de aplicaciones web PHP

h2. Documenta de la forma más precisa posible cada uno de los pasos que has dado para migrar una de las dos aplicaciones y capturas de pantalla donde se demuestre que esta funcionando el cliente de NextCloud.

1. Preparación de la *VPS*

Lo primero que vamos a hacer es esto:

<pre>
   22  sudo apt update && sudo apt upgrade -y
   23  sudo apt install nginx -y
   24  sudo apt install mysql-server -y
   25  sudo apt install mariadb-server -y
   26  sudo apt install php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-zip -y
   27  sudo apt install php8.3 php8.3-cli php8.3-common php8.3-curl php8.3-mbstring php8.3-mysql php8.3-xml php8.3-zip php8.3-gd -y
   28  sudo apt update
   29  sudo apt install software-properties-common -y
   30  sudo add-apt-repository ppa:ondrej/php
   31  sudo apt update
   32  sudo apt install php8.3 php8.3-cli php8.3-common php8.3-curl php8.3-mbstring php8.3-mysql php8.3-xml php8.3-zip php8.3-gd -y
   33  php -v
   34  sudo add-apt-repository ppa:ondrej/php -y
   35  php -v
   36  sudo add-apt-repository ppa:ondrej/php
   37  sudo apt install --reinstall software-properties-common -y
   38  sudo add-apt-repository ppa:ondrej/php
   39  echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ondrej-php.list
   40  sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E5267A6C
   41  sudo apt update
   42  sudo apt update
   43  sudo apt install php8.3 php8.3-cli php8.3-common php8.3-curl php8.3-mbstring php8.3-mysql php8.3-xml php8.3-zip php8.3-gd -y
   44  sudo apt install ppa-purge -y
   45  sudo ppa-purge ppa:ondrej/php
   46  php --v
   47  php -v
   48  apt list | grep php8.3
   49  sudo apt update
   50  sudo apt install -y build-essential libxml2-dev libssl-dev libcurl4-openssl-dev libjpeg-dev libpng-dev libwebp-dev libfreetype6-dev libonig-dev libzip-dev
   51  wget https://www.php.net/distributions/php-8.3.13.tar.gz
   52  tar -xzvf php-8.3.13.tar.gz
   53  cd php-8.3.13
   54  ./configure --prefix=/usr/local/php8.3 --with-zlib --with-curl --with-openssl --enable-mbstring --with-mysqli --with-pdo-mysql --enable-soap --enable-zip --with-gd --with-jpeg --with-webp --with-freetype
   55  make
   56  php -v
   57  ./configure --prefix=/usr/local/php8.3 --with-zlib --with-curl --with-openssl --enable-mbstring --with-mysqli --with-pdo-mysql --enable-soap --enable-zip --with-gd --with-jpeg --with-webp --with-freetype
   58  make
   59  sudo apt install -y pkg-config libxml2-dev libssl-dev libcurl4-openssl-dev libjpeg-dev libpng-dev libwebp-dev libfreetype6-dev
   60  ./configure --prefix=/usr/local/php8.3 --with-zlib --with-curl --with-openssl --enable-mbstring --with-mysqli --with-pdo-mysql --enable-soap --with-gd --with-jpeg --with-webp --with-freetype
   61  make
   62  sudo apt install -y libsqlite3-dev
   63  ./configure --prefix=/usr/local/php8.3 --with-zlib --with-curl --with-openssl --enable-mbstring --with-mysqli --with-pdo-mysql --enable-soap --with-gd --with-jpeg --with-webp --with-freetype
   64  make
   65  sudo apt-get install libgd-dev
   66  ./configure --enable-gd --with-jpeg --with-webp --with-freetype
   67  make
   68  free -h
   69  sudo fallocate -l 2G /swapfile
   70  sudo chmod 600 /swapfile
   71  sudo mkswap /swapfile
   72  sudo swapon /swapfile
   73  free -h
   74  sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
   75  make -j1
   76  sudo make test
   77  sudo apt install autoconf
   78  sudo apt install build-essential libxml2-dev libssl-dev libcurl4-openssl-dev
   79  sudo make test
   80  sudo make install
   81  php -v
   82  update-alternatives --display php
   83  sudo update-alternatives --install /usr/bin/php php /usr/local/bin/php 1
   84  sudo update-alternatives --install /usr/bin/phpize phpize /usr/local/bin/phpize 1
   85  sudo update-alternatives --install /usr/bin/php-config php-config /usr/local/bin/php-config 1
   86  sudo update-alternatives --config php
   87  php -v


</pre>

1. Exportar la base de datos

Ahora exportaremos la base de datos de nuestro servidor local. Esto lo haremos tanto para drupal que es el CMS que hemos usado como para Nextcloud, gracias al siguiente ocmando:

<pre>

mysqldump -u andy -p drupal_db > migracion.sql

</pre>

Aqui la comprobación de ambos comandos:

<pre>
vagrant@bd-andy:~$ mysqldump -u andy -p drupal_db > migracion.sql
Enter password: 
vagrant@bd-andy:~$ ls
migracion.sql

</pre>

Ahora procederemos a exportarlo con el comando scp a for-raptor.madand1.info

<pre>
vagrant@bd-andy:~$ scp migracion.sql root@ford-raptor.madand1.info:/home/root
The authenticity of host 'ford-raptor.madand1.info (217.160.226.33)' can't be established.
ED25519 key fingerprint is SHA256:Ek2rQnoGBO8Fpt2/kR3Q9YT9Dp58fKibLSgG+FNze70.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ford-raptor.madand1.info' (ED25519) to the list of known hosts.
root@ford-raptor.madand1.info's password: 
Permission denied, please try again.
root@ford-raptor.madand1.info's password: 
migracion.sql                                 100%   12MB   9.1MB/s   00:01    
vagrant@bd-andy:~$ 

</pre>

Y si vemos en nuestro vps:

<pre>
root@ford-raptor:~# ls
migracion.sql  php-8.3.13
root@ford-raptor:~# 

</pre>

Ahora vamos a proceder a entrar en el servidor web antiguo, y comprimir la carpeta Nextcloud:

<pre>
vagrant@web-andy:$ cd /var/www/

vagrant@web-andy:/var/www$ sudo apt install zip -y

vagrant@web-andy:/var/www$ sudo zip -r nextcloud.zip nextcloud

vagrant@web-andy:/var/www$ ls
drupal  drupal-11.0.6  drupal.tar.gz  html  nextcloud  nextcloud.zip

</pre>

Y lo migramos a la *VPS*:

<pre>
vagrant@web-andy:/var/www$ scp nextcloud.zip root@ford-raptor.madand1.info:/root
root@ford-raptor.madand1.info's password: 
nextcloud.zip                                100%  363MB   8.0MB/s   00:45                               100%  363MB   8.2MB/s   00:44  
</pre>

Comprobamos en nuestra VPS:

<pre>
root@ford-raptor:~# ls
migracion.sql  nextcloud.zip  php-8.3.13
root@ford-raptor:~# 

</pre>
Ahroa en nuestra VPS descomprimimos la carpeta:

<pre>
root@ford-raptor:~# unzip nextcloud.zip -d /var/www/nextcloud

</pre>

ya tenemos los módulos necesarios intalados con anterioridad, con lo que procederemos a darles permiso a la carpeta:

<pre>
root@ford-raptor:~# sudo chown -R www-data:www-data /var/www/nextcloud
root@ford-raptor:~# sudo chmod -R 755 /var/www/nextcloud

</pre>
Y la verificamos:

<pre>
root@ford-raptor:/var/www/nextcloud/nextcloud# ls -l
total 1304
drwxr-xr-x 42 www-data www-data    4096 Nov  7 08:40 3rdparty
-rwxr-xr-x  1 www-data www-data   26314 Nov  7 08:37 AUTHORS
-rwxr-xr-x  1 www-data www-data   34520 Nov  7 08:37 COPYING
drwxr-xr-x  2 www-data www-data    4096 Nov  7 08:37 LICENSES
drwxr-xr-x 60 www-data www-data    4096 Nov  7 17:41 apps
-rwxr-xr-x  1 www-data www-data    2079 Nov  7 08:37 composer.json
-rwxr-xr-x  1 www-data www-data    3400 Nov  7 08:37 composer.lock
drwxr-xr-x  2 www-data www-data    4096 Nov  7 17:40 config
-rwxr-xr-x  1 www-data www-data    3810 Nov  7 08:37 console.php
drwxr-xr-x 24 www-data www-data    4096 Nov  7 08:40 core
-rwxr-xr-x  1 www-data www-data    7550 Nov  7 08:37 cron.php
drwxr-xr-x  4 www-data www-data    4096 Nov  7 17:40 data
drwxr-xr-x  2 www-data www-data   20480 Nov  7 08:37 dist
-rwxr-xr-x  1 www-data www-data     331 Nov  7 08:37 index.html
-rwxr-xr-x  1 www-data www-data    3485 Nov  7 08:37 index.php
drwxr-xr-x  6 www-data www-data    4096 Nov  7 08:37 lib
-rwxr-xr-x  1 www-data www-data     308 Nov  7 08:37 occ
drwxr-xr-x  2 www-data www-data    4096 Nov  7 08:37 ocs
drwxr-xr-x  2 www-data www-data    4096 Nov  7 08:37 ocs-provider
-rwxr-xr-x  1 www-data www-data 1131946 Nov  7 08:37 package-lock.json
-rwxr-xr-x  1 www-data www-data    7227 Nov  7 08:37 package.json
-rwxr-xr-x  1 www-data www-data    2821 Nov  7 08:37 public.php
-rwxr-xr-x  1 www-data www-data    4435 Nov  7 08:37 remote.php
drwxr-xr-x  4 www-data www-data    4096 Nov  7 08:37 resources
-rwxr-xr-x  1 www-data www-data      26 Nov  7 08:37 robots.txt
-rwxr-xr-x  1 www-data www-data    1327 Nov  7 08:37 status.php
drwxr-xr-x  3 www-data www-data    4096 Nov  7 08:37 themes
drwxr-xr-x  2 www-data www-data    4096 Nov  7 08:38 updater
-rwxr-xr-x  1 www-data www-data     383 Nov  7 08:40 version.php

</pre>

Configuraremos la base de datos en la VPS:

<pre>
root@ford-raptor:~# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 31
Server version: 10.11.6-MariaDB-0+deb12u1 Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE drupal_db;
Query OK, 1 row affected (0.004 sec)

MariaDB [(none)]> CREATE USER 'andy'@'localhost' IDENTIFIED BY 'andy';
Query OK, 0 rows affected (0.021 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON drupal_db.* TO 'andy'@'localhost';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.005 sec)

MariaDB [(none)]> EXIT;
Bye

</pre>

Importamos la base de datos:

<pre>
root@ford-raptor:~# mysql -u andy -p drupal_db < migracion.sql 
Enter password: 

</pre>

Verificamos:

<pre>
root@ford-raptor:~# mysql -u andy -p drupal_db
Enter password: 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 37
Server version: 10.11.6-MariaDB-0+deb12u1 Debian 12

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [drupal_db]> show tables;
+----------------------------------+
| Tables_in_drupal_db              |
+----------------------------------+
| batch                            |
| block_content                    |
| block_content__body              |
| block_content_field_data         |
| block_content_field_revision     |
| block_content_revision           |
| block_content_revision__body     |
| cache_access_policy              |
| cache_bootstrap                  |
| cache_config                     |
| cache_container                  |
| cache_data                       |
| cache_default                    |
| cache_discovery                  |
| cache_dynamic_page_cache         |
| cache_entity                     |
| cache_menu                       |
| cache_page                       |
| cache_render                     |
| cachetags                        |
| comment                          |
| comment__comment_body            |
| comment_entity_statistics        |
| comment_field_data               |
| config                           |
| file_managed                     |
| file_usage                       |
| help_search_items                |
| history                          |
| key_value                        |
| key_value_expire                 |
| locale_file                      |
| locales_location                 |
| locales_source                   |
| locales_target                   |
| menu_link_content                |
| menu_link_content_data           |
| menu_link_content_field_revision |
| menu_link_content_revision       |
| menu_tree                        |
| node                             |
| node__body                       |
| node__comment                    |
| node__field_image                |
| node__field_tags                 |
| node_access                      |
| node_field_data                  |
| node_field_revision              |
| node_revision                    |
| node_revision__body              |
| node_revision__comment           |
| node_revision__field_image       |
| node_revision__field_tags        |
| oc_accounts                      |
| oc_accounts_data                 |
| oc_activity                      |
| oc_activity_mq                   |
| oc_addressbookchanges            |
| oc_addressbooks                  |
| oc_appconfig                     |
| oc_appconfig_ex                  |
| oc_authorized_groups             |
| oc_authtoken                     |
| oc_bruteforce_attempts           |
| oc_calendar_appt_bookings        |
| oc_calendar_appt_configs         |
| oc_calendar_invitations          |
| oc_calendar_reminders            |
| oc_calendar_resources            |
| oc_calendar_resources_md         |
| oc_calendar_rooms                |
| oc_calendar_rooms_md             |
| oc_calendarchanges               |
| oc_calendarobjects               |
| oc_calendarobjects_props         |
| oc_calendars                     |
| oc_calendarsubscriptions         |
| oc_cards                         |
| oc_cards_properties              |
| oc_circles_circle                |
| oc_circles_event                 |
| oc_circles_member                |
| oc_circles_membership            |
| oc_circles_mount                 |
| oc_circles_mountpoint            |
| oc_circles_remote                |
| oc_circles_share_lock            |
| oc_circles_token                 |
| oc_collres_accesscache           |
| oc_collres_collections           |
| oc_collres_resources             |
| oc_comments                      |
| oc_comments_read_markers         |
| oc_dav_absence                   |
| oc_dav_cal_proxy                 |
| oc_dav_shares                    |
| oc_direct_edit                   |
| oc_directlink                    |
| oc_ex_apps                       |
| oc_ex_apps_daemons               |
| oc_ex_apps_routes                |
| oc_ex_event_handlers             |
| oc_ex_occ_commands               |
| oc_ex_settings_forms             |
| oc_ex_speech_to_text             |
| oc_ex_speech_to_text_q           |
| oc_ex_task_processing            |
| oc_ex_text_processing            |
| oc_ex_text_processing_q          |
| oc_ex_translation                |
| oc_ex_translation_q              |
| oc_ex_ui_files_actions           |
| oc_ex_ui_scripts                 |
| oc_ex_ui_states                  |
| oc_ex_ui_styles                  |
| oc_ex_ui_top_menu                |
| oc_federated_reshares            |
| oc_file_locks                    |
| oc_filecache                     |
| oc_filecache_extended            |
| oc_files_metadata                |
| oc_files_metadata_index          |
| oc_files_reminders               |
| oc_files_trash                   |
| oc_files_versions                |
| oc_flow_checks                   |
| oc_flow_operations               |
| oc_flow_operations_scope         |
| oc_group_admin                   |
| oc_group_user                    |
| oc_groups                        |
| oc_jobs                          |
| oc_known_users                   |
| oc_login_flow_v2                 |
| oc_mail_accounts                 |
| oc_mail_aliases                  |
| oc_mail_attachments              |
| oc_mail_classifiers              |
| oc_mail_coll_addresses           |
| oc_mail_internal_address         |
| oc_mail_local_messages           |
| oc_mail_mailboxes                |
| oc_mail_message_tags             |
| oc_mail_messages                 |
| oc_mail_messages_retention       |
| oc_mail_messages_snoozed         |
| oc_mail_provisionings            |
| oc_mail_recipients               |
| oc_mail_smime_certificates       |
| oc_mail_tags                     |
| oc_mail_trusted_senders          |
| oc_migrations                    |
| oc_mimetypes                     |
| oc_mounts                        |
| oc_notes_meta                    |
| oc_notifications                 |
| oc_notifications_pushhash        |
| oc_notifications_settings        |
| oc_oauth2_access_tokens          |
| oc_oauth2_clients                |
| oc_open_local_editor             |
| oc_photos_albums                 |
| oc_photos_albums_collabs         |
| oc_photos_albums_files           |
| oc_preferences                   |
| oc_preferences_ex                |
| oc_privacy_admins                |
| oc_profile_config                |
| oc_properties                    |
| oc_ratelimit_entries             |
| oc_reactions                     |
| oc_recent_contact                |
| oc_richdocuments_assets          |
| oc_richdocuments_direct          |
| oc_richdocuments_template        |
| oc_richdocuments_wopi            |
| oc_schedulingobjects             |
| oc_share                         |
| oc_share_external                |
| oc_shares_limits                 |
| oc_storages                      |
| oc_storages_credentials          |
| oc_systemtag                     |
| oc_systemtag_group               |
| oc_systemtag_object_mapping      |
| oc_talk_attachments              |
| oc_talk_attendees                |
| oc_talk_bans                     |
| oc_talk_bots_conversation        |
| oc_talk_bots_server              |
| oc_talk_bridges                  |
| oc_talk_commands                 |
| oc_talk_consent                  |
| oc_talk_internalsignaling        |
| oc_talk_invitations              |
| oc_talk_poll_votes               |
| oc_talk_polls                    |
| oc_talk_proxy_messages           |
| oc_talk_reminders                |
| oc_talk_retry_ocm                |
| oc_talk_rooms                    |
| oc_talk_sessions                 |
| oc_taskprocessing_tasks          |
| oc_text2image_tasks              |
| oc_text_documents                |
| oc_text_sessions                 |
| oc_text_steps                    |
| oc_textprocessing_tasks          |
| oc_trusted_servers               |
| oc_twofactor_backupcodes         |
| oc_twofactor_providers           |
| oc_user_status                   |
| oc_user_transfer_owner           |
| oc_users                         |
| oc_vcategory                     |
| oc_vcategory_to_object           |
| oc_webauthn                      |
| oc_webhook_listeners             |
| oc_whats_new                     |
| path_alias                       |
| path_alias_revision              |
| queue                            |
| router                           |
| search_dataset                   |
| search_index                     |
| search_total                     |
| semaphore                        |
| sequences                        |
| sessions                         |
| shortcut                         |
| shortcut_field_data              |
| shortcut_set_users               |
| taxonomy_index                   |
| taxonomy_term__parent            |
| taxonomy_term_data               |
| taxonomy_term_field_data         |
| taxonomy_term_field_revision     |
| taxonomy_term_revision           |
| taxonomy_term_revision__parent   |
| user__roles                      |
| user__user_picture               |
| users                            |
| users_data                       |
| users_field_data                 |
| watchdog                         |
+----------------------------------+
245 rows in set (0.001 sec)
</pre>

Ahora creamos un nuevo archivo en /etc/ngninx/sites-available/nextcloud:


Y pondremos el *CNAME* donde corresponda

<pre>
root@ford-raptor:~# cat /etc/nginx/sites-available/nextcloud 
upstream php-handler {
    server unix:/run/php/php8.2-fpm.sock; # Asegúrate de que coincida con la versión de PHP
}

server {
    listen 80;
    listen [::]:80;

    server_name cloud.madand1.info;

    client_max_body_size 512M;
    client_body_timeout 300s;
    fastcgi_buffers 64 4K;

    root /var/www/nextcloud/nextcloud; # Asegúrate de que esta sea la ruta correcta
    index index.php index.html /index.php$request_uri;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_read_timeout 600;
    }

    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    add_header Referrer-Policy "no-referrer" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Download-Options "noopen" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Permitted-Cross-Domain-Policies "none" always;
    add_header X-Robots-Tag "none" always;
    add_header X-XSS-Protection "1; mode=block" always;

    fastcgi_hide_header X-Powered-By;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ^~ /.well-known {
        location = /.well-known/carddav { return 301 /remote.php/dav/; }
        location = /.well-known/caldav { return 301 /remote.php/dav/; }
        location /.well-known/acme-challenge { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation { try_files $uri $uri/ =404; }
        return 301 /index.php$request_uri;
    }

    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/) { return 404; }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) { return 404; }

    location ~ \.php(?:$|/) {
        rewrite ^/(?!index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|oc[ms]-provider/.+|.+/richdocumentscode/proxy) /index.php$request_uri;

        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;

        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;

        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ \.(?:css|js|svg|gif|png|jpg|ico)$ {
        try_files $uri /index.php$request_uri;
        expires 6M;
        access_log off;
    }

    location ~ \.woff2?$ {
        try_files $uri /index.php$request_uri;
        expires 7d;
        access_log off;
    }

    location /remote {
        return 301 /remote.php$request_uri;
    }

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }
}
root@ford-raptor:~# 

</pre>

Creamos un enlace simbólico para activar el sitio:

<pre>sudo ln -s /etc/nginx/sites-available/nextcloud /etc/nginx/sites-enabled/</pre>

Probamos la configuración de Nginx:

<pre>
root@ford-raptor:~# sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
</pre>
Reiniciamos nginx y php:

<pre>
root@ford-raptor:~# sudo systemctl restart nginx
root@ford-raptor:~# sudo systemctl restart php8.2-fpm.service
</pre>

Modificamos el archivo config.php para que se adapte al nuevo escenario:

<pre>
root@ford-raptor:~# cat /var/www/nextcloud/nextcloud/config/config.php 
<?php
$CONFIG = array (
  'instanceid' => 'oc7czrect518',
  'passwordsalt' => 'lKb4qUsJpnZikkeHzwhBqZHBQcEJpo',
  'secret' => '/0S3P+H5sDWxdiZJOEirXAWT3N2OcTe+RIdFxFDVt0pfISP8',
  'trusted_domains' => 
  array (
    0 => 'cloud.madand1.info',
  ),
  'datadirectory' => '/var/www/nextcloud/nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '30.0.2.2',
  'overwrite.cli.url' => 'http://cloud.madand1.info',
  'dbname' => 'drupal_db',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => '#######',
  'dbpassword' => '#######',
  'installed' => true,
);


</pre>


Y ya podremos acceder aqui: <pre>http://cloud.madand1.info</pre>

Desde aqui ponemos el dominio que tenemos contratad que es *cloud.madan1.info* 

Y hemos tenido que irnos a ionos y poner el CNAME, el cual esta ubicada en *Dominios & SSL*

!image1.png!

Una vez hecho esto probamos a conectarnos desde la URL:

!image2.png!

Ahora vamos a instalar en un ordenaro el cliente de NextCloud y realizar la configuración adecuada para acceder a lo que es *mi nube*, para ello haremos lo siguiente:

<pre>
madandy@toyota-hilux:~$ sudo apt install nextcloud-desktop
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
El paquete indicado a continuación se instaló de forma automática y ya no es necesario.
  linux-image-6.1.0-25-amd64
Utilice «sudo apt autoremove» para eliminarlo.
Se instalarán los siguientes paquetes adicionales:
  libharfbuzz-subset0 libkf5archive-data libkf5archive5 libnextcloudsync0
  libqt5keychain1 libqt5positioning5 libqt5qmlworkerscript5
  libqt5quickcontrols2-5 libqt5quicktemplates2-5 libqt5quickwidgets5
  libqt5sql5 libqt5sql5-sqlite libqt5webchannel5 libqt5webengine-data
  libqt5webenginecore5 libqt5webenginewidgets5 libqt5websockets5
  nextcloud-desktop-common nextcloud-desktop-doc nextcloud-desktop-l10n
  qml-module-qt-labs-folderlistmodel qml-module-qt-labs-platform
  qml-module-qt-labs-settings qml-module-qtgraphicaleffects qml-module-qtqml
  qml-module-qtqml-models2 qml-module-qtquick-controls2
  qml-module-qtquick-dialogs qml-module-qtquick-layouts
  qml-module-qtquick-privatewidgets qml-module-qtquick-templates2
  qml-module-qtquick-window2 qml-module-qtquick2
Se instalarán los siguientes paquetes NUEVOS:
  libharfbuzz-subset0 libkf5archive-data libkf5archive5 libnextcloudsync0
  libqt5keychain1 libqt5positioning5 libqt5qmlworkerscript5
  libqt5quickcontrols2-5 libqt5quicktemplates2-5 libqt5quickwidgets5
  libqt5sql5 libqt5sql5-sqlite libqt5webchannel5 libqt5webengine-data
  libqt5webenginecore5 libqt5webenginewidgets5 libqt5websockets5
  nextcloud-desktop nextcloud-desktop-common nextcloud-desktop-doc
  nextcloud-desktop-l10n qml-module-qt-labs-folderlistmodel
  qml-module-qt-labs-platform qml-module-qt-labs-settings
  qml-module-qtgraphicaleffects qml-module-qtqml qml-module-qtqml-models2
  qml-module-qtquick-controls2 qml-module-qtquick-dialogs
  qml-module-qtquick-layouts qml-module-qtquick-privatewidgets
  qml-module-qtquick-templates2 qml-module-qtquick-window2 qml-module-qtquick2
0 actualizados, 34 nuevos se instalarán, 0 para eliminar y 110 no actualizados.
Se necesita descargar 58,0 MB de archivos.
Se utilizarán 192 MB de espacio de disco adicional después de esta operación.
¿Desea continuar? [S/n] s
Des:1 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtquick-window2 amd64 5.15.8+dfsg-3 [27,8 kB]
Des:2 http://deb.debian.org/debian bookworm/main amd64 libqt5qmlworkerscript5 amd64 5.15.8+dfsg-3 [34,8 kB]
Des:3 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtquick2 amd64 5.15.8+dfsg-3 [36,5 kB]
Des:4 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtgraphicaleffects amd64 5.15.8-2 [68,9 kB]
Des:5 http://deb.debian.org/debian bookworm/main amd64 libharfbuzz-subset0 amd64 6.0.0+dfsg-3 [1.931 kB]
Des:6 http://deb.debian.org/debian bookworm/main amd64 libkf5archive-data all 5.103.0-1 [29,3 kB]
Des:7 http://deb.debian.org/debian bookworm/main amd64 libkf5archive5 amd64 5.103.0-1 [90,6 kB]
Des:8 http://deb.debian.org/debian bookworm/main amd64 libqt5keychain1 amd64 0.13.2-5 [49,7 kB]
Des:9 http://deb.debian.org/debian bookworm/main amd64 libqt5websockets5 amd64 5.15.8-2 [60,8 kB]
Des:10 http://deb.debian.org/debian bookworm/main amd64 libnextcloudsync0 amd64 3.7.3-1+deb12u1 [833 kB]
Des:11 http://deb.debian.org/debian bookworm/main amd64 libqt5positioning5 amd64 5.15.8+dfsg-3+deb12u1 [203 kB]
Des:12 http://deb.debian.org/debian bookworm/main amd64 libqt5quicktemplates2-5 amd64 5.15.8+dfsg-2 [397 kB]
Des:13 http://deb.debian.org/debian bookworm/main amd64 libqt5quickcontrols2-5 amd64 5.15.8+dfsg-2 [62,7 kB]
Des:14 http://deb.debian.org/debian bookworm/main amd64 libqt5quickwidgets5 amd64 5.15.8+dfsg-3 [39,4 kB]
Des:15 http://deb.debian.org/debian bookworm/main amd64 libqt5sql5 amd64 5.15.8+dfsg-11+deb12u2 [122 kB]
Des:16 http://deb.debian.org/debian bookworm/main amd64 libqt5sql5-sqlite amd64 5.15.8+dfsg-11+deb12u2 [57,0 kB]
Des:17 http://deb.debian.org/debian bookworm/main amd64 libqt5webchannel5 amd64 5.15.8-2 [58,2 kB]
Des:18 http://deb.debian.org/debian bookworm/main amd64 libqt5webengine-data all 5.15.13+dfsg-1~deb12u1 [7.285 kB]
Des:19 http://deb.debian.org/debian bookworm/main amd64 libqt5webenginecore5 amd64 5.15.13+dfsg-1~deb12u1 [39,9 MB]
Des:20 http://deb.debian.org/debian bookworm/main amd64 libqt5webenginewidgets5 amd64 5.15.13+dfsg-1~deb12u1 [118 kB]
Des:21 http://deb.debian.org/debian bookworm/main amd64 nextcloud-desktop-common all 3.7.3-1+deb12u1 [337 kB]
Des:22 http://deb.debian.org/debian bookworm/main amd64 nextcloud-desktop-l10n all 3.7.3-1+deb12u1 [769 kB]
Des:23 http://deb.debian.org/debian bookworm/main amd64 qml-module-qt-labs-platform amd64 5.15.8+dfsg-2 [75,6 kB]
Des:24 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtqml amd64 5.15.8+dfsg-3 [20,2 kB]
Des:25 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtqml-models2 amd64 5.15.8+dfsg-3 [21,0 kB]
Des:26 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtquick-templates2 amd64 5.15.8+dfsg-2 [70,4 kB]
Des:27 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtquick-controls2 amd64 5.15.8+dfsg-2 [903 kB]
Des:28 http://deb.debian.org/debian bookworm/main amd64 qml-module-qt-labs-folderlistmodel amd64 5.15.8+dfsg-3 [36,9 kB]
Des:29 http://deb.debian.org/debian bookworm/main amd64 qml-module-qt-labs-settings amd64 5.15.8+dfsg-3 [28,3 kB]
Des:30 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtquick-privatewidgets amd64 5.15.8-2 [44,6 kB]
Des:31 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtquick-dialogs amd64 5.15.8-2 [122 kB]
Des:32 http://deb.debian.org/debian bookworm/main amd64 qml-module-qtquick-layouts amd64 5.15.8+dfsg-3 [54,2 kB]
Des:33 http://deb.debian.org/debian bookworm/main amd64 nextcloud-desktop amd64 3.7.3-1+deb12u1 [2.189 kB]
Des:34 http://deb.debian.org/debian bookworm/main amd64 nextcloud-desktop-doc all 3.7.3-1+deb12u1 [1.952 kB]
Descargados 58,0 MB en 8s (7.415 kB/s)                                         
Extrayendo plantillas para los paquetes: 100%
Seleccionando el paquete qml-module-qtquick-window2:amd64 previamente no selecci
onado.
(Leyendo la base de datos ... 297145 ficheros o directorios instalados actualmen
te.)
Preparando para desempaquetar .../00-qml-module-qtquick-window2_5.15.8+dfsg-3_am
d64.deb ...
Desempaquetando qml-module-qtquick-window2:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete libqt5qmlworkerscript5:amd64 previamente no seleccionad
o.
Preparando para desempaquetar .../01-libqt5qmlworkerscript5_5.15.8+dfsg-3_amd64.
deb ...
Desempaquetando libqt5qmlworkerscript5:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete qml-module-qtquick2:amd64 previamente no seleccionado.
Preparando para desempaquetar .../02-qml-module-qtquick2_5.15.8+dfsg-3_amd64.deb
 ...
Desempaquetando qml-module-qtquick2:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete qml-module-qtgraphicaleffects:amd64 previamente no sele
ccionado.
Preparando para desempaquetar .../03-qml-module-qtgraphicaleffects_5.15.8-2_amd6
4.deb ...
Desempaquetando qml-module-qtgraphicaleffects:amd64 (5.15.8-2) ...
Seleccionando el paquete libharfbuzz-subset0:amd64 previamente no seleccionado.
Preparando para desempaquetar .../04-libharfbuzz-subset0_6.0.0+dfsg-3_amd64.deb 
...
Desempaquetando libharfbuzz-subset0:amd64 (6.0.0+dfsg-3) ...
Seleccionando el paquete libkf5archive-data previamente no seleccionado.
Preparando para desempaquetar .../05-libkf5archive-data_5.103.0-1_all.deb ...
Desempaquetando libkf5archive-data (5.103.0-1) ...
Seleccionando el paquete libkf5archive5:amd64 previamente no seleccionado.
Preparando para desempaquetar .../06-libkf5archive5_5.103.0-1_amd64.deb ...
Desempaquetando libkf5archive5:amd64 (5.103.0-1) ...
Seleccionando el paquete libqt5keychain1:amd64 previamente no seleccionado.
Preparando para desempaquetar .../07-libqt5keychain1_0.13.2-5_amd64.deb ...
Desempaquetando libqt5keychain1:amd64 (0.13.2-5) ...
Seleccionando el paquete libqt5websockets5:amd64 previamente no seleccionado.
Preparando para desempaquetar .../08-libqt5websockets5_5.15.8-2_amd64.deb ...
Desempaquetando libqt5websockets5:amd64 (5.15.8-2) ...
Seleccionando el paquete libnextcloudsync0:amd64 previamente no seleccionado.
Preparando para desempaquetar .../09-libnextcloudsync0_3.7.3-1+deb12u1_amd64.deb
 ...
Desempaquetando libnextcloudsync0:amd64 (3.7.3-1+deb12u1) ...
Seleccionando el paquete libqt5positioning5:amd64 previamente no seleccionado.
Preparando para desempaquetar .../10-libqt5positioning5_5.15.8+dfsg-3+deb12u1_am
d64.deb ...
Desempaquetando libqt5positioning5:amd64 (5.15.8+dfsg-3+deb12u1) ...
Seleccionando el paquete libqt5quicktemplates2-5:amd64 previamente no selecciona
do.
Preparando para desempaquetar .../11-libqt5quicktemplates2-5_5.15.8+dfsg-2_amd64
.deb ...
Desempaquetando libqt5quicktemplates2-5:amd64 (5.15.8+dfsg-2) ...
Seleccionando el paquete libqt5quickcontrols2-5:amd64 previamente no seleccionad
o.
Preparando para desempaquetar .../12-libqt5quickcontrols2-5_5.15.8+dfsg-2_amd64.
deb ...
Desempaquetando libqt5quickcontrols2-5:amd64 (5.15.8+dfsg-2) ...
Seleccionando el paquete libqt5quickwidgets5:amd64 previamente no seleccionado.
Preparando para desempaquetar .../13-libqt5quickwidgets5_5.15.8+dfsg-3_amd64.deb
 ...
Desempaquetando libqt5quickwidgets5:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete libqt5sql5:amd64 previamente no seleccionado.
Preparando para desempaquetar .../14-libqt5sql5_5.15.8+dfsg-11+deb12u2_amd64.deb
 ...
Desempaquetando libqt5sql5:amd64 (5.15.8+dfsg-11+deb12u2) ...
Seleccionando el paquete libqt5sql5-sqlite:amd64 previamente no seleccionado.
Preparando para desempaquetar .../15-libqt5sql5-sqlite_5.15.8+dfsg-11+deb12u2_am
d64.deb ...
Desempaquetando libqt5sql5-sqlite:amd64 (5.15.8+dfsg-11+deb12u2) ...
Seleccionando el paquete libqt5webchannel5:amd64 previamente no seleccionado.
Preparando para desempaquetar .../16-libqt5webchannel5_5.15.8-2_amd64.deb ...
Desempaquetando libqt5webchannel5:amd64 (5.15.8-2) ...
Seleccionando el paquete libqt5webengine-data previamente no seleccionado.
Preparando para desempaquetar .../17-libqt5webengine-data_5.15.13+dfsg-1~deb12u1
_all.deb ...
Desempaquetando libqt5webengine-data (5.15.13+dfsg-1~deb12u1) ...
Seleccionando el paquete libqt5webenginecore5:amd64 previamente no seleccionado.
Preparando para desempaquetar .../18-libqt5webenginecore5_5.15.13+dfsg-1~deb12u1
_amd64.deb ...
Desempaquetando libqt5webenginecore5:amd64 (5.15.13+dfsg-1~deb12u1) ...
Seleccionando el paquete libqt5webenginewidgets5:amd64 previamente no selecciona
do.
Preparando para desempaquetar .../19-libqt5webenginewidgets5_5.15.13+dfsg-1~deb1
2u1_amd64.deb ...
Desempaquetando libqt5webenginewidgets5:amd64 (5.15.13+dfsg-1~deb12u1) ...
Seleccionando el paquete nextcloud-desktop-common previamente no seleccionado.
Preparando para desempaquetar .../20-nextcloud-desktop-common_3.7.3-1+deb12u1_al
l.deb ...
Desempaquetando nextcloud-desktop-common (3.7.3-1+deb12u1) ...
Seleccionando el paquete nextcloud-desktop-l10n previamente no seleccionado.
Preparando para desempaquetar .../21-nextcloud-desktop-l10n_3.7.3-1+deb12u1_all.
deb ...
Desempaquetando nextcloud-desktop-l10n (3.7.3-1+deb12u1) ...
Seleccionando el paquete qml-module-qt-labs-platform:amd64 previamente no selecc
ionado.
Preparando para desempaquetar .../22-qml-module-qt-labs-platform_5.15.8+dfsg-2_a
md64.deb ...
Desempaquetando qml-module-qt-labs-platform:amd64 (5.15.8+dfsg-2) ...
Seleccionando el paquete qml-module-qtqml:amd64 previamente no seleccionado.
Preparando para desempaquetar .../23-qml-module-qtqml_5.15.8+dfsg-3_amd64.deb ..
.
Desempaquetando qml-module-qtqml:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete qml-module-qtqml-models2:amd64 previamente no seleccion
ado.
Preparando para desempaquetar .../24-qml-module-qtqml-models2_5.15.8+dfsg-3_amd6
4.deb ...
Desempaquetando qml-module-qtqml-models2:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete qml-module-qtquick-templates2:amd64 previamente no sele
ccionado.
Preparando para desempaquetar .../25-qml-module-qtquick-templates2_5.15.8+dfsg-2
_amd64.deb ...
Desempaquetando qml-module-qtquick-templates2:amd64 (5.15.8+dfsg-2) ...
Seleccionando el paquete qml-module-qtquick-controls2:amd64 previamente no selec
cionado.
Preparando para desempaquetar .../26-qml-module-qtquick-controls2_5.15.8+dfsg-2_
amd64.deb ...
Desempaquetando qml-module-qtquick-controls2:amd64 (5.15.8+dfsg-2) ...
Seleccionando el paquete qml-module-qt-labs-folderlistmodel:amd64 previamente no
 seleccionado.
Preparando para desempaquetar .../27-qml-module-qt-labs-folderlistmodel_5.15.8+d
fsg-3_amd64.deb ...
Desempaquetando qml-module-qt-labs-folderlistmodel:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete qml-module-qt-labs-settings:amd64 previamente no selecc
ionado.
Preparando para desempaquetar .../28-qml-module-qt-labs-settings_5.15.8+dfsg-3_a
md64.deb ...
Desempaquetando qml-module-qt-labs-settings:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete qml-module-qtquick-privatewidgets:amd64 previamente no 
seleccionado.
Preparando para desempaquetar .../29-qml-module-qtquick-privatewidgets_5.15.8-2_
amd64.deb ...
Desempaquetando qml-module-qtquick-privatewidgets:amd64 (5.15.8-2) ...
Seleccionando el paquete qml-module-qtquick-dialogs:amd64 previamente no selecci
onado.
Preparando para desempaquetar .../30-qml-module-qtquick-dialogs_5.15.8-2_amd64.d
eb ...
Desempaquetando qml-module-qtquick-dialogs:amd64 (5.15.8-2) ...
Seleccionando el paquete qml-module-qtquick-layouts:amd64 previamente no selecci
onado.
Preparando para desempaquetar .../31-qml-module-qtquick-layouts_5.15.8+dfsg-3_am
d64.deb ...
Desempaquetando qml-module-qtquick-layouts:amd64 (5.15.8+dfsg-3) ...
Seleccionando el paquete nextcloud-desktop previamente no seleccionado.
Preparando para desempaquetar .../32-nextcloud-desktop_3.7.3-1+deb12u1_amd64.deb
 ...
Desempaquetando nextcloud-desktop (3.7.3-1+deb12u1) ...
Seleccionando el paquete nextcloud-desktop-doc previamente no seleccionado.
Preparando para desempaquetar .../33-nextcloud-desktop-doc_3.7.3-1+deb12u1_all.d
eb ...
Desempaquetando nextcloud-desktop-doc (3.7.3-1+deb12u1) ...
Configurando nextcloud-desktop-common (3.7.3-1+deb12u1) ...
Configurando libqt5webengine-data (5.15.13+dfsg-1~deb12u1) ...
Configurando libqt5keychain1:amd64 (0.13.2-5) ...
Configurando qml-module-qtqml:amd64 (5.15.8+dfsg-3) ...
Configurando libqt5positioning5:amd64 (5.15.8+dfsg-3+deb12u1) ...
Configurando qml-module-qtquick-window2:amd64 (5.15.8+dfsg-3) ...
Configurando qml-module-qtquick-layouts:amd64 (5.15.8+dfsg-3) ...
Configurando qml-module-qt-labs-folderlistmodel:amd64 (5.15.8+dfsg-3) ...
Configurando libqt5qmlworkerscript5:amd64 (5.15.8+dfsg-3) ...
Configurando qml-module-qt-labs-settings:amd64 (5.15.8+dfsg-3) ...
Configurando libqt5quickwidgets5:amd64 (5.15.8+dfsg-3) ...
Configurando libqt5sql5:amd64 (5.15.8+dfsg-11+deb12u2) ...
Configurando libqt5websockets5:amd64 (5.15.8-2) ...
Configurando libqt5webchannel5:amd64 (5.15.8-2) ...
Configurando qml-module-qtqml-models2:amd64 (5.15.8+dfsg-3) ...
Configurando libharfbuzz-subset0:amd64 (6.0.0+dfsg-3) ...
Configurando libqt5quicktemplates2-5:amd64 (5.15.8+dfsg-2) ...
Configurando libkf5archive-data (5.103.0-1) ...
Configurando nextcloud-desktop-l10n (3.7.3-1+deb12u1) ...
Configurando qml-module-qtquick2:amd64 (5.15.8+dfsg-3) ...
Configurando libnextcloudsync0:amd64 (3.7.3-1+deb12u1) ...
Configurando nextcloud-desktop-doc (3.7.3-1+deb12u1) ...
Configurando qml-module-qtquick-privatewidgets:amd64 (5.15.8-2) ...
Configurando libqt5sql5-sqlite:amd64 (5.15.8+dfsg-11+deb12u2) ...
Configurando qml-module-qtquick-templates2:amd64 (5.15.8+dfsg-2) ...
Configurando qml-module-qt-labs-platform:amd64 (5.15.8+dfsg-2) ...
Configurando libqt5quickcontrols2-5:amd64 (5.15.8+dfsg-2) ...
Configurando libkf5archive5:amd64 (5.103.0-1) ...
Configurando qml-module-qtgraphicaleffects:amd64 (5.15.8-2) ...
Configurando libqt5webenginecore5:amd64 (5.15.13+dfsg-1~deb12u1) ...
Configurando qml-module-qtquick-dialogs:amd64 (5.15.8-2) ...
Configurando qml-module-qtquick-controls2:amd64 (5.15.8+dfsg-2) ...
Configurando libqt5webenginewidgets5:amd64 (5.15.13+dfsg-1~deb12u1) ...
Configurando nextcloud-desktop (3.7.3-1+deb12u1) ...
Procesando disparadores para desktop-file-utils (0.26-1) ...
Procesando disparadores para hicolor-icon-theme (0.17-2) ...
Procesando disparadores para gnome-menus (3.36.0-1.1) ...
Procesando disparadores para libc-bin (2.36-9+deb12u8) ...
Procesando disparadores para man-db (2.11.2-2) ...
Procesando disparadores para shared-mime-info (2.2-1) ...
Procesando disparadores para mailcap (3.70+nmu1) ...


</pre>
Ahora ejecutamos el comando por terminal *nextcloud*:

y nos aparecera esto por pantalla:

!image3.png!
Le damos a entrar y metemso nuestra dirección:

!image4.png!

Pinchamos en siguiente, el cual nos redirigira hasta aqui:

!image5.png!

El cual hemos tenido que conectar, y el cual en nextcloud desktop, tendremos lo siguiente:

!image6.png!

Y le damos a sincronizar por lo que ahroa nos saldra esta pantalla:

!image7.png!

Y ahora haciendo la prueba vamos a mover una foto:

<pre>
madandy@toyota-hilux:~/Nextcloud/Photos$ sudo mv /home/madandy/Descargas/Kodandy.jpeg .
madandy@toyota-hilux:~/Nextcloud/Photos$ ls -l
total 6128
-rw-r--r-- 1 madandy madandy  593508 nov  7 18:40  Birdie.jpg
-rw-r--r-- 1 madandy madandy  457744 nov  7 18:40  Frog.jpg
-rw-r--r-- 1 madandy madandy  474653 nov  7 18:40  Gorilla.jpg
-rw-r--r-- 1 madandy madandy  598795 oct  7 14:28  Kodandy.jpeg
-rw-r--r-- 1 madandy madandy 2170375 nov  7 18:40  Library.jpg
-rw-r--r-- 1 madandy madandy  797325 nov  7 18:40 'Nextcloud community.jpg'
-rw-r--r-- 1 madandy madandy     150 nov  7 18:40  Readme.md
-rw-r--r-- 1 madandy madandy  567689 nov  7 18:40  Steps.jpg
-rw-r--r-- 1 madandy madandy  167989 nov  7 18:40  Toucan.jpg
-rw-r--r-- 1 madandy madandy  427030 nov  7 18:40  Vineyard.jpg
madandy@toyota-hilux:~/Nextcloud/Photos$ 

</pre>

y ahora nos volvemos a meter y comrpobamos:

!image8.png!


h2. Las URL de acceso a las aplicaciones.

</pre>
http://cloud.madand1.info
http://www.madand1.info

</pre>

################################################################################################################################################################

h1. Práctica (3 / 3): Instalación/migración de aplicaciones web PHP

h2. Captura de pantalla para comprobar que el navegador tiene el certificado de Let’s Encrypt.

![alt text](img/image9.png)

h2. ¿Qué opción has elegido? ¿Qué pruebas realiza Let’s Encrypt para asegurar que somos los administrados del sitio web al elegir esa opción?

Para la página de cloud, lo haré desde aquí desde **certbot**:

<pre>
root@ford-raptor:~# sudo certbot --nginx -d cloud.madand1.info
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): asirandyglez@gmail.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Account registered.
Requesting a certificate for cloud.madand1.info

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/cloud.madand1.info/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/cloud.madand1.info/privkey.pem
This certificate expires on 2025-02-08.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for cloud.madand1.info to /etc/nginx/sites-enabled/nextcloud
Congratulations! You have successfully enabled HTTPS on https://cloud.madand1.info

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
root@ford-raptor:~# 


</pre>

Comprobación de como podemos ver si se genero el certificado:

<pre>
root@ford-raptor:~# sudo ls -l /etc/letsencrypt/live/cloud.madand1.info/
total 4
-rw-r--r-- 1 root root 692 Nov 10 10:19 README
lrwxrwxrwx 1 root root  42 Nov 10 10:19 cert.pem -> ../../archive/cloud.madand1.info/cert1.pem
lrwxrwxrwx 1 root root  43 Nov 10 10:19 chain.pem -> ../../archive/cloud.madand1.info/chain1.pem
lrwxrwxrwx 1 root root  47 Nov 10 10:19 fullchain.pem -> ../../archive/cloud.madand1.info/fullchain1.pem
lrwxrwxrwx 1 root root  45 Nov 10 10:19 privkey.pem -> ../../archive/cloud.madand1.info/privkey1.pem
root@ford-raptor:~# 

</pre>

h2. Entrega la configuración de nginx (los dos ficheros de cada virtualhost) para que funcione HTTPS y la redirección.

h3. Nextcloud
<pre>
root@ford-raptor:~# cat  /etc/nginx/sites-available/nextcloud 
upstream php-handler {
    server unix:/run/php/php8.2-fpm.sock; # Asegúrate de que coincida con la versión de PHP
}

server {

    server_name cloud.madand1.info;

    client_max_body_size 512M;
    client_body_timeout 300s;
    fastcgi_buffers 64 4K;

    root /var/www/nextcloud/nextcloud; # Asegúrate de que esta sea la ruta correcta
    index index.php index.html /index.php$request_uri;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_read_timeout 600;
    }

    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    add_header Referrer-Policy "no-referrer" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Download-Options "noopen" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Permitted-Cross-Domain-Policies "none" always;
    add_header X-Robots-Tag "none" always;
    add_header X-XSS-Protection "1; mode=block" always;

    fastcgi_hide_header X-Powered-By;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ^~ /.well-known {
        location = /.well-known/carddav { return 301 /remote.php/dav/; }
        location = /.well-known/caldav { return 301 /remote.php/dav/; }
        location /.well-known/acme-challenge { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation { try_files $uri $uri/ =404; }
        return 301 /index.php$request_uri;
    }

    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/) { return 404; }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) { return 404; }

    location ~ \.php(?:$|/) {
        rewrite ^/(?!index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|oc[ms]-provider/.+|.+/richdocumentscode/proxy) /index.php$request_uri;

        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;

        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;

        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ \.(?:css|js|svg|gif|png|jpg|ico)$ {
        try_files $uri /index.php$request_uri;
        expires 6M;
        access_log off;
    }

    location ~ \.woff2?$ {
        try_files $uri /index.php$request_uri;
        expires 7d;
        access_log off;
    }

    location /remote {
        return 301 /remote.php$request_uri;
    }

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/cloud.madand1.info/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/cloud.madand1.info/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}


server {
    if ($host = cloud.madand1.info) {
        return 301 https://$host$request_uri; #Redirigir HTTP a HTTPS
    } # managed by Certbot


    listen 80;
    listen [::]:80;

    server_name cloud.madand1.info;
    return 404; # managed by Certbot


}
</pre>

h3. Drupal

<pre>

root@ford-raptor:/etc/nginx/sites-available# cat drupal
server {
    listen 80;
    server_name www.madand1.info;
    root /var/www/drupla/drupal;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    return 301 https://$host$request_uri;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock; # Asegúrate de que esta línea coincida con tu versión de PHP
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}

</pre>

h2. Entrega la configuración del cron donde se ve que se hará la renovación cada 3 meses.


<pre>
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command

0 0 1 */3 * /usr/bin/certbot renew –quiet
</pre>


h2. Captura de pantalla accediendo a las dos páginas con https. Captura de pantalla con los detalles del certificado.

![alt text](img/image13.png)

![alt text](img/image10.png)

h2. Captura de pantalla donde se vea el cliente de NextCloud conectado por https.

![alt text](image14.png)