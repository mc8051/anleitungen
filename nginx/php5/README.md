## NGINX mit PHP5

NGINX ist ein stabieler Open Source HTTP-Server. Er besitzt ein geringen Resourcenverbauch und hat einen großen Funktionsumfrang durch Module wie z.B. einen [RTMP Server](https://github.com/mc8051/anleitungen/tree/master/nginx_rtmp).

### Paketinstallation

    apt-get install nginx php5-fpm

### Konfiguration
Der Server sollte nun im Browser erreichbar sein. Um allerdings PHP zur Verfügung zu stellen muss die /etc/nginx/sites-available/default Konfiguration bearbeitet werden.

    server {
        listen 80 default_server;
        listen [::]:80 default_server;
    
        root /var/www/html;
        # index.php hinzufügen
        index index.php index.html index.htm index.nginx-debian.html;
        server_name _;
    
        location / {
                try_files $uri $uri/ =404;
        }
    
        # PHP Datein müssen interpretiert werden
        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
    }

### Optionales Rewrite
Sollte der NGINX Server von außen bspw. per 8081 erreibar sein, aber intern auf 80 geroutet werden, so geht dies spätestens bei einem rewrite schief.  
Dieser findet schon statt, wenn man keinen Slash am Ende eines Verzeichnisses anfügt.

Als Abhilfe dafür gibt es folgende Rewriterule.  
**Achtung** falls kein http_host gesendet wird, funktioniert dieser Rewrite nicht!

    if (-d $request_filename) {
            rewrite [^/]$ $scheme://$http_host$uri/ permanent;
    }
