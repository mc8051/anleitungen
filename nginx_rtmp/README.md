## NGINX Streaming Server

Der stabile NGINX Server hat viele Module wie auch unter anderem einen RTMP Server.  
Die Kompelierte Version beinhaltet dieses Modul nicht, dafür muss es selber Kompeliert werden.

### Kompelieren
Build Tools installieren

    apt-get install build-essential libpcre3 libpcre3-dev libssl-dev

Neuste Version herunterladen

    curl -O http://nginx.org/download/nginx-1.9.9.zip

RTMP Modul herunterladen aus dem GIT

    curl -O https://github.com/arut/nginx-rtmp-module/archive/master.zip

Build durchfüherunterladen

    unzip nginx-1.9.9.zip
    unzip master.zip
    cd nginx-1.9.9
    ./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-master
    make
    make install

Sollte alles ohne Fehler durchgelaufen sein, so ist dieser hier installiert */usr/local/nginx/*  
Solange NGINX nicht als Service eingerichtet ist muss dieser wie folgt gestartet und gestopt werden:

    /usr/local/nginx/sbin/nginx
    /usr/local/nginx/sbin/nginx -s restart

### Standard Konfiguration
/usr/local/nginx/conf/nginx.conf

    rtmp {
      server {
        listen 1935;
        chunk_size 4096;

        application live {
          live on;
          push rtmp://<streaming service rtmp url>/<stream key>
        }
      }
    }

### Erweiterte Konfiguration für Dual-Streaming

    rtmp {
      server {
        listen 1935;
        chunk_size 4096;

        application live {
          live on;
          push rtmp://localhost/host1
          push rtmp://localhost/host2
        }

        application host1 {
          live on;
          push rtmp://<streaming service rtmp url>/<stream key>
        }

        application host2 {
          live on;
          push rtmp://<streaming service rtmp url>/<stream key>
        }
      }
    }

### Erweiterte Konfiguration mit Transcoding
ffmpeg installieren, für das Transcoding. Dafür muss in */etc/apt/source.list* folgendes hinzugefügt werden:

    deb ftp://ftp.deb-multimedia.org jessie main non-free

Updaten und installieren von ffmpeg

    apt-get update
    apt-get install deb-multimedia-keyring
    apt-get update
    apt-get install ffmpeg

/usr/local/nginx/conf/nginx.conf

    rtmp {
      server {
        listen 1935;
        chunk_size 8192;

        application live {
          live on;
          record off;

          allow publish <insert IP adress>;
          deny publish all;

          push rtmp://localhost/youtube/;
          exec_push ffmpeg -i rtmp://localhost:1935/live/1080 -r 30 -s 1280x720 -c:v libx264 -g 60 -maxrate 3000k -bufsize 900k -preset ultrafast -c:a copy -threads 6 -f flv rtmp://127.0.0.1/twitch/1080;
        }

        application youtube {
          live on;
          record off;

          allow publish 127.0.0.1;
          deny publish all;
          push rtmp://a.rtmp.youtube.com/live2/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX;
        }

        application twitch {
          live on;
          record off;

          allow publish 127.0.0.1;
          deny publish all;
          push rtmp://live-fra.twitch.tv/app/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX;
        }
      }
    }

### NGINX RTMP Statistik
/usr/local/nginx/conf/nginx.conf

    http {
        server {
            listen 80;

            # This URL provides RTMP statistics in XML
            location /stat {
                rtmp_stat all;

                # Use this stylesheet to view XML as web page
                # in browser
                rtmp_stat_stylesheet stat.xsl;
            }

            location /stat.xsl {
                # XML stylesheet to view RTMP stats.
                # Copy stat.xsl wherever you want
                # and put the full directory path here
                root /path/to/stat.xsl/;
            }
        }
    }

### RTMP Absichern
Hierfür muss NGINX PHP unterstützen. [Siehe hier](https://github.com/mc8051/anleitungen/tree/master/nginx_php5).

Zum authentifizieren wird folgendes PHP Script verwendet.

    <?php
    // www.server.com/auth.php?user=felix&pass=felixpassword

    //check if querystrings exist or not
    if(empty($_GET['user']) || empty($_GET['pass']))
       {
        //no querystrings or wrong syntax
        echo "wrong query input";
        header('HTTP/1.0 404 Not Found');
        exit(1);
       }

    else
       {
        //querystring exist
        $username = $_GET['user'];
        $password = $_GET['pass'];
       }

    $savedpassword =  felixpassword ;
    $saveduser = felix ;


    //check pass and user string
    if (strcmp($password,$savedpassword)==0 &&  strcmp($username,$saveduser)==0 )
          {
    	echo "Password and Username OK! ";

    	}

    else
          {
    	echo "password or username wrong! ";
    	header('HTTP/1.0 404 Not Found'); //kein stream
            }

    ?>

Quelle: https://groups.google.com/d/msg/nginx-rtmp/y028v8RVx9o/dND4THOLUc0J


/usr/local/nginx/conf/nginx.conf

    application live {
      live on;
      on_publish http://localhost/auth.php;
      notify_method get;
      push rtmp://<streaming service rtmp url>/<stream key>
    }

Somit muss jeder, der über die application live senden möchte, einen Streamschlüssel mit angeben. Die Einstellung in OBS sähe wie folgt aus.
    Server: rtmp://<rtmp server>/live
    Playpath/Streamkey: streamkey?user=username&pass=userpassword

### Sonstiges
Variabeln für NGINX RTMP

    rtmp://localhost/${app}/${name}
