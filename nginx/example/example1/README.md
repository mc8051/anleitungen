HTTP/2 und add_header benögt NGINX 1.10+ 
to generate your dhparam.pem file, run in the terminal
 
    openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
    
Sources:
- https://gist.github.com/plentz/6737338#file-nginx-conf-L30
- https://mozilla.github.io/server-side-tls/ssl-config-generator/
- https://www.htbridge.com/ssl/