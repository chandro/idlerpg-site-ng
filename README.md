# IdleRpg Site NG

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/c407b983eb894a01a8b4d1e80e50dc4a)](https://www.codacy.com/app/falsovsky/idlerpg-site-ng?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=falsovsky/idlerpg-site-ng&amp;utm_campaign=Badge_Grade)
[![CodeFactor](https://www.codefactor.io/repository/github/falsovsky/idlerpg-site-ng/badge)](https://www.codefactor.io/repository/github/falsovsky/idlerpg-site-ng)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/falsovsky/idlerpg-site-ng/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/falsovsky/idlerpg-site-ng/?branch=master)
[![Build Status](https://travis-ci.org/falsovsky/idlerpg-site-ng.svg?branch=master)](https://travis-ci.org/falsovsky/idlerpg-site-ng)
[![Coverage Status](https://coveralls.io/repos/github/falsovsky/idlerpg-site-ng/badge.svg?branch=master)](https://coveralls.io/github/falsovsky/idlerpg-site-ng?branch=master)

## Introduction

This is a rewrite of the original site from http://idlerpg.net/ using Zend Framework 3.
See a live demo at http://idle.deadbsd.org/.

## Installation

The easiest way to install the site is to use
[Composer](https://getcomposer.org/).  If you don't have it already installed,
then please install as per the [documentation](https://getcomposer.org/doc/00-intro.md).



INSTALAR COMPOSER en UBUNTU

```bash
#!/bin/bash
sudo apt update && sudo apt upgrade -y
sudo apt install php php-cli php-xml php-mbstring unzip curl git -y
sudo apt install php7.4 php7.4-cli php7.4-fpm php7.4-mysql php7.4-xml php7.4-mbstring php7.4-curl -y
sudo update-alternatives --set php /usr/bin/php7.4
curl -sS https://getcomposer.org/installer | php
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --version=1.10.26
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
composer update -y
```

To create your new idlerpg-site-ng:

```bash
git clone https://github.com/falsovsky/idlerpg-site-ng.git idlerpg
```

Edit the default configuration file to set the path to your bot files, bot name, etc:

```bash
cd idlerpg
cp config/autoload/local.php.dist config/autoload/local.php
nano config/autoload/local.php
```

Now lets install the PHP dependencies via composer (Choose **"Do not inject"** if asked):

```bash
$ composer install

$ apt remove apache2
```

Once everything is installed, you can test it out immediately using PHP's built-in web server:

```bash
$ php -S 0.0.0.0:8080 -t public/ public/index.php
# OR use the composer alias:
$ composer run --timeout 0 serve

# or use nohup
$ nohup php -S 0.0.0.0:80 -t public/ public/index.php > server.log 2>&1 &

```

This will start the cli-server on port 8080, and bind it to all network
interfaces. You can then visit the site at http://localhost:8080/

**Note:** The built-in CLI server is *for development only*.

## Development mode

The site ships with [laminas-development-mode](https://github.com/laminas/laminas-development-mode)
by default, and provides three aliases for consuming the script it ships with:

```bash
$ composer development-enable  # enable development mode
$ composer development-disable # disable development mode
$ composer development-status  # whether or not development mode is enabled
```

You may provide development-only modules and bootstrap-level configuration in
`config/development.config.php.dist`, and development-only application
configuration in `config/autoload/development.local.php.dist`. Enabling
development mode will copy these files to versions removing the `.dist` suffix,
while disabling development mode will remove those copies.

Development mode is automatically enabled as part of the site installation process. 
After making changes to one of the above-mentioned `.dist` configuration files you will
either need to disable then enable development mode for the changes to take effect,
or manually make matching updates to the `.dist`-less copies of those files.

## Running Unit Tests

To run the supplied site unit tests, you need to do one of the following:

```bash
$ ./vendor/bin/phpunit
```

If you need to make local modifications for the PHPUnit test setup, copy
`phpunit.xml.dist` to `phpunit.xml` and edit the new file; the latter has
precedence over the former when running tests, and is ignored by version
control. (If you want to make the modifications permanent, edit the
`phpunit.xml.dist` file.)

## Using docker-compose

This site provides a `docker-compose.yml` for use with
[docker-compose](https://docs.docker.com/compose/); it
uses the `Dockerfile` provided as its base. Build and start the image using:

```bash
$ docker-compose up -d --build
```

At this point, you can visit http://localhost:8080 to see the site running.

You can also run composer from the image. The container environment is named
"zf", so you will pass that value to `docker-compose run`:

```bash
$ docker-compose run zf composer install
```

## Web server setup

### Apache setup

To setup apache, setup a virtual host to point to the public/ directory of the
project and you should be ready to go! It should look something like below:

```apache
<VirtualHost *:80>
    ServerName idlerpg.localhost
    DocumentRoot /path/to/idlerpg/public
    <Directory /path/to/idlerpg/public>
        DirectoryIndex index.php
        AllowOverride All
        Order allow,deny
        Allow from all
        <IfModule mod_authz_core.c>
        Require all granted
        </IfModule>
    </Directory>
</VirtualHost>
```

### Nginx setup

To setup nginx, open your `/path/to/nginx/nginx.conf` and add an
[include directive](http://nginx.org/en/docs/ngx_core_module.html#include) below
into `http` block if it does not already exist:

```nginx
http {
    # ...
    include sites-enabled/*.conf;
}
```


Create a virtual host configuration file for your project under `/path/to/nginx/sites-enabled/idlerpg.localhost.conf`
it should look something like below:

```nginx
server {
    listen       80;
    server_name  idlerpg.localhost;
    root         /path/to/idlerpg/public;

    location / {
        index index.php;
        try_files $uri $uri/ @php;
    }

    location @php {
        # Pass the PHP requests to FastCGI server (php-fpm) on 127.0.0.1:9000
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_param  SCRIPT_FILENAME /path/to/idlerpg/public/index.php;
        include fastcgi_params;
    }
}
```

Restart the nginx, now you should be ready to go!

## QA Tools

```bash
# Run CS checks:
$ composer cs-check
# Fix CS errors:
$ composer cs-fix
# Run PHPUnit tests:
$ composer test
```
