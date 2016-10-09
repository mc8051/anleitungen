## Mailserver einrichten

Diese Anleitung wurde im September 2016 erstellt und ist zu diesem Zeitpunkt aktuell.  
Verwendet werden: Dovecot, Postfix, MySQL, PostfixAdmin, Sieve Filter und Spamassasin   
*Diese Anleitung bezieht sich auf das OS Debian Jessie.*

### Mindestvoraussetzungen:
* RAM: 512MB+ (Da clamav als AntiViren System mitläuft)
* 1 Core
* 10GB+ (je nach dem, wie viele Postfächer es geben wird)
* LAMP/ LNMP Server
* Domain mit MX Record
* Statische IP Adresse, die bestenfalls nicht als Spam gekennzeichnet ist!
  - Uberprüfbar bei [Blacklist Checks](http://whatismyipaddress.com/blacklist-check).
* rDNS Eintrag
* (vertauenswürdiges) SSL-Zertifikat mit 2048bit Verschlüsselung oder höher
  - Bspw. von [StartSSL](https://www.startssl.com/)
  - Seit dem Oktober 2016 sollte [Let's Encrypt](https://github.com/mc8051/anleitungen/tree/master/ssl_certificate/letsencrypt) verwendet werden, da Mozilla und Apple StartSSL nicht mehr so wirklich vertauen
* einen [LAMP](https://github.com/mc8051/anleitungen/tree/master/lamp) Server

---

Bevor andere Pakete installiert werden einmal ein Update ausführen.

    apt-get update && apt-get upgrade

---

### Mail Server Installation
#### Voreinstellung:
Mail Account einrichten mit der User ID 6000

    useradd -u 6000 vmail -d /var/vmail/
    mkdir /var/vmail/
    chown vmail:vmail /var/vmail/

Datenbank Benutzer erstellen mit eigener Datenbank

    CREATE USER 'mail'@'%' IDENTIFIED WITH mysql_native_password;GRANT USAGE ON *.* TO 'mail'@'%' REQUIRE NONE WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0;SET PASSWORD FOR 'mail'@'%' = 'meinPasswort';CREATE DATABASE IF NOT EXISTS `mail`;GRANT ALL PRIVILEGES ON `mail`.* TO 'mail'@'%';

Vorgefertigte Datein herrunterladen (aus dem Ordner config).  
In den Datein muss dass Passwort (Platzhalter: "EUER_PASSWORT") mit dem MYSQL User Passwort ersetzt werden.

#### Paketinstallation:
    apt-get install dovecot-common dovecot-imap dovecot-pop3d postfix postfix-mysql

Bei der Installation möchte Postfix eine Konfiguration haben, diese sieht wie folgt aus.
- Allgemeine Art: *Internet-Site*
- System-E-Mail-Name: *Domain Name*

#### Konfiguration von Postfix:
Hochladen der geänderten Vorgefertigte Datein von postfix.   
Anpassen der Rechte

    chown 600 /etc/postfix/mysql-* && chmod g+r /etc/postfix/mysql-*

Bearbeiten der */etc/postfix/main.cf*

    nano /etc/postfix/main.cf

Und komplett ersetzten. Bitte komplett durchlesen und dabei **mydomain** ersetzten!

    smtpd_banner = $myhostname ESMTP $mail_name (Debian/GNU)
    biff = no
    append_dot_mydomain = no
    readme_directory = no

    # TLS parameters
    smtpd_tls_cert_file=/etc/ssl/mydomain/mydomain.crt
    smtpd_tls_key_file=/etc/ssl/mydomain/mydomain.key
    smtpd_tls_CAfile = /etc/ssl/mydomain/root.crt
    smtpd_use_tls=yes

    # disable SSLv3
    smtpd_tls_mandatory_protocols=!SSLv2,!SSLv3
    smtp_tls_mandatory_protocols=!SSLv2,!SSLv3
    smtpd_tls_protocols=!SSLv2,!SSLv3
    smtp_tls_protocols=!SSLv2,!SSLv3

    smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
    myhostname = mydomain
    alias_maps = hash:/etc/aliases
    alias_database = hash:/etc/aliases
    myorigin = /etc/mailname
    mydestination = mydomain, localhost.net, , localhost
    relayhost = 
    mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
    mailbox_size_limit = 0
    recipient_delimiter = +
    inet_interfaces = all

    # a bit more spam protection
    disable_vrfy_command = yes

    # Auth
    smtpd_sasl_type=dovecot
    smtpd_sasl_path=private/auth_dovecot
    smtpd_sasl_auth_enable = yes
    smtpd_sasl_authenticated_header = yes
    broken_sasl_auth_clients = yes

    proxy_read_maps = $local_recipient_maps $mydestination $virtual_alias_maps $virtual_alias_domains $virtual_mailbox_maps $virtual_mailbox_domains $relay_recipient_maps $relay_domains $canonical_maps $sender_canonical_maps $recipient_canonical_maps $relocated_maps $transport_maps $mynetworks $smtpd_sender_login_maps

    smtpd_sender_login_maps = proxy:mysql:/etc/postfix/mysql-sender-login-maps.cf

    smtpd_sender_restrictions = reject_authenticated_sender_login_mismatch
            reject_unknown_sender_domain

    smtpd_recipient_restrictions = permit_sasl_authenticated
            permit_mynetworks
            reject_unauth_destination


    # Virtual mailboxes
    virtual_alias_maps = proxy:mysql:/etc/postfix/mysql-virtual-alias-maps.cf 
    virtual_mailbox_base = /var/vmail/ 
    virtual_mailbox_domains = proxy:mysql:/etc/postfix/mysql-virtual-domains-maps.cf 
    virtual_mailbox_limit = 0 
    virtual_mailbox_maps = proxy:mysql:/etc/postfix/mysql-virtual-mailbox-maps.cf 
    virtual_minimum_uid = 104 
    virtual_transport = dovecot
    local_transport = virtual
    virtual_uid_maps = static:6000 
    virtual_gid_maps = static:6000 
    dovecot_destination_recipient_limit = 1
    

Bearbeiten der */etc/postfix/master.cf*

    nano /etc/postfix/master.cf

Und komplett ersetzten. Bitte komplett durchlesen!

    #
    # Postfix master process configuration file.  For details on the format
    # of the file, see the master(5) manual page (command: "man 5 master" or
    # on-line: http://www.postfix.org/master.5.html).
    #
    # Do not forget to execute "postfix reload" after editing this file.
    #
    # ==========================================================================
    # service type  private unpriv  chroot  wakeup  maxproc command + args
    #               (yes)   (yes)   (yes)   (never) (100)
    # ==========================================================================
    smtp      inet  n       -       -       -       -       smtpd
    #smtp      inet  n       -       -       -       1       postscreen
    #smtpd     pass  -       -       -       -       -       smtpd
    #dnsblog   unix  -       -       -       -       0       dnsblog
    #tlsproxy  unix  -       -       -       -       0       tlsproxy
    submission inet n       -       -       -       -       smtpd
    #  -o syslog_name=postfix/submission
    #  -o smtpd_tls_security_level=encrypt
    #  -o smtpd_sasl_auth_enable=yes
    #  -o smtpd_reject_unlisted_recipient=no
    #  -o smtpd_client_restrictions=$mua_client_restrictions
    #  -o smtpd_helo_restrictions=$mua_helo_restrictions
    #  -o smtpd_sender_restrictions=$mua_sender_restrictions
    #  -o smtpd_recipient_restrictions=
    #  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
    #  -o milter_macro_daemon_name=ORIGINATING
    smtps     inet  n       -       -       -       -       smtpd
    #  -o syslog_name=postfix/smtps
    -o smtpd_tls_wrappermode=yes
    #  -o smtpd_sasl_auth_enable=yes
    #  -o smtpd_reject_unlisted_recipient=no
    #  -o smtpd_client_restrictions=$mua_client_restrictions
    #  -o smtpd_helo_restrictions=$mua_helo_restrictions
    #  -o smtpd_sender_restrictions=$mua_sender_restrictions
    #  -o smtpd_recipient_restrictions=
    #  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
    #  -o milter_macro_daemon_name=ORIGINATING
    #628       inet  n       -       -       -       -       qmqpd
    pickup    unix  n       -       -       60      1       pickup
    cleanup   unix  n       -       -       -       0       cleanup
    qmgr      unix  n       -       n       300     1       qmgr
    #qmgr     unix  n       -       n       300     1       oqmgr
    tlsmgr    unix  -       -       -       1000?   1       tlsmgr
    rewrite   unix  -       -       -       -       -       trivial-rewrite
    bounce    unix  -       -       -       -       0       bounce
    defer     unix  -       -       -       -       0       bounce
    trace     unix  -       -       -       -       0       bounce
    verify    unix  -       -       -       -       1       verify
    flush     unix  n       -       -       1000?   0       flush
    proxymap  unix  -       -       n       -       -       proxymap
    proxywrite unix -       -       n       -       1       proxymap
    smtp      unix  -       -       -       -       -       smtp
    relay     unix  -       -       -       -       -       smtp
    #       -o smtp_helo_timeout=5 -o smtp_connect_timeout=5
    showq     unix  n       -       -       -       -       showq
    error     unix  -       -       -       -       -       error
    retry     unix  -       -       -       -       -       error
    discard   unix  -       -       -       -       -       discard
    local     unix  -       n       n       -       -       local
    virtual   unix  -       n       n       -       -       virtual
    lmtp      unix  -       -       -       -       -       lmtp
    anvil     unix  -       -       -       -       1       anvil
    scache    unix  -       -       -       -       1       scache
    #
    # ====================================================================
    # Interfaces to non-Postfix software. Be sure to examine the manual
    # pages of the non-Postfix software to find out what options it wants.
    #
    # Many of the following services use the Postfix pipe(8) delivery
    # agent.  See the pipe(8) man page for information about ${recipient}
    # and other message envelope options.
    # ====================================================================
    #
    # maildrop. See the Postfix MAILDROP_README file for details.
    # Also specify in main.cf: maildrop_destination_recipient_limit=1
    #
    maildrop  unix  -       n       n       -       -       pipe
    flags=DRhu user=vmail argv=/usr/bin/maildrop -d ${recipient}
    #
    # ====================================================================
    #
    # Recent Cyrus versions can use the existing "lmtp" master.cf entry.
    #
    # Specify in cyrus.conf:
    #   lmtp    cmd="lmtpd -a" listen="localhost:lmtp" proto=tcp4
    #
    # Specify in main.cf one or more of the following:
    #  mailbox_transport = lmtp:inet:localhost
    #  virtual_transport = lmtp:inet:localhost
    #
    # ====================================================================
    #
    # Cyrus 2.1.5 (Amos Gouaux)
    # Also specify in main.cf: cyrus_destination_recipient_limit=1
    #
    #cyrus     unix  -       n       n       -       -       pipe
    #  user=cyrus argv=/cyrus/bin/deliver -e -r ${sender} -m ${extension} ${user}
    #
    # ====================================================================
    # Old example of delivery via Cyrus.
    #
    #old-cyrus unix  -       n       n       -       -       pipe
    #  flags=R user=cyrus argv=/cyrus/bin/deliver -e -m ${extension} ${user}
    #
    # ====================================================================
    #
    # See the Postfix UUCP_README file for configuration details.
    #
    uucp      unix  -       n       n       -       -       pipe
    flags=Fqhu user=uucp argv=uux -r -n -z -a$sender - $nexthop!rmail ($recipient)
    #
    # Other external delivery methods.
    #
    ifmail    unix  -       n       n       -       -       pipe
    flags=F user=ftn argv=/usr/lib/ifmail/ifmail -r $nexthop ($recipient)
    bsmtp     unix  -       n       n       -       -       pipe
    flags=Fq. user=bsmtp argv=/usr/lib/bsmtp/bsmtp -t$nexthop -f$sender $recipient
    scalemail-backend unix	-	n	n	-	2	pipe
    flags=R user=scalemail argv=/usr/lib/scalemail/bin/scalemail-store ${nexthop} ${user} ${extension}
    mailman   unix  -       n       n       -       -       pipe
    flags=FR user=list argv=/usr/lib/mailman/bin/postfix-to-mailman.py
    ${nexthop} ${user}

    # Eigen Konfiguration
    dovecot   unix  -       n       n       -       -       pipe
    flags=DRhu user=vmail:vmail argv=/usr/lib/dovecot/deliver -d ${recipient}

Überprüfen der Konfiguration

    postfix check

#### Konfiguration von Dovecot:
Hochladen der geänderten Vorgefertigte Datein von dovecot.  
Anpassen der Rechte

    chown 600 /etc/dovecot/dovecot-mysql.conf && chmod g+r /etc/dovecot/dovecot-mysql.conf

Bearbeiten der */etc/dovecot/dovecot.conf*

    nano /etc/dovecot/dovecot.conf

Und komplett ersetzten. Bitte komplett durchlesen und dabei **mydomain** ersetzten!

    auth_mechanisms = plain login
    log_timestamp = "%Y-%m-%d %H:%M:%S "
    mail_home = /var/vmail/%d/%n

    passdb {
        args = /etc/dovecot/dovecot-mysql.conf
        driver = sql
    }
    protocols = imap pop3
    service auth {
        unix_listener /var/spool/postfix/private/auth_dovecot {
            group = postfix
            mode = 0660
            user = postfix
        }
        unix_listener auth-master {
            mode = 0600
            user = vmail
        }
        user = root
    }
    listen = *
    ssl = yes
    ssl_cert = </etc/ssl/mydomain/mydomain.crt
    ssl_key = </etc/ssl/mydomain/mydomain.key
    ssl_ca = </etc/ssl/mydomain/root.crt
    ssl_protocols = !SSLv2 !SSLv3
    userdb {
        args = /etc/dovecot/dovecot-mysql.conf
        driver = sql
    }

    protocol pop3 {
        pop3_uidl_format = %08Xu%08Xv
        pop3_client_workarounds = oe-ns-eoh
    }

    lda_mailbox_autosubscribe = yes
    lda_mailbox_autocreate = yes
    protocol lda {
        auth_socket_path = /var/run/dovecot/auth-master
        postmaster_address = info@mydomain
    }

#### Final
    service spamassassin restart
    service amavis restart
    service postfix restart
    service dovecot restart

---

### PostfixAdmin Installation
#### Paketinstallation:
    apt-get install php5-imap 

Letze Version von [Postfix](https://sourceforge.net/projects/postfixadmin/files/postfixadmin/) herrunterladen und auf den Server hochladen in */var/www/html/*.

    tar xzvf postfix.tar.gz

#### Konfiguration von PostfixAdmin:
Rechte gewähren für den Webserver

    chown www-data:www-data -R /var/www/html/postfix/

Bearbeiten der postfix Config

    nano /var/www/html/postfix/config.inc.php

Bearbeiten folgender Einträge, falls nicht vorhanden hinzufügen.

    $CONF['configured'] = true;
    $CONF['default_language'] = 'de';
    $CONF['database_type'] = 'mysqli';
    $CONF['database_host'] = 'localhost';
    $CONF['database_user'] = 'mail';
    $CONF['database_password'] = 'EUER_PASSWORT';
    $CONF['database_name'] = 'mail';
    $CONF['admin_email'] = 'info@example.tld';
    $CONF['encrypt'] = 'md5';
    $CONF['domain_path'] = 'YES'; // muss hinzugefügt werden
    $CONF['domain_in_mailbox'] = 'NO'; // muss hinzugefügt werden

Aufrufen des Setups im Browser (bspw. *www.example.tld/postfix/setup.php*)  
Hier sollte bei dem Check alles bestanden sein, falls nicht einfach Nachinstallieren.  

Setup-Passwort generieren lassen und den Hashwert in die *config.inc.php* einfügen bei *$CONF['setup_password']*.

Zum Schluss nur noch einen Admin Account erstellen.

Nun sollte postfix.admin erreichbar sein und man kann sich einloggen.

In Postfix sollte nun die Domain unter *Domain hinzufügen* eingetragen werden.

Zudem sollte man bereits jetzt schon unter Virtual Liste ein EMail Account erstellen.

---

### Spam Filter Installation

#### Paketinstallation:
    apt-get install spamassassin razor pyzor rsyslog
    apt-get install amavisd-new
    apt-get install dovecot-sieve dovecot-lmtpd
    apt-get install clamav clamav-daemon

    /etc/init.d/clamav-freshclam restart
    /etc/init.d/clamav-daemon restart

    apt-get install arj bzip2 cabextract cpio file gzip nomarch pax unzip zoo zip zoo

    razor-admin -create
    razor-admin -register
    pyzor discover

#### spamassassin
/etc/default/spamassassin  
ENABLED=1 und CRON=1 setzen:

    # systemctl enable spamassassin.service
    # Change to "1" to enable spamd on systems using sysvinit:
    ENABLED=1

    # Cronjob
    # Set to anything but 0 to enable the cron job to automatically update
    # spamassassin's rules on a nightly basis
    CRON=1

#### amavis
/etc/amavis/conf.d/50-user

    use strict;

    #
    # Place your configuration directives here.  They will override those in
    # earlier files.
    #
    # See /usr/share/doc/amavisd-new/ for documentation and examples of
    # the directives you can use in this file
    #

    # @local_domains_acl = ( ".$mydomain");

    @bypass_virus_checks_maps = (
       \%bypass_virus_checks, \@bypass_virus_checks_acl, \$bypass_virus_checks_re);

    @bypass_spam_checks_maps = (
       \%bypass_spam_checks, \@bypass_spam_checks_acl, \$bypass_spam_checks_re);

    $sa_spam_modifies_subj = 1;

    $undecipherable_subject_tag = undef;

    $sa_spam_subject_tag = '***SPAM*** '; # Spam in den Betreff schreiben
    $sa_tag_level_deflt  = undef;
    $sa_tag2_level_deflt = 5;
    $sa_kill_level_deflt = 10; # Dieser Wert macht sich in einem Produktiv System sehr bewährt
    $sa_dsn_cutoff_level = 10;   # spam level beyond which a DSN is not sent

    #------------ Do not modify anything below this line -------------
    1;  # ensure a defined return

#### sieve
    mkdir /var/vmail/sieve/
    chown -R mail:mail /var/vmail/sieve/

Default Sieve Script /var/lib/dovecot/sieve/default.sieve

    require ["fileinto"];

    if header :contains "X-Spam-Flag" "YES"
    {
      fileinto "Junk";
    }

#### dovecot
/etc/dovecot/local.conf

    # Protokol LMTP und Sieve hinzufügen
    protocols = imap pop3 lmtp sieve

    # Vorraussetzung fuer Spamhandling

    protocol lmtp {
      # Space separated list of plugins to load (default is global mail_plugins).
      postmaster_address = webmaster@mydomain.tld
      mail_plugins = $mail_plugins sieve
    }

    plugin {
        sieve_before = /var/lib/dovecot/sieve/default.sieve
        sieve_dir = /var/vmail/sieve/scripts/%u
        sieve = /var/vmail/sieve/%u.sieve
    }

    protocol sieve {
        managesieve_max_line_length = 65536
        managesieve_implementation_string = dovecot
        log_path = /var/log/dovecot-sieve-errors.log
        info_log_path = /var/log/dovecot-sieve.log
    }

    service lmtp {
        unix_listener /var/spool/postfix/private/dovecot-lmtp {
            group = postfix
            mode = 0600
            user = postfix
        }
    }

#### postfix
/etc/postfix/main.cf

    # Spamfilter
    content_filter=smtp-amavis:[127.0.0.1]:10024
    mailbox_transport = lmtp:unix:private/dovecot-lmtp

**Hinweis**: check_relay_domains muss jetzt reject_unauth_destination sein

In */etc/postfix/main.cf* muss *local_transport = virtual* entfernt werden.

/etc/postfix/master.cf

**Hinweis**: Vorsicht beim Einfuegen (copy-past) in master.cf mit emacs unter linux wird die Einrueckung recht komisch.

    # Spambehandlung ANFANG

    smtp-amavis     unix    -       -       -       -       2       smtp
            -o smtp_data_done_timeout=1200
            -o smtp_send_xforward_command=yes
            -o disable_dns_lookups=yes
            -o max_use=20
            -o smtp_tls_security_level=none
    127.0.0.1:10025 inet    n       -       -       -       -       smtpd
            -o content_filter=
            -o local_recipient_maps=
            -o relay_recipient_maps=
            -o smtpd_restriction_classes=
            -o smtpd_delay_reject=no
            -o smtpd_client_restrictions=permit_mynetworks,reject
            -o smtpd_helo_restrictions=
            -o smtpd_sender_restrictions=
            -o smtpd_recipient_restrictions=permit_mynetworks,reject
            -o smtpd_data_restrictions=reject_unauth_pipelining
            -o smtpd_end_of_data_restrictions=
            -o mynetworks=127.0.0.0/8
            -o smtpd_error_sleep_time=0
            -o smtpd_soft_error_limit=1001
            -o smtpd_hard_error_limit=1000
            -o smtpd_client_connection_count_limit=0
            -o smtpd_client_connection_rate_limit=0
            -o receive_override_options=no_header_body_checks,no_unknown_recipient_checks
            -o smtp_tls_security_level=none

    # Spambehandlung ENDE


    # -o eintraege sind fuer spamfilter
    pickup    fifo  n       -       -       60      1       pickup
        -o content_filter=
        -o receive_override_options=no_header_body_checks

#### clamav
/etc/clamav/clamd.conf

    # Anpassung von Martin Enders
    AllowSupplementaryGroups true
    usermod -a -G amavis clamav

#### Final
    service spamassassin restart
    service amavis restart
    service postfix restart
    service dovecot restart

[*Quelle*](https://gist.github.com/MartinEnders/c3bbbc6ce5145658451d)

---

### DNS Einstellung
#### Hostnamen anpassen
Der Hostname muss in eure Domain geändert werden

    nano /etc/hostname
#### MX-Record
Folgende DNS Einstellungen müssen getätigt werden
<img src="https://github.com/mc8051/anleitungen/raw/master/mailserver/postfix_dovecot/screenshots/mx-cloudflare.png" alt="MX Record" style="width: 50%;"/>

#### SPF Record
Der SPF Record kann mit dem [SPF Record Generator](http://www.spf-record.de/generator) erstellt werden. Der Eintrag soltle jedoch als TXT eingetragen werden.

#### rDNS
Die IP Adresse des Servers MUSS die Domain auflösen. Dies kann man meist im Webinterface des Hosting Providers ändern.

---

### Einrichtung eines EMail Kontos
<img src="https://github.com/mc8051/anleitungen/raw/master/mailserver/postfix_dovecot/screenshots/thunderbird-settings.png" alt="Thunderbird" style="width: 50%;"/>



## Deaktivieren des Anti-Viren Programms ClamAV

Bearbeiten der amvis User Config

    nano /etc/amavis/conf.d/50-user

Auskommentieren der folgenden zwei Zeilen

    @bypass_virus_checks_maps = (
        \%bypass_virus_checks, \@bypass_virus_checks_acl, \$bypass_virus_checks_re);

So dass es wie folgt aussieht

    # @bypass_virus_checks_maps = (
    #    \%bypass_virus_checks, \@bypass_virus_checks_acl, \$bypass_virus_checks_re);

    @bypass_spam_checks_maps = (
    \%bypass_spam_checks, \@bypass_spam_checks_acl, \$bypass_spam_checks_re);

Amvis Service neustarten

    service amavis restart

Stoppen der CalmAV Services

    service clamav-daemon stop
    service clamav-freshclam stop

CalmAV aus dem Autostart nehmen

    update-rc.d -f clamav-daemon remove
    update-rc.d -f clamav-freshclam remove

*Für's rückgängig machen*

    update-rc.d clamav-daemon defaults
    update-rc.d clamav-freshclam defaults
    service clamav-daemon start
    service clamav-freshclam start