## Linux Monitoring Tool

![Screenshot](https://raw.githubusercontent.com/mc8081/anleitungen/master/linux_dash/screen.png)

linux-dash ist ein Open Source Monitoring Tool. Es ist ziemlich einfach aufzusetzten.

Herunterladen und entpacken

    cd /var/www/html/
    git clone https://github.com/afaqurk/linux-dash.git
    mv linux-dash dash

Linux Dash Config erstellen für NGINX

    server {
      listen 80;
      server_name dash.example.tld;

      root /var/www/html/dash;
      index index.html index.php;
      access_log /var/log/nginx/access.log;
      error_log /var/log/nginx/error.log;

      location ~* \.(?:xml|ogg|mp3|mp4|ogv|svg|svgz|eot|otf|woff|ttf|css|js|jpg|jpeg|gif|png|ico)$ {
        try_files $uri =404;
        expires max;
        access_log off;
        add_header Pragma public;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
      }

      location / {
        index index.html index.php;
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswd;    
      }

      location ~ \.php(/|$) {
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        if (!-f $document_root$fastcgi_script_name) {
          return 404;
        }
        try_files $uri $uri/ /index.php?$args;
        include fastcgi_params;
      }
    }

Zum Sichern des Verzeichnisses wird von Apache2 htpasswd eingesetzt. Um eine htpasswd erstellen zu können werden die apache2-utils gebraucht.

    apt-get install apache2-utils

Nun nur noch einen Benutzer erstellen, dabei wird man nach einem Passwort gefragt, welches verschlüsselt abgespeichert wird.

    htpasswd -c /etc/nginx/.htpasswd <username>
