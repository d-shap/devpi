# DevPI web server
Docker image for PyPI caching web server.

Container runs as non-root user.
This user owns DevPI process and owns DevPI files.

To run container next volumes should be mapped:
* folder for DevPI files
* logs folder
* backups folder

## Installation
### Installation from docker image
1. Pull docker image.

2. Create user and group to own DevPI files and to run docker container:
    ```
    sudo groupadd -g 963 devpi
    ```
    ```
    useradd -u 963 -g 963 -M devpi
    ```

3. Proceed to configuration.

### Installation from source
1. Pull project sources from version control system.

2. Create user and group to own DevPI files and to run docker container:
    ```
    sudo useradd -r devpi
    ```

3. Make **build** executable:
    ```
    sudo chmod u+x ./build
    ```

4. Execute **build**:
    ```
    sudo ./build devpi
    ```

5. Proceed to configuration.

### Configuration
1. Create folders for DevPI files:
    ```
    sudo mkdir /devpi
    ```
    ```
    sudo mkdir /devpi/data
    ```

2. Create folder for logs:
    ```
    sudo mkdir /var/log/devpi
    ```

3. Create folder for backups:
    ```
    sudo mkdir /var/backups/devpi
    ```

4. Grant permit to all folders:
    ```
    sudo chown -R devpi:devpi /devpi
    ```
    ```
    sudo chown devpi:devpi /var/log/devpi
    ```
    ```
    sudo chown devpi:devpi /var/backups/devpi
    ```

5. Copy **etc/init.d/devpi** to **/etc/init.d** folder:
    ```
    sudo cp ./etc/init.d/devpi /etc/init.d
    ```

6. Copy **usr/sbin/devpi** to **/usr/sbin** folder:
    ```
    sudo cp ./usr/sbin/devpi /usr/sbin
    ```

7. Copy **usr/bin/dputil** to **/usr/bin** folder:
    ```
    sudo cp ./usr/bin/dputil /usr/bin
    ```

8. Make all files executable:
    ```
    sudo chmod a+x /etc/init.d/devpi
    ```
    ```
    sudo chmod a+x /usr/sbin/devpi
    ```
    ```
    sudo chmod a+x /usr/bin/dputil
    ```

9. Register service:
    ```
    sudo update-rc.d devpi defaults
    ```

10. Start DevPI service:
    ```
    sudo service devpi start
    ```

## Client configuration
1. Create file **~/.pip/pip.conf** (Linux) or **%APPDATA%\pip\pip.ini** (Windows).

2. Specify pip index URL in this file:
    ```
    [global]
    index-url = http://server/devpi/root/pypi/+simple/
    trusted-host = server
    ```
   where **server** is DevPi server's name or IP.

## Management
### Service management
```
sudo service devpi (start|stop|status|restart)
```

### Run DevPi client
```
sudo dputil devpi <arguments>
```

**devpi** command is passed to the running docker container with **dputil** utility.

### Create backup
```
sudo dputil backup <filename>
```

Backup file **/var/backups/devpi/&lt;filename&gt;.tar.gz** will be created.

### Restore backup
```
sudo dputil restore <filename>
```

### Command line (bash)
```
sudo dputil bash
```

## Apache mod_proxy configuration
DevPI web server can be located with another web applications.
For example, mercurial, bugzilla, wiki etc can be run as docker containers on the same host.
In this case apache server can be used to redirect requests to different docker containers.

1. Enable apache mod_proxy:
    ```
    sudo a2enmod deflate headers proxy proxy_ajp proxy_balancer proxy_connect proxy_html proxy_http rewrite
    ```

2. Configure proxy:
    ```
    <VirtualHost *:80>
        ...
        ProxyPreserveHost On
        <Proxy *>
            Order allow,deny
            Allow from all
        </Proxy>
        ...
    </VirtualHost>
    ```

3. Copy **./etc/apache2/sites-available/devpi.conf** to **/etc/apache2/sites-available** folder:
    ```
    sudo cp ./etc/apache2/sites-available/devpi.conf /etc/apache2/sites-available
    ```

4. Enable apache sites:
    ```
    sudo a2ensite devpi
    ```

5. Restart apache service:
    ```
    sudo service apache2 restart
    ```

## HOW TO
### Connect to the server
1. Specify the server to use:
    ```
    sudo dputil devpi use http://localhost:3141
    ```

2. Connect to the server:
    ```
    sudo dputil devpi login root
    ```
    The default root password is empty (simply press Enter).

### How to create new user
1. Create user:
    ```
    sudo dputil devpi user -c packages email=packaging@company.com password=packages
    ```

2. Verify that user is created:
    ```
    sudo dputil devpi user -l
    ```

### How to create new index
1. Create stable index:
    ```
    sudo dputil devpi index -c packages/stable bases=root/pypi volatile=False
    ```

2. Create staging index:
    ```
    sudo dputil devpi index -c packages/staging bases=packages/stable volatile=True
    ```

### How to create cron job for backups
```
sudo crontab -l | { cat; echo "minute hour * * * /usr/bin/dputil backup <filename>"; echo ""; } | sudo crontab -
```

### How to create new user

# Donation
If you find my code useful, you can [bye me a coffee](https://www.paypal.me/dshapovalov)
