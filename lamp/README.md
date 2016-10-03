## LAMP Server

Ein LAMP/ LNMP Server steht für Linux Apache/ NGINX MySQL PHP. Dies sind die Grundpakete die auf fast Linux System wieder zufinden sind.

## LAMP/ LNMP Installation

In meinem Fall werde ich den LNMP Server installieren, da dieser auf einem bereits ausgereizten System besser und stabiler läuft.


### Paketinstallation:

    apt-get install mysql-server nginx php5-fpm

###### Alternative PHP5 Module:

    php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl

### Konfiguration:

Während der Installation wird für MySQL ein Passwort vergeben, dies sollte nicht vergessen werden, da es für den Root Zugriff des MySQL Servers dient.

Wie man einen NGINX mit PHP 5 einrichtet kann man [hier](https://github.com/mc8051/anleitungen/tree/master/nginx_php5) Nachlesen.
