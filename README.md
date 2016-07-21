# Docker-lamp
Docker compose for a lamp stack (with MongoDB or MySQL).

- [Docker-lamp](#docker-lamp)
- [Docker installation](#docker-installation)
- [add the right repo according to the doc. Such as:](#add-the-right-repo-according-to-the-doc-such-as)
- [deb https://apt.dockerproject.org/repo ubuntu-trusty main](#deb-httpsaptdockerprojectorgrepo-ubuntu-trusty-main)
- [Docker compose installation](#docker-compose-installation)
- [Clone the repository](#clone-the-repository)
- [Configuration](#configuration)
	- [Config Parameters](#config-parameters)
- [Files location](#files-location)
	- [Add binaries](#add-binaries)
- [HostNames](#hostnames)
- [Writing a plugin](#writing-a-plugin)
- [Before running any command](#before-running-any-command)
- [Usage](#usage)
	- [Start the servers](#start-the-servers)
	- [Stop the servers](#stop-the-servers)
	- [Restart the servers](#restart-the-servers)
	- [Status](#status)
	- [Enter a VM](#enter-a-vm)
	- [PHP usage](#php-usage)
	- [MySQL usage](#mysql-usage)


# Docker installation
Read https://docs.docker.com/engine/installation/ubuntulinux/ but in summary (be careful to define the right user instead of {LDAP USER}):
```
sudo su -
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
nano /etc/apt/sources.list.d/docker.list
# add the right repo according to the doc. Such as:
# deb https://apt.dockerproject.org/repo ubuntu-trusty main
apt-get update
apt-get purge lxc-docker
apt-get install linux-image-extra-$(uname -r)
apt-get install docker-engine
service docker start
usermod -aG docker {LDAP USER}
```

**Reboot your computer**


# Docker compose installation
Read https://docs.docker.com/compose/install/ and **take the link to the right version**. Below an example with the 1.6.2:
```bash
sudo su -
curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
exit
```

Verify with `docker-compose --version`

# Clone the repository
You can clone the repository as many times as you want as you can have multiple instances at the same time. A good practice is too have one clone for one project or one clone for projects with the same versions of PHP / MySQL / Elasticsearch, etc ...

Once cloned, you need to run the `install.sh` script:
```bash
git clone ssh://git@git.inetcrm.com:22824/sysadmin/docker-lamp.git
cd docker-lamp
./install.sh
```


# Configuration
Copy the file `conf/compose.ini.tpl` to `conf/compose.ini` and set the right Configuration parameters.

As that tool has been made to develop for SugarCRM, there are two sample configuration files that could be useful `conf/compose.ini.sugar6` and `conf/compose.ini.sugar7`.


## Config Parameters
Everything should be defined in the `[main]` section. **Don't use double quotes to protect your values.**. All values are defined in the compose.ini.tpl.

### Services
You can define the list of services you want to have. Each machin will have the same hostname than their service name. To reach, for example, the elasticsearch server from a web application use `elasticsearch` or to connect to mysql use as the server name `mysql`.
```ini
; Comma separated, valid values: mongo / mongoclient / mysql / mailcatcher / maildev / elasticsearch
services=mongo,maildev,elasticsearch,mysql
```

A service can launch a post-start script that has the same name with an `.sh` extension (example: `service/mysql.sh`).

### Other useful parameters
Machines prefix. It should be different for each project (else the folder name is used).
```ini
; Change Machines names only if you need it
project_name=lamp
```

PHP Version :
```ini
; Set your sugar version to 5.6 or 7.0 (5.6 by default)
php.version=7.0
```

MySQL Password if mysql is defined in the services list:
```ini
; Password set on first start. Once the data exist won't be changed
mysql.root_password=changeme
```

Memory assigned to the VMS:
```ini
apache.ram=512M
elasticsearch.ram=512M
mysql.ram=512M
php.ram=512M
```

# Files location
* All files served by the web server are located into `www/`
* MySQL data is into `mysql/` (created on the first run). If you need to override the mysql configuration you can put a file in `conf/mysql-override` with a `.cnf` extension.
* Mongo data is into `mongo/` (created on the first run)
* Logs for Apache and PHP are located into `logs/`
* If you need to override the PHP configuration you can put a file in `conf/php-fpm-override` with a `.conf` extension. The format is the fpm configuration files one. Example: `php_value[memory_limit] = 127M`.

## Add binaries
You can add binaries (such as sugarcli.phar) that will automatically be available from the PATH by putting it to
`home/www-data/bin/`


# HostNames
VM's urls are given once the servers are started.
* ElasticSearch Server: `elasticsearch`
* MySQL Server: `mysql`
* Mongo Server : `mongo`
* SMTP Server : `maildev` (or `mailcatcher`) with port `25`


# Writing a plugin
To write a plugin you need to create a folder in the plugins/ directory that contains your commands. Each file with a
`.py` extension will be taken as a plugin. The main function should be named exactly like the file.

Example for a file that is in `plugins/my_command/hi.py`:
```python
import click


@click.command(help="Example")
def hi():
    print('Hi!')
```

Once your plugin has been written you need to re-run:
```bash
lamp refresh-plugins
```


# Before running any command
You have to be in a virtual environement:
```bash
source ${PWD##*/}_lamp/bin/activate
```

To leave that environment:
```bash
deactivate
```


# Usage
__WARNING: Make sure that you are in a virtual environment. To verify that, check that your prompt starts with something like `(xyz_lamp) `__

## Start the servers
Run `lamp start` to start the docker environment.

After the run you'll get something like that (contain all the useful URLs):
```bash
lamp is running

To access the web server use : http://172.18.0.7

For maildev use : http://172.18.0.6
            and in your VM use the server "maildev" with the port 25

For phpMyAdmin use : http://172.18.0.6
```


## Stop the servers
Run `lamp stop` to stop all applications.


## Restart the servers
Run `lamp restart` to restart all applications.


## Status
Run `lamp status` to see the list of running VMS


## Enter a VM
If you need to run some commands into a VM you can use the console mode by running `lamp console` with the vm name at the end (only PHP and MySQL are supported).

Example to enter the PHP Machine:
```bash
lamp console php
```

You can also define the user to run the commands with by setting `--user` (choices are _www-data_ or _root_)

**Be careful that all changes are overwritten when you change a parameter in the config and run a start that will update the images (in case the official images have been updated). If you need a definitive change, ask an sysadmin.**


## PHP usage
Use `lamp run php -f www/filename.php` to launch PHP scripts like if you were locally. The path is relative so launch everything from your docker project root (the same folder than this file). If you want to run it from a sub-directory, just use the full path of lamp (example: `/home/user/docker/lamp`) and the relative path of your file.

That:
```bash
cd /home/user/docker
lamp run php -f www/filename.php
```
Is equal to:
```bash
cd /home/user/docker/www
/home/user/docker/lamp run php -f filename.php
```

You can also use that command to run any PHP command (example: `lamp run php -v`). You can also define the user to run the commands with by setting `--user` (choices are _www-data_ or _root_)


## MySQL usage
Use `lamp run mysql` to enter the mysql console.


If you want to create a Database (You can also use the phpMyAdmin service of course):
```bash
lamp run mysql -e "CREATE DATABASE my_db;"
```

If you need to import a file, read it and pipe the command like below:
```bash
zcat file.sql.gz | lamp run mysql db
```
