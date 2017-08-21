Seafile Community or Professional Edition Docker container based on Ubuntu

### Features

* Tailored to use the newest Seafile version at rebuild (so it should always be up-to-date)
* Running under dumb-init to prevent the "child reaping problem"
* Configurable to run with MySQL/MariaDB or SQLite
* Auto-setup at initial run
* Allows usage of Community or Professional Edition

### Quickstart Community Edition

If you want to run with SQLite:
```bash
docker run -d -e SEAFILE_NAME=Seaflail \
    -e SEAFILE_ADDRESS=seafile.example.org \
    -e SEAFILE_ADMIN=seafile@example.org \
    -e SEAFILE_ADMIN_PW=LoremIpsum \
    -v /home/data/seafile:/seafile \
  cmer/seafile
```
If you want to use MySQL:
```bash
docker run -d -e SEAFILE_NAME=Seaflail \
    -e SEAFILE_ADDRESS=seafile.example.org \
    -e SEAFILE_ADMIN=seafile@example.org \
    -e SEAFILE_ADMIN_PW=LoremIpsum \
  -e MYSQL_SERVER=172.17.0.2 \
  -e MYSQL_USER=seafile \
  -e MYSQL_USER_PASSWORD=Seafail \
  -e MYSQL_ROOT_PASSWORD=hunter2 \
    -v /home/data/seafile:/seafile \
  cmer/seafile
```

### Quickstart Professional Edition

Start by setting up a temporary Community Edition Seafile:
```bash
docker run -d -e SEAFILE_NAME=Seafile_setup \
    -e SEAFILE_ADDRESS=seafile.example.org \
    -e SEAFILE_ADMIN=seafile@example.org \
    -e SEAFILE_ADMIN_PW=LoremIpsum \
    -e FORCE_PERMISSIONS=true
    -v /path/to/seafile:/seafile \
    -v /path/to/logs:/opt/haiwen/logs
  danielmohl/docker-seafile
```
This will generate the files and folders needed and speeds up the setup process. After that, stop the container (you can delete it when everything is up & running) and start the pro container:
```bash
docker run -d -e SEAFILE_NAME=Seafile \
    -e SEAFILE_ADDRESS=seafile.example.org \
    -e SEAFILE_ADMIN=seafile@example.org \
    -e SEAFILE_ADMIN_PW=LoremIpsum \
    -e PRO=true
    -v /path/to/seafile:/seafile \
    -v /path/to/logs:/opt/haiwen/logs
  danielmohl/docker-seafile
```
The first start will fail, but correctly convert your community edition installation to pro edition.

Don't forget to double check the settings in System-Administration after login!

### Overview

Filetree:
```
/seafile/
|-- ccnet
|-- conf
|-- seafile-data
`-- seahub-data
/opt/
`-- haiwen
    |-- ccnet -> /seafile/ccnet
    |-- conf -> /seafile/conf
    |-- logs
    |-- pids
    |-- seafile-data -> /seafile/seafile-data
    |-- seafile-server-5.1.3
    |-- seafile-server-latest -> seafile-server-5.1.3
    `-- seahub-data -> /seafile/seahub-data
```
All important data is stored under /seafile, so you should be mounting a volume there (recommended) or at the respective subdirectories. This will not happen automatically!
There are a plethora of environment variables which might be needed for your setup. I recommend using Dockers `--env-file` option.

**Mandatory ENV variables for auto setup**

* **SEAFILE_NAME**: Name of your Seafile installation
* **SEAFILE_ADDRESS**: URL to your Seafile installation
* **SEAFILE_ADMIN**: E-mail address of the Seafile admin
* **SEAFILE_ADMIN_PW**: Password of the Seafile admin

If you want to use MySQL/MariaDB, the following variables are needed:

**Mandatory ENV variables for MySQL/MariaDB**

* **MYSQL_SERVER**: Address of your MySQL server
* **MYSQL_USER**: MySQL user Seafile should use
* **MYSQL_USER_PASSWORD**: Password for said MySQL User
*Optionali:*
* **MYSQL_PORT**: Port MySQL runs on

**Optional ENV variables for auto setup with MySQL/MariaDB**
* **MYSQL_USER_HOST**: Host the MySQL User is allowed from (default: '%')
* **MYSQL_ROOT_PASSWORD**: If you haven't set up the MySQL tables by yourself, Seafile will do it for you when being provided with the MySQL root password

**Optional ENV variable to use the Professional edition of Seafile**
* **PRO**: Download and run Pro (default: false)

**Other optional ENV variables**
* **FORCE_PERMISSIONS**: Force chown'ing Seafile mounted volume at startup. This is slow. Only use if needed. (default: false)

If you plan on omitting /seafile as a volume and mount the subdirectories instead, you'll need to additionally specify `SEAHUB_DB_DIR` which containes the subdirectory of /seafile the *seahub.db* file shall be put in.

There are some more variables which could be changed but have not been tested and are probably not fully functional as well. Therefore those not mentioned here. Inspect the `seafile-entrypoint.sh` script if you have additional needs for customization.

### Migration to Professional Edition
If you have a Professional Edition license file, you must place it in the directory that is mounted as `/seafile` inside the container. For example: `/home/data/seafile/seafile-license.txt` on the Docker host.

For further instructions, see [Migrate from Seafile Community Server](https://manual.seafile.com/deploy_pro/migrate_from_seafile_community_server.html).

### Web server
This container does not include a web server. It's intended to be run behind a reverse proxy. You can read more about that in the Seafile manual: http://manual.seafile.com/deploy/

If you want to run seahub in fastcgi mode, you can pass ENV variables **SEAFILE_FASTCGI=1** and **SEAFILE_FASTCGI_HOST=0.0.0.0**

if you want to use this container with jwilder's nginx-gen and jrcs' letsencrypt-nginx-docker-companion, you have to mount a volume nginx's vhost.d folder and create a file called `seafile.example.com_location` (correct filename according to your settings) with the following content:

```nginx
location /seafhttp {
        rewrite ^/seafhttp(.*)$ $1 break;
        proxy_pass <REPLACE_WITH_CONTAINER_IP - eg 172.17.0.2>:8082;
        client_max_body_size 0;
        proxy_connect_timeout  36000s;
        proxy_read_timeout  36000s;
        proxy_send_timeout  36000s;
        send_timeout  36000s;
    }
```
You have to keep this file up to date manually, so if the container IP address changes, seafile won't be working until you correct this entry.

### Credits

Special thanks to [Till Wiese](https://github.com/m3adow). This container is a fork of his work. He deserves all the credit.

