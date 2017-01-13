## GIT Server aufsetzen

gitea ist ein Fork des Open Source Projektes Gogs. Da Gogs nur langsam weiter entwickelt wird und nicht viele PR annimmt wurde die Abspaltung gitea erstellt. Diese ist eher Community bezogen. Beite Projekte sind auf Github unter der MIT Lizenz zu finden:
- [Gogs](https://github.com/gogits/gogs)
- [Gitea](https://github.com/go-gitea/gitea)

Das Ziel dieses Projektes ist einen leichten Git-Service zu schreiben. Dafür wird die Programmiersprache Go verwendet. Mit dem Designe orrientiert sich das Projekt an GitHub.

<img src="https://github.com/mc8051/anleitungen/raw/master/git/gutea_gogs/screenshots/explore.png" alt="explore page" style="width: 50%;"/>  

*[Explore](https://try.gitea.io/)*

### Link-Sammlung
- [Dokumentation](https://docs.gitea.io/en-us/)
- [Gitter](https://gitter.im/go-gitea/gitea/)
- [Start Stop Script Sammlung](https://github.com/go-gitea/gitea/tree/master/scripts)

### Einrichtung
Die Einrichtung kann auf einem Grät durchgefüght werden, welches mind. 2 CPUs und mind. 1GB RAM besitzt. Da Go auch die ARM Architektur unterstützt kann ohne Probleme ein Raspberry Pi verwendet werden.

Am einfachsten geht es über die bereits kompelierte Version von Gitea.

    apt-get install git
    adduser git && su git
    wget -O gitea https://dl.gitea.io/gitea/1.0.0/gitea-1.0.0-linux-amd64
    chmod +x gitea

Diese Befehle installieren Git und Gitea. Der Deamon wird unter dem Namen *git* laufen da es ein üblicher Name für den SSH login ist (*git@example.org*) 

Nun kann auch bereits der Webserver gestartet werden.

    ./gitea web

Im Anschluss wird die Installation beendet im Webbrowser, hiernach werden automatisch die nötigen Datein erstellt.  
Standardmäßig läuft der Server auf dem Port 3000.

**WICHTIG** Die ersten 1024 Ports können nur von Root Usern gebindet werden!

### init.d
In */etc/init.d/gitea*

    #! /bin/sh 
    ### BEGIN INIT INFO 
    # Provides:             gogs 
    # Required-Start:       $remote_fs $syslog 
    # Required-Stop:        $remote_fs $syslog 
    # Default-Start:        2 3 4 5 
    # Default-Stop:         0 1 6 
    # Short-Description:    Git repository manager Gogs
    # Description:          Starts and stops the self-hosted git repository manager Gogs
    ### END INIT INFO 

    # PATH should only include /usr/* if it runs after the mountnfs.sh script
    PATH=/sbin:/usr/sbin:/bin:/usr/bin
    DESC="Git repository manager Gitea"
    NAME=gitea

    # DIR is the gogs installation directory. Change if necessary.
    DIR=/home/git/
    DAEMON=$DIR$NAME
    DAEMON_ARGS="web"
    PIDFILE=/var/run/$NAME.pid
    SCRIPTNAME=/etc/init.d/$NAME

    # The daemon will be run as this user
    USER=git
    HOME=$(grep "^$USER:" /etc/passwd | cut -d: -f6)
    USER_ID=$(id -u $USER)
    GROUP_ID=$(id -g $USER)

    # Exit if the package is not installed
    [ -x "$DAEMON" ] || exit 0

    # Read configuration variable file if it is present
    [ -r /etc/default/$NAME ] && . /etc/default/$NAME

    # Load the VERBOSE setting and other rcS variables
    . /lib/init/vars.sh

    # Define LSB log_* functions.
    # Depend on lsb-base (>= 3.2-14) to ensure that this file is present
    # and status_of_proc is working.
    . /lib/lsb/init-functions

    #
    # Function that starts the daemon/service
    #
    do_start()
    {
            # Return
            #   0 if daemon has been started
            #   1 if daemon was already running
            #   2 if daemon could not be started
            export USER HOME
            start-stop-daemon --start --pidfile $PIDFILE --make-pidfile --user $USER --chdir $DIR --exec $DAEMON --test > /dev/null \
                    || return 1
            start-stop-daemon --start --background --pidfile $PIDFILE --make-pidfile --user $USER --chdir $DIR --chuid $USER_ID:$GROUP_ID --exec $DAEMON -- \
                    $DAEMON_ARGS \
                    || return 2
    }

    #
    # Function that stops the daemon/service
    #
    do_stop()
    {
            # Return
            #   0 if daemon has been stopped
            #   1 if daemon was already stopped
            #   2 if daemon could not be stopped
            #   other if a failure occurred
            start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE --name $NAME
            RETVAL="$?"
            [ "$RETVAL" = 2 ] && return 2
            # Wait for children to finish too if this is a daemon that forks
            # and if the daemon is only ever run from this initscript.
            # If the above conditions are not satisfied then add some other code
            # that waits for the process to drop all resources that could be
            # needed by services started subsequently.  A last resort is to
            # sleep for some time.
            start-stop-daemon --stop --quiet --oknodo --retry=0/30/KILL/5 --exec $DAEMON
            [ "$?" = 2 ] && return 2
            # Many daemons don't delete their pidfiles when they exit.
            rm -f $PIDFILE
            return "$RETVAL"
    }

    #
    # Function that sends a SIGHUP to the daemon/service
    #
    do_reload() {
            #
            # If the daemon can reload its configuration without
            # restarting (for example, when it is sent a SIGHUP),
            # then implement that here.
            #
            start-stop-daemon --stop --signal 1 --quiet --pidfile $PIDFILE --name $NAME
            return 0
    }

    case "$1" in
    start)
            [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC" "$NAME"
            do_start
            case "$?" in
                    0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
                    2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
            esac
            ;;
    stop)
            [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
            do_stop
            case "$?" in
                    0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
                    2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
            esac
            ;;
    status)
            status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
            ;;
    #reload|force-reload)
            #
            # If do_reload() is not implemented then leave this commented out
            # and leave 'force-reload' as an alias for 'restart'.
            #
            #log_daemon_msg "Reloading $DESC" "$NAME"
            #do_reload
            #log_end_msg $?
            #;;
    restart|force-reload)
            #
            # If the "reload" option is implemented then remove the
            # 'force-reload' alias
            #
            log_daemon_msg "Restarting $DESC" "$NAME"
            do_stop
            case "$?" in
            0|1)
                    do_start
                    case "$?" in
                            0) log_end_msg 0 ;;
                            1) log_end_msg 1 ;; # Old process is still running
                            *) log_end_msg 1 ;; # Failed to start
                    esac
                    ;;
            *)
                    # Failed to stop
                    log_end_msg 1
                    ;;
            esac
            ;;
    *)
            #echo "Usage: $SCRIPTNAME {start|stop|restart|reload|force-reload}" >&2
            echo "Usage: $SCRIPTNAME {start|stop|status|restart|force-reload}" >&2
            exit 3
            ;;
    esac

    :

