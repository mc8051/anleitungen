## Rainloop Webmailer installieren

Folgende Pakete werden gebraucht für Rainloop

    apt-get install php5 php5-mysql curl libcurl3 libcurl3-dev php5-curl php5-json

  
**_[Git Repository der Community Edition](https://github.com/RainLoop/rainloop-webmail)_**  
**_[Download Seite](http://www.rainloop.net/downloads/)_**
### Standalone
Downloaden der aktuellsten Rainloop Community Edition

    cd /var/www/html
    curl -O http://repository.rainloop.net/v2/webmail/rainloop-community-latest.zip
    unzip rainloop-latest.zip
    rm rainloop-latest.zip


### NextCloud/ OwnCloud
Downloaden der aktuellen Community Edtion für die NextCloud/ OwnCloud

    cd /var/www/html/owncloud/apps
    mkdir rainloop
    curl -O http://repository.rainloop.net/v2/other/owncloud/rainloop.zip

**_Hinweis:_** Die Rainloop Daten finden sich in der Cloud Storage wieder als neuen Benutzer Ordner!


Rechte vergeben

    chown -R www-data:www-data /var/www/html/rainloop/
    find . -type d -exec chmod 755 {} \;
    find . -type f -exec chmod 644 {} \;


Administations Seite aufrufen -> User=admin und Passwort=12345

    http://host/rainloop/?admin

In der Cloud Version findet man die Rainloop Administation als Link in der Cloud Administation

Hier können belibiege Einstellungen vorgenommen werden. Wichtig sind die Einstellungen "Admin Kennwort", "Datenbank Verbindung" und "Domains".

Die Domain Einstellungen für [diese Mailserver](https://github.com/mc8051/anleitungen/tree/master/mailserver/postfix_dovecot) Konfiguration lauten:  
* IMAP: SSL/TLS Port 993  
* SMTP: SSL/TLS Port 465  
* SIEVE: STARTTLS Port 4190  