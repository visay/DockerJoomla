# DockerJoomla helps you developing Joomla CMS projects

DockerJoomla creates the necessary Docker containers (webserver, database, php, mail, redis, elasticsearch, couchdb)
to run your Joomla CMS project. The package provides a wrapper script in `vendor/bin/dockerjoomla`
which simplifies the handling of docker and does all the configuration necessary.

We created this package to make development on Joomla CMS projects easier and
to create a simple reusable package which can easily be maintained and serves well for the standard project.

Development will continue further as the package is already reused in several projects.
Contributions and feedback are very welcome.

## Install docker

    https://docs.docker.com/installation/ (tested with docker v1.11)

## Install docker-compose

We use docker-compose to do all the automatic configuration:

    http://docs.docker.com/compose/install/ (tested with docker-compose v1.6)

The repository contains a Dockerfile which will automatically be built in the
[docker hub](https://registry.hub.docker.com/u/visay/dockerjoomla/) after each change
and used by docker-compose to build the necessary containers.

## Install DockerJoomla into your distribution

Add `visay/dockerjoomla` as dev dependency in your composer, using the latest stable release is highly recommended.

*Example*:

```
composer require --dev visay/dockerjoomla dev-master
```

*Note*:

DockerJoomla uses port 80 for web access so you need to make sure that your host machine does not have any software
using that port. Usually this happens if you have apache or nginx installed in your host machine, so you can stop it with:

```
sudo service apache2 stop
sudo service nginx stop
```

## Run DockerJoomla

    vendor/bin/dockerjoomla up -d

The command will echo the url with which you can access your project. Add the hostname then to your `/etc/hosts`
and set the ip to your docker host (default for linux is 0.0.0.0). You can also use any subdomain with *.hostname and
it will point to the same server. What you need to do is to add exact subdomain name to your `/etc/hosts`.
The parameter `-d` will keep it running in the background until you run:

    vendor/bin/dockerjoomla stop

## Setup Database Connection

    <?php
    class JConfig {
        public $offline = '0';
        public $offline_message = 'This site is down for maintenance.<br />Please check back again soon.';
        public $display_offline_message = '1';
        public $offline_image = '';
        public $sitename = 'Joomla Demo';
        public $editor = 'tinymce';
        public $captcha = '0';
        public $list_limit = '20';
        public $access = '1';
        public $debug = '0';
        public $debug_lang = '0';
        public $dbtype = 'mysqli';
        public $host = 'db';
        public $user = 'root';
        public $password = 'root';
        public $db = 'dockerjoomla';
        public $dbprefix = 'vi7fd_';
        public $live_site = '';
        public $secret = 'rVwOsoPiMdAdQRqx';
        public $gzip = '0';
        public $error_reporting = 'default';
        public $helpurl = 'https://help.joomla.org/proxy/index.php?option=com_help&amp;keyref=Help{major}{minor}:{keyref}';
        public $ftp_host = '';
        public $ftp_port = '';
        public $ftp_user = '';
        public $ftp_pass = '';
        public $ftp_root = '';
        public $ftp_enable = '0';
        public $offset = 'UTC';
        public $mailonline = '1';
        public $mailer = 'mail';
        public $mailfrom = 'keo@visay.info';
        public $fromname = 'Joomla Demo';
        public $sendmail = '/usr/sbin/sendmail';
        public $smtpauth = '0';
        public $smtpuser = '';
        public $smtppass = '';
        public $smtphost = 'localhost';
        public $smtpsecure = 'none';
        public $smtpport = '25';
        public $caching = '0';
        public $cache_handler = 'file';
        public $cachetime = '15';
        public $cache_platformprefix = '0';
        public $MetaDesc = 'This is Joomla Demo';
        public $MetaKeys = '';
        public $MetaTitle = '1';
        public $MetaAuthor = '1';
        public $MetaVersion = '0';
        public $robots = '';
        public $sef = '1';
        public $sef_rewrite = '0';
        public $sef_suffix = '0';
        public $unicodeslugs = '0';
        public $feed_limit = '10';
        public $feed_email = 'none';
        public $log_path = '/var/www/logs';
        public $tmp_path = '/var/www/tmp';
        public $lifetime = '15';
        public $session_handler = 'database';
    }

## Check the status

    vendor/bin/dockerjoomla ps

This will show the running containers. The `data` container can be inactive to do it's work.

# Tips & Tricks

## Configure remote debugging from your host to container

DockerJoomla installs by the default xdebug with the following config on the server:

    xdebug.remote_enable = On
    xdebug.remote_host = 'dockerhost'
    xdebug.remote_port = '9001'
    xdebug.max_nesting_level = 500

So you can do remote debugging from your host to the container through port 9001. From your IDE, you need to configure
the port accordingly. If you are using PHPStorm, this link may be useful for you to configure your IDE properly.

## Running a shell in one of the service containers

    vendor/bin/dockerjoomla run SERVICE /bin/bash

SERVICE can currently be `app`, `web`, `data`, `db`, `redis`, `elasticsearch` or `couchdb`.

## Access database inside container from docker host

While you can easily login to shell of the `db` container with `vendor/bin/dockerjoomla run db /bin/bash`
and execute your mysql commands, there are some cases that you want to run mysql commands directly
from your host without having to login to the `db` container first. One of the best use cases,
for example, is to access the databases inside the container from MySQL Workbench tool.
To be able to do that, we have mapped database port inside the container (which is `3306`) to your
host machine through `3307` port.

![Screenshot of MySQL Workbench interface](/docs/MySQL-Workbench.png "MySQL Workbench interface")

## Access CouchDB

From your host machine, you can access couchdb from web interface or command line:

__Web__: [http://0.0.0.0:5984/_utils/](http://0.0.0.0:5984/_utils/)

__Cli__: `curl -X GET http://0.0.0.0:5984/_all_dbs`

From inside your `app` container, you can also access couchdb through the command line:

```
vendor/bin/dockerjoomla run app /bin/bash
curl -X GET http://couchdb:5984/_all_dbs
```

## Attach to a running service

Run `vendor/bin/dockerjoomla ps` and copy the container's name that you want to attach to.

Run `docker exec -it <containername> /bin/bash` with the name you just copied.
With this you can work in a running container instead of creating a new one.

## Check open ports in a container

    vendor/bin/dockerjoomla run SERVICE netstat --listen

# Further reading

* [blog post on php-fpm](http://mattiasgeniar.be/2014/04/09/a-better-way-to-run-php-fpm/)
* [nginx+php-fpm+mysql tutorial](http://www.lonelycoder.be/nginx-php-fpm-mysql-phpmyadmin-on-ubuntu-12-04/)
* [Docker documentation](http://docs.docker.com/reference/builder/)
* [docker-compose documentation](http://docs.docker.com/compose)