Die Benutzung sieht dann wie folgt aus:

    /etc/init.d/gitea start
    /etc/init.d/gitea status
    /etc/init.d/gitea restart
    /etc/init.d/gitea stop


### systemd
In */lib/systemd/system/gitea.service*

    [Unit]
    Description=Gitea (Git with a cup of tea)
    After=syslog.target
    After=network.target
    #After=mysqld.service
    #After=postgresql.service
    #After=memcached.service
    #After=redis.service

    [Service]
    # Modify these two values and uncomment them if you have
    # repos with lots of files and get an HTTP error 500 because
    # of that
    ###
    #LimitMEMLOCK=infinity
    #LimitNOFILE=65535
    Type=simple
    User=git
    Group=git
    WorkingDirectory=/home/git/gitea
    ExecStart=/home/git/gitea/gitea web
    Restart=always
    Environment=USER=git HOME=/home/git

    [Install]
    WantedBy=multi-user.target

Die Benutzung sieht dann wie folgt aus:

    systemctl start gitea.service
    systemctl status gitea.service
    systemctl restart gitea.service
    systemctl stop gitea.service
    systemctl enable gitea.service   #  <- AUTOSTART

### Reverse Proxy
Für den Reverse Proxy wird NGINX verwendet. Eine Mögliche Konfiguration sieht wie folgt aus.

    server {
        listen       443 ssl;
        server_name  git.example.org;

        ssl_certificate /etc/nginx/ssl/cert.crt;
        ssl_certificate_key /etc/nginx/ssl/privkey.key;

        root   /var/www/html;
        index  index.html index.php index.htm;

        ## Definition Reverse Proxy ##
        location / {
            proxy_pass  http://localhost:3000/;
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            proxy_redirect off;
            proxy_buffering off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }

Hiernach muss in der app.ini noch die Domain auf die externe gesetzt werden, da somit nun alles durch den Proxy geht.