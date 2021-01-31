FROM        debian:buster
MAINTAINER  Dmitry Shapovalov

ARG         hostuid=963
ARG         hostgid=963

ENV         DEBIAN_FRONTEND=noninteractive \
            RUN_USER=devpi \
            RUN_GROUP=devpi \
            RUN_PORT=8003

COPY        . /tmp

# Install common packages and configure environment
RUN         apt-get update && \
            apt-get install --assume-yes \
                apt-utils \
                build-essential \
                software-properties-common \
                unzip \
                vim \
                wget && \
            apt-get update && \
            apt-get install --assume-yes \
                sqlite \
                tk && \
            apt-get clean

# Create user and group
RUN         groupadd -g $hostgid $RUN_GROUP && \
            useradd -u $hostuid -g $hostgid -M $RUN_USER

# Configure application folders
RUN         mkdir /devpi && \
            mkdir /devpi/data && \
            mkdir /devpi/scripts && \
            mkdir /devpi/logs && \
            mkdir /devpi/backups && \
            chown -R $RUN_USER:$RUN_GROUP /devpi

# Install Python
RUN         apt-get update && \
            apt-get install --assume-yes \
                python3 \
                python3-pip && \
            apt-get clean

# Install Nginx
RUN         apt-get update && \
            apt-get install --assume-yes \
                nginx && \
            apt-get clean && \
            sed -i '\|user.*|a daemon off;' /etc/nginx/nginx.conf && \
            sed -i '\|user .*;|d' /etc/nginx/nginx.conf && \
            sed -i "s|\(\s*\)# server_names_hash_bucket_size 64;|\1server_names_hash_bucket_size 64;|" /etc/nginx/nginx.conf && \
            sed -i "s|\(\s*\)listen 80 default_server;|\1listen 9999 default_server;|" /etc/nginx/sites-available/default && \
            sed -i "s|\(\s*\)listen \[::\]:80 default_server;|\1listen [::]:9999 default_server;|" /etc/nginx/sites-available/default && \
            mkdir /var/lib/nginx/body && \
            mkdir /var/lib/nginx/fastcgi && \
            mkdir /var/lib/nginx/proxy && \
            mkdir /var/lib/nginx/scgi && \
            mkdir /var/lib/nginx/uwsgi && \
            rm -rf /var/log/nginx && \
            ln -s /devpi/logs /var/log/nginx && \
            chown -h $RUN_USER:$RUN_GROUP /var/log/nginx && \
            chmod a+w /run

# Configure app
RUN         pip3 install -U devpi-server && \
            pip3 install -U devpi-web && \
            pip3 install -U devpi-client && \
            mkdir /home/devpi && \
            mkdir /home/devpi/.pip && \
            mv /tmp/pip.conf  /home/devpi/.pip && \
            chown -R $RUN_USER:$RUN_GROUP /home/devpi && \
            cd /tmp && \
            devpi-gen-config && \
            mv /tmp/gen-config/nginx-devpi.conf /etc/nginx/sites-available && \
            rm -rf /tmp/gen-config && \
            ln -s /etc/nginx/sites-available/nginx-devpi.conf /etc/nginx/sites-enabled/nginx-devpi.conf && \
            sed -i "s|\(\s*\)listen 80;|\1listen $RUN_PORT;|" /etc/nginx/sites-available/nginx-devpi.conf && \
            sed -i "s|\(\s*\)root /root/\.devpi/server;|\1root /devpi/data;|" /etc/nginx/sites-available/nginx-devpi.conf && \
            sed -i "s|\(\s*\)try_files /+files\\\$uri @proxy_to_app;|\1try_files /devpi/+files\$uri @proxy_to_app;|" /etc/nginx/sites-available/nginx-devpi.conf && \
            sed -i "s|\(\s*\)location / {|\1location /devpi/ {|" /etc/nginx/sites-available/nginx-devpi.conf && \
            sed -i "s|\(\s*\)\(location @proxy_to_app {\)|\1\2\n\1\1rewrite /devpi/(.*) /\$1 break;|" /etc/nginx/sites-available/nginx-devpi.conf && \
            sed -i "s|\(\s*\)\(location /devpi/\)|\1location /devpi {\n\1\1alias /devpi/;\n\1}\n\1\2|" /etc/nginx/sites-available/nginx-devpi.conf && \
            sed -i "s|\(\s*\)proxy_set_header X-outside-url \\\$scheme://\\\$http_host;|\1proxy_set_header X-outside-url \$scheme://\$http_host/devpi;|" /etc/nginx/sites-available/nginx-devpi.conf && \
            mv /tmp/logging.yaml /devpi

# Configure helper scripts
RUN         mv /tmp/startup /devpi/scripts && \
            mv /tmp/shutdown /devpi/scripts && \
            mv /tmp/backup /devpi/scripts && \
            mv /tmp/restore /devpi/scripts && \
            chmod 755 /devpi/scripts/*

USER        $RUN_USER

EXPOSE      $RUN_PORT

ENTRYPOINT  ["/devpi/scripts/startup"]
