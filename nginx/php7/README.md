## Installation von PHP7

### Debian
**Source**: https://packages.sury.org/php/README.txt

    apt-get install apt-transport-https lsb-release ca-certificates
    wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list
    apt-get update


### Raspberry PI
**Source**: https://www.randombrick.de/raspberry-pi-nginx-und-php-installieren-und-einrichten/  
Raspbian basiert auf Debian Jessie und wird mit PHP 5.6 ausgeliefert. Das ist soweit auch gut aber zwischenzeitlich ist mit PHP 7.0 eine bessere Option auf dem Markt. PHP 7.0 wurde im Dezember 2015 vorgestellt und bringt erhebliche Verbesserungen in den Bereichen Leistung, Sprache und nutzt dabei deutlich weniger Speicher. Optimal für den Raspberry Pi.

Damit PHP 7.0 auf dem Raspberry Pi installiert werden kann, wird Zugriff auf die Entwicklerversion von Raspbian benötigt. Diese trägt den Codename stretch. Dazu ist die Datei sources.list abzuändern welche von Aptitude (apt-get) genutzt wird.

    sudo nano /etc/apt/sources.list

Unter der ersten Zeile die das Wort jessie enthält, ist folgende Zeile einzufügen:

    deb http://mirrordirector.raspbian.org/raspbian/ stretch main contrib non-free rpi
    
Gespeichert wird mit Strg + X, Y und dann Enter drücken.

Damit würden alle Installationen und Updates automatisch die neusten verfügbaren Dateien aus dem stretch Release ziehen. Diese sind jedoch nicht immer 100 Prozent stabil. Damit zukünftig keine instabilen Pakete installiert werden, werden automatisch alle Pakete gezwungen, das jessie Release mit einer höheren Priorität zu wählen.

    sudo nano /etc/apt/preferences

Hier ist folgendes einzufügen:

    Package: *
    Pin: release n=jessie
    Pin-Priority: 600

Gespeichert wird mit Strg + X, Y und dann Enter drücken.

Anschließend werden die Updates noch einmal ausgeführt mit:

    sudo apt-get update
    
Jetzt erfolgt die Installation von PHP 7.0 auf dem Raspberry Pi aus dem stretch Release inklusive aller PHP-Pakete.

    sudo apt-get install -t stretch php7.0 php7.0-curl php7.0-gd php7.0-fpm php7.0-cli php7.0-opcache php7.0-mbstring php7.0-xml php7.0-zip

Mit nachfolgendem Befehl lässt sich die Funktion von PHP testen:

    php -v

Anschließend sollte folgende oder ähnliche Ausgabe erscheinen:

    PHP 7.0.7-4 (cli) ( NTS )
    Copyright (c) 1997-2016 The PHP Group
    Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
    with Zend OPcache v7.0.6-dev, Copyright (c) 1999-2016, by Zend Technologies

Fertig. PHP 7.0 wurde erfolgreich auf dem Raspberry Pi installiert.

Wie bei PHP 7.0 gibt es im stretch Release eine neue und verbesserte Version mit der Nummer 1.9. Diese wird wie folgt installiert:
    
    sudo apt-get install -t stretch nginx

**Source**: https://www.randombrick.de/raspberry-pi-nginx-und-php-installieren-und-einrichten/

    server {
        listen 80 default_server;
        listen [::]:80 default_server;
    
        root /var/www/html;
        # index.php hinzufügen
        index index.php index.html index.htm;
        server_name _;
    
        location / {
                try_files $uri $uri/ =404;
        }
    
        # PHP Datein müssen interpretiert werden
        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }
    }