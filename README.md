# install_nexus_with_https
install nexus with https

### 0. OS
```
CentOS release 6.10 (Final)
```

### 1. install nginx
- http://nginx.org
```zsh
# vi /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/6/$basearch
gpgcheck=0
enabled=1

# yum install nginx

# service nginx start
Starting nginx:                                            [  OK  ]

# service nginx status
nginx (pid  7320) is running...

```

### 2. install nexus
- https://www.sonatype.com
```zsh
# tar xzvf nexus-3.xx.x-xx-unix.tar.gz

```

### 3. configure nginx.conf 
```zsh
# cd /etc/nginx/conf.d

# vi nexus.abc.com.conf
server {
    server_name  nexus.abc.com;

    #charset koi8-r;
    access_log  /var/log/nginx/nexus.abc.com.access.log  main;

    location / {
      proxy_pass 		http://127.0.0.1:8081;
      
      proxy_set_header 	X-Forwarded-For	$proxy_add_x_forwarded_for;
      # for https
      proxy_set_header	X-Forwarded-Proto	https;
      proxy_set_header	Host	$host;
      proxy_set_header	X-Real-IP	$remote_addr;
      
      proxy_redirect          off;
      proxy_buffering         off;
      proxy_connect_timeout 	90;
      proxy_send_timeout 	90;
      proxy_read_timeout 	90;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
    
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/nexus.abc.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/nexus.abc.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = nexus.abc.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    server_name  nexus.abc.com;
    listen 80;
    return 404; # managed by Certbot


}    
```

### 4. configure nexus-default.properties
```zsh
# vi nexus-3.xx.x-xx/etc/nexus-default.properties
## DO NOT EDIT - CUSTOMIZATIONS BELONG IN $data-dir/etc/nexus.properties
##
# Jetty section
application-port=8081
application-host=0.0.0.0
nexus-args=${jetty.etc}/jetty.xml,${jetty.etc}/jetty-http.xml,${jetty.etc}/jetty-requestlog.xml
nexus-context-path=/

# Nexus section
nexus-edition=nexus-pro-edition
nexus-features=\
 nexus-pro-feature

nexus.hazelcast.discovery.isEnabled=true

# cd nexus/bin
# ./nexus start
```
