## Kostenloses SSL Zertifikat

Seit dem Betrieb von Let’s Encrypt (unter anderem gesponsort von Mozilla, Facebook und Cisco) kann jeder sich sein eigenes vertrauenswürdiges SSL Zertifikat erstellen - kostenlos. Dies kann man mit jeder eigenen Domain abwickeln oder auch mit bestimmten Public Domains.  
Da die Zertifikate immer nur für 90 Tage gelten, muss ein Bot dies für uns erlediegen. Diese Anleitung nutzt den "Certbot".


### Certbot ACME-Client
    wget https://dl.eff.org/certbot-auto
    chmod a+x certbot-auto
    ./certbot-auto

**Das erstmalige erstellen eines Zertifikat für eine Domain**  
Der Certbot erstellt seinen eigenen WebServer, so dass er ohne Probleme die Domainüberprüfung durchführen kann.

    ./certbot-auto certonly --standalone --email email@mydomain.tld --agree-tos --rsa-key-size 4096 -d mydomain.tld -d www.mydomain.tld -d mail.mydomain.tld

Ist dies erfolgreich abgeschlossen, so liegen die Zertifikate in */etc/letsencrypt/live/[domain]*, dort sind folgende vier Datein als Symbolischer Link hinterlegt:
* cert.pem (Das öffentliche Zertifikat in Reinform)
* chain.pem (Root Zertifikat aus der sog. Keychain)
* fullchain.pem (entspricht cert.pem + chain.pem)
* privkey.pem (Der private Schlüssel)


### Cronjob für Renew
Testdurchlauf starten

    /path/to/certbot-auto renew --dry-run 


Wenn der Testdurchlauf ohne Probleme durchgelaufen ist, in *crontab -e*

    30 3 * * 1 service [webservice] stop && /path/to/certbot-auto renew --quiet && service [webservice] start && service postfix reload && service dovecot reload


### Neue Subdomain hinzufügen

    /path/to/certbot-auto certonly --standalone --email email@mydomain.tld --agree-tos --rsa-key-size 4096 -d new.domaintld --expand --renew-by-default

### Einbinden in einem WebServer oder Mailserver
**NGINX**

    ssl_certificate /etc/letsencrypt/live/[domain]/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/[domain]/privkey.pem;

**APACHE**

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/[domain]/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/[domain]/privkey.pem

**Postfix**

    smtpd_tls_cert_file=/etc/letsencrypt/live/[domain/cert.pem
    smtpd_tls_key_file=/etc/letsencrypt/live/[domain/privkey.pem
    smtpd_tls_CAfile = /etc/letsencrypt/live/[domain/chain.pem

