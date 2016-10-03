## [PrestaShop](https://www.prestashop.com/de/) einrichten

PrestaShop eine OpenSource Software aus Frankreich für kleine und Mittelständische Unternehmen.

Die Vorteile von PrestaShop
* Großer Funktionsumfang
* Moderne und Individuelle Gestaltung
* Umfrangreiche Produktseiten
* Einfache Verwaltung (Back Office)
* Durch Bootstrap einfaches responsive Design

Die Systemanforderungen sind
* eigene Domain
* Webserver: Apache 1.3, Apache 2.x, Nginx oder Microsoft IIS
* PHP 5.2+
* MySQL 5.0+ installiert, mit erstellter Datenbank


Die Installation ist recht einfach gehalten und kann auf jedem bereits bestehenden System eingerichtet werden.

Downloaden der aktuellen Version im WWW Verzeichnis

    cd /va/www/html/
    curl -O https://download.prestashop.com/download/releases/prestashop_1.6.1.7_de.zip
    unzip prestashop_1.6.1.7_de.zip
    mv prestashop_1.6.1.7_de shop

Nun kann im Browser die Installation aufgerufen werden und durchgeklickt werden. Dabei werden die Systemanforderungen geprüft und Datenbanken angelegt.  
Am Ende der installation muss im Back Office der Demo Modus deaktiviert werden.

Weitere Infos findet man in der [Dokumentation](http://doc.prestashop.com/display/PS16/Installing+PrestaShop).
