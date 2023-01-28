## Components
Install a lightweight WordPress container with OpenLiteSpeed Edge or Stable version based on Ubuntu 22.04 Linux.
The docker image installs the following packages on your system:

|Component|Version|
| :-------------: | :-------------: || :-------------: | :-------------: || :-------------: | :-------------: |
|                                                                                                             |
|Linux|Ubuntu 22.04|                                                                                          |
|OpenLiteSpeed|[Latest version](https://openlitespeed.org/downloads/)|                                        |
|MariaDB|[Stable version: 10.5](https://hub.docker.com/_/mariadb)|                                            |
|PHP|[Latest version](http://rpms.litespeedtech.com/debian/)|                                                 |
|LiteSpeed Cache|[Latest from WordPress.org](https://wordpress.org/plugins/litespeed-cache/)|                 |
|ACME|[Latest from ACME official](https://github.com/acmesh-official/get.acme.sh)|                            |
|WordPress|[Latest from WordPress](https://wordpress.org/download/)|                                          |
|phpMyAdmin|[Latest from dockerhub](https://hub.docker.com/r/bitnami/phpmyadmin/)|                            |
|Redis|[Latest from dockerhub](https://hub.docker.com/_/redis)|                                               |
|Duplicati|[Latest from dockerhub](https://hub.docker.com/r/linuxserver/duplicati/)|                          |
|                                                                                                             |
| :-------------: | :-------------: || :-------------: | :-------------: || :-------------: | :-------------: |

## Data Structure
Cloned project 
bash
├── acme
├── bin
│   └── container
├── data
│   └── db
├── logs
│   ├── access.log
│   ├── error.log
│   ├── lsrestart.log
│   └── stderr.log
├── lsws
│   ├── admin-conf
│   └── conf
├── sites
│   └── localhost
├── custom
│   └── dockerfile
├── duplicati
│   ├── backups
│   └── config
├── LICENSE
├── README.md
├── redis.conf
└── docker-compose.yml
-----------------------

  * `acme` contains all applied certificates from Lets Encrypt

  * `bin` contains multiple CLI scripts to allow you add or delete virtual hosts, install applications, upgrade, etc 

  * `data` stores the MySQL database

  * `logs` contains all of the web server logs and virtual host access logs

  * `lsws` contains all web server configuration files

  * `sites` contains the document roots (the WordPress application will install here)


### Prerequisites
1. [Install Docker](https://www.docker.com/)
2. [Install Docker Compose](https://docs.docker.com/compose/)

# Install docker and docker-compose
Open terminal and write a command:
------------------------------------------------------------------------------------------------------------
sudo apt-get update && sudo apt-get upgrade
------------------------------------------------------------------------------------------------------------
sudo apt-get install curl
------------------------------------------------------------------------------------------------------------
sudo apt-get install gnupg
------------------------------------------------------------------------------------------------------------
sudo apt-get install ca-certificates
------------------------------------------------------------------------------------------------------------
sudo apt-get install lsb-release
------------------------------------------------------------------------------------------------------------
sudo mkdir -p /etc/apt/keyrings
------------------------------------------------------------------------------------------------------------
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
------------------------------------------------------------------------------------------------------------
sudo apt-get update
------------------------------------------------------------------------------------------------------------
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
------------------------------------------------------------------------------------------------------------

# Download image 
Clone this repository or copy the files from this repository into a new folder:
-------------------------------------------------------------------------------
git clone https://github.com/niezle-ziolko/niezle-ziolko.git
-------------------------------------------------------------------------------

## Configuration
PHP:
Edit the `.env` file to update the demo site domain, default MySQL user, and password.
Feel free to check [Docker hub Tag page](https://hub.docker.com/repository/docker/litespeedtech/openlitespeed/tags) if you want to update default openlitespeed and php versions.

Redis:
Create a folder named "redis" by going to the path /usr/local/etc/.
Then move the file named "redis.conf" to the /usr/local/etc/redis/ path.

After completing these steps, start using swap space:
-----------------------------------------------
sudo fallocate -l 2G  swap
-----------------------------------------------
sudo chmod 600 swap
-----------------------------------------------
sudo mkswap swap
-----------------------------------------------
sudo swapon swap
-----------------------------------------------

## Installation
Open a terminal, `cd` to the folder in which `docker compose.yml` is saved, and run:
------------------------------------------------------------------------------------
docker compose up
Note: If you wish to run a single web server container, please see the [usage method here](https://github.com/litespeedtech/ols-dockerfiles#usage).
------------------------------------------------------------------------------------

## Usage
### Starting a Container
Start the container with the `up` or `start` methods:
------------------------------------------------------------------------------------
docker compose up
------------------------------------------------------------------------------------
You can run with daemon mode, like so:
------------------------------------------------------------------------------------
docker compose up -d
------------------------------------------------------------------------------------
The container is now built and running. 

### Stopping a Container
Stop the constainer with command:
------------------------------------------------------------------------------------
docker compose stop
------------------------------------------------------------------------------------

### Removing Containers
To stop and remove all containers, use the `down` command:
------------------------------------------------------------------------------------
docker compose down
------------------------------------------------------------------------------------

### Rename Containers
When you rename the container, use the `rename` command:
------------------------------------------------------------------------------------
docker rename old_name new_name
------------------------------------------------------------------------------------

### Setting the WebAdmin Password
We strongly recommend you set your personal password right away.
------------------------------------------------------------------------------------
bash bin/webadmin.sh my_password
------------------------------------------------------------------------------------

### Starting a Demo Site
After running the following command, you should be able to access the WordPress installation with the configured domain. By default the domain is http://localhost.
------------------------------------------------------------------------------------
bash bin/demosite.sh
------------------------------------------------------------------------------------

### Creating a Domain and Virtual Host
When you create a domain and virtual host, use this command:
------------------------------------------------------------------------------------
bash bin/domain.sh -add example.com
------------------------------------------------------------------------------------
> Please ignore SSL certificate warnings from the server. They happen if you haven't applied the certificate.

### Deleting a Domain and Virtual Host 
When you delete a domain and virtual host, use this command:
------------------------------------------------------------------------------------
bash bin/domain.sh -del example.com
------------------------------------------------------------------------------------

### Creating a Database
You can either automatically generate the user, password, and database names, or specify them. Use the following to auto generate:
------------------------------------------------------------------------------------
bash bin/database.sh -domain example.com
------------------------------------------------------------------------------------
Use this command to specify your own names, substituting `user_name`, `my_password`, and `database_name` with your preferred values:
------------------------------------------------------------------------------------
bash bin/database.sh -domain example.com -user USER_NAME -password MY_PASS -database DATABASE_NAME
------------------------------------------------------------------------------------

### Installing a WordPress Site
To preconfigure the `wp-config` file, run the `database.sh` script for your domain, before you use the following command to install WordPress:
------------------------------------------------------------------------------------
./bin/appinstall.sh -app wordpress -domain example.com
------------------------------------------------------------------------------------
After install Wordpress and login to `wp-admin`, you must add this command to `wp-config`:
------------------------------------------------------------------------------------
define('WP_REDIS_HOST', 'redis');
define('WP_REDIS_PASSWORD', 'zQktQGuAoUmNPaRXFOeCB1Sp2TB729IW');
define('WP_CACHE_KEY_SALT', 'redis');

### Install ACME 
We need to run the ACME installation command the **first time only**. 
With email notification:
------------------------------------------------------------------------------------
./bin/acme.sh --install --email EMAIL_ADDR
------------------------------------------------------------------------------------

### Applying a Let's Encrypt Certificate
Use the root domain in this command, and it will check for a certificate and automatically apply one with and without `www`:
------------------------------------------------------------------------------------
./bin/acme.sh --domain example.com
------------------------------------------------------------------------------------

Other parameters:

  * [--renew]: Renew a specific domain with -D or --domain parameter if posibile. To force renew, use -f parameter.

  * [--renew-all]: Renew all domains if possible. To force renew, use -f parameter.  

  * [--force]: Force renew for a specific domain or all domains. 

  * [--revoke]: Revoke a domain.  

  * [--remove]: Remove a domain.   

### Update Web Server
To upgrade the web server to latest stable version, run the following:
------------------------------------------------------------------------------------
bash bin/webadmin.sh -upgrade
------------------------------------------------------------------------------------

### Apply OWASP ModSecurity
Enable OWASP `mod_secure` on the web server: 
------------------------------------------------------------------------------------
bash bin/webadmin.sh -mod-secure enable
------------------------------------------------------------------------------------
Disable OWASP `mod_secure` on the web server: 
------------------------------------------------------------------------------------
bash bin/webadmin.sh -mod-secure disable
------------------------------------------------------------------------------------
>Please ignore ModSecurity warnings from the server. They happen if some of the rules are not supported by the server.

### Accessing the Database
After installation, you can use phpMyAdmin to access the database by visiting `http://127.0.0.1:8080` or `https://127.0.0.1:8443`. The default username is `root`, and the password is the same as the one you supplied in the `.env` file.

## Customization
If you want to customize the image by adding some packages, e.g. `lsphp81-pspell`, just extend it with a Dockerfile. 
1. In image is create a `custom` folder and a `custom/Dockerfile`.
2. Add the following example code to `Dockerfile` under the custom folder

| :-------------: | :-------------: || :-------------: | :-------------: || :-------------: | :-------------: |
|                                                                                                             |
|FROM litespeedtech/openlitespeed:latest                                                                      |
|                                                                                                             |
|RUN apt-get update && apt-get install lsphp81-pspell lsphp81-redis -y                                        |
|                                                                                                             |
| :-------------: | :-------------: || :-------------: | :-------------: || :-------------: | :-------------: |

3. Add `build: ./custom` line under the "image: litespeedtech" of docker-composefile. So it will looks like this 

| :-------------: | :-------------: || :-------------: | :-------------: || :-------------: | :-------------: |
| litespeed:                                                                                                  |
|   #image: litespeedtech/openlitespeed:${OLS_VERSION}-${PHP_VERSION}                                         |
|   build: ./custom                                                                                           |
| :-------------: | :-------------: || :-------------: | :-------------: || :-------------: | :-------------: |

4. Build and start it with command:
------------------------------------------------------------------------------------
docker compose up --build
------------------------------------------------------------------------------------
