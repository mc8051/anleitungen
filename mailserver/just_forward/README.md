## Postfix Catch-All

Wenn man eine Domain besitzt, bei der man gerne einen MailServer zu verfügung stellen möchte, bei dem es aber reicht EMails anzunehmen und dann an eine bereits existierende EMail weiter zu schicken, so kann man postfix auf "Catch-All" einrichten. Somit nimmt Postfix alle oder nur bestimmte EMails an und sendet sie an einen Alias weiter.

Öffnen der Postfix *main.cf*

    nano /etc/postfix/main.cf

Folgende zwei Einräge hinzufügen/ bearbeiten. (**mydomain.tld** mit der Domain ersetzten)

    virtual_alias_domains = mydomain.tld 
    virtual_alias_maps = hash:/etc/postfix/virtual

Weiterleitungen bearbeiten

    nano /etc/postfix/virtual

EMails die an name@mydomain.tld ankommen werden an name@gmail.com weitergeleitet

    name@mydomain.tld name@gmail.com name@live.com

Ebenso kann man ein "Catch-All" machen, also ALLE EMails annehmen

    @mydomain.tld name@gmail.com

Nach jeder änderung muss ein postmap durchgeführt werden, damit Postfix die Änderung mitbekommt

    postmap /etc/postfix/virtual
    service postfix reload