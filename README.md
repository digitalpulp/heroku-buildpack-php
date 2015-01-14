Apache+PHP build pack
========================

This is a build pack bundling PHP and Apache for Heroku apps.

Update / Warning
----------------

This buildpack currently has issues when serving to newly-created apps: deploying works, but the app can't start and the following message is logged (visible using `heroku logs`):

```
httpd: Syntax error on line 58 of /app/apache/conf/httpd.conf: Cannot load /app/apache/modules/libphp5.so into server: libssl.so.0.9.8: cannot open shared object file: No such file or directory
```

This is because [libssl0.9.8 is no longer present in `cedar-14`](https://devcenter.heroku.com/articles/cedar-ubuntu-packages), which is the new default [Heroku stack](https://devcenter.heroku.com/articles/stack). I'm unsure as to when this change was implemented, but I just bumped my head in this a few days ago.

There are two workarounds:

* Remain using the `cedar` stack

* Use the official Heroku-supported PHP buildpack, [heroku-buildpack-php](https://github.com/heroku/heroku-buildpack-php), but it lacks [the customizations we made](https://github.com/digitalpulp/heroku-buildpack-php/compare/heroku:master...master) (`ACCESSFILENAME` and the NewRelic stuff). Note that we forked from heroku-buildpack-php, but that was a long a while ago (back in 2013; the last commit from ddollar was Dec 2012) and the buildpack has undergone substantial changes since then.

I have not yet decided a clear path forward on this. Just documenting for the masses. -JSB 2015-01-14



Configuration
-------------

The config files are bundled with the build pack itself:

* conf/httpd.conf
* conf/php.ini

Apache .htaccess files
----------------------
The apache configuration has a setting, AccessFileName, that tells the web server the name of the override file. This is usually set to ".htaccess".
This build pack will use the ACCESSFILENAME environment variable to determine the name of the override files. We do this to follow our normal pattern of different override files for each environment in the development process (e.g. Production, Staging, Development).

The default value is .htaccess

Pre-compiling binaries
----------------------

    # apache
    mkdir /app
    wget http://apache.cyberuse.com//httpd/httpd-2.2.19.tar.gz
    tar xvzf httpd-2.2.19.tar.gz
    cd httpd-2.2.19
    ./configure --prefix=/app/apache --enable-rewrite
    make
    make install
    cd ..
    
    # php
    wget http://us2.php.net/get/php-5.3.6.tar.gz/from/us.php.net/mirror 
    mv mirror php.tar.gz
    tar xzvf php.tar.gz
    cd php-5.3.6/
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl
    make
    make install
    cd ..
    
    # php extensions
    mkdir /app/php/ext
    cp /usr/lib/libmysqlclient.so.15 /app/php/ext/
    
    # pear
    apt-get install php5-dev php-pear
    pear config-set php_dir /app/php
    pecl install apc
    mkdir /app/php/include/php/ext/apc
    cp /usr/lib/php5/20060613/apc.so /app/php/ext/
    cp /usr/include/php5/ext/apc/apc_serializer.h /app/php/include/php/ext/apc/
    
    
    # package
    cd /app
    echo '2.2.19' > apache/VERSION
    tar -zcvf apache.tar.gz apache
    echo '5.3.6' > php/VERSION
    tar -zcvf php.tar.gz php


Hacking
-------

To change this buildpack, fork it on Github. Push up changes to your fork, then create a test app with --buildpack <your-github-url> and push to it.


Meta
----

Created by Pedro Belo.
Many thanks to Keith Rarick for the help with assorted Unix topics :)