---
layout: posts
title: Laravel on an Ubuntu VM
date: 2024-09-13 15:30:45
tags: laravel,devops,nginx,ubuntu,mysql,redis
published: true
---
## Development on a VM?

This sounds crazy, but I will be developing my Laravel apps on a VM. We don't all have the luxury of having an over powered VM server running in our basement, but I do. If you also want deploy on a VPS running Ubuntu, you could follow these steps to have MySQL, NGINX, Redis and a running a Laravel web app. This is just a starting point.

I'm installing Ubuntu 22.04 on my VM. It gets 4 logical cores and 8 GiB of memory. Make sure you have the public SSH key for the machine your working on stored in your GitHub account before the installer finishes, since it allows you to import SSH keys automatically into your VPS as your setting it up, all you need to know is your github username. Once the install is complete, SSH into the VM.

The next hurdle is installing PHP. If you're using Ubuntu 22.04, then the default version of PHP is 8.1, but Laravel 11 requires PHP 8.2. In order to do so, you will have to add the PHP repository to apt using the following command:

{% codeblock lang:bash line_number:false %}
sudo add-apt-repository ppa:ondrej/php
{% endcodeblock %}

The next hurdle is installing PHP; you can't just install PHP8.2 by itself, becuase if you do it will install `apache2` as a dependency by default. It's a fine server, but we will be using NGINX. In order to avoid `apache2`, you need to install a set of packages that include `php-fpm` or `php-cli`. We will be using PHP FPM to handle our application so we're going to install PHP and our other services; Redis, MySQL, and NGINX with the following command:

{% codeblock lang:bash line_number:false %}
sudo apt install php8.2-fpm php8.2 unzip php8.2-curl php8.2-xml php8.2-mbstring php8.2-redis redis-server php8.2-mysql mysql-server nginx
{% endcodeblock %}

If you'd like to see what packages are installed before you install them, tag the flag `--dry-run` on the end of the command. It will show you the packages to be installed and prove that `libapache` is no where to be seen! This command includes a few extra PHP module packages that are required by Laravel 11.

The next step is to install Composer, which can be accomplished many ways, but the way recommended by the Composer team is to use their install script:

{% codeblock lang:bash line_number:false %}
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
{% endcodeblock %}

If you copy and paste that into your terminal, it will download and check the hash of the install file, then you can use the following command to move it into your binaries folder:

{% codeblock lang:bash line_number:false %}
sudo mv composer.phar /usr/local/bin/composer
{% endcodeblock %}

## Configuring NodeJS
To install NodeJS and NPM, use the following command:

{% codeblock lang:bash line_number:false %}
sudo apt install nodejs npm
{% endcodeblock %}

Unfortunately, the version of Node that ships with Ubuntu 22 is well behind the stable version and there are many ways to get stable, but I find the that the easiest ist to globally install the `n` package via npm:

{% codeblock lang:bash line_number:false %}
sudo npm install -g n
{% endcodeblock %}

The `n` package is able to update NodeJs with this comically short command:

{% codeblock lang:bash line_number:false %}
sudo n stable
{% endcodeblock %}

You will probably have to leave your SSH session or restart your terminal session in order for the changes to NodeJS to take effect.

## Configuring NGINX

Although NGINX can be configured to serve a single site, I would recommend using it's built-in multi-site configuration by adding a valid configuration to the `/etc/nginx/sites-available` directory and then add a symbolic link from that file to the `/etc/nginx/sites-enabled` directory. The [Laravel documentation](https://laravel.com/docs/11.x/deployment#nginx) has an NGINX config, which you should use as a starting point. If you make configuration file named `laravel.conf` the command to symlink it would be:

{% codeblock lang:bash line_number:false %}
sudo ln -s /etc/nginx/sites-available/laravel.conf /etc/nginx/sites-enabled
{% endcodeblock %}

The default user associated with the NGINX service is named `www-data`; therefore any files that are being served by NGINX need to be owned by the `www-data` user or managed by the `www-data` group.

## Setting permissions on your application

Whether you're starting a new Laravel project or cloning a repository, you will have to set permissions on your codebase such that the `www-data` user will be able to access the files. By convention, the `www-data` user owns the `/var/www` directory, but if you deploy your application there using your username, the permissions will result in an error if you try to load any of your applications routes. After you have cloned or copied your sourcecode into a subdirectory of `/var/www`, you should install your Composer and NPM packages and build your front end. Afterwards, run the following commands to set permissions and file ownership:

{% codeblock lang:bash line_number:false %}
sudo chown -R $USER:www-data .
{% endcodeblock %}

This makes you the owner of the all the files, but the group is set to `www-data`. Next, you need to make sure that all the files going forward are owned by the `www-data` group.

The following commands add your user to the `www-data` group, ensure that all files created in the folder going forward are owned by the `www-data` group and that others cannot read write or execute them.

{% codeblock lang:bash line_number:false %}
sudo usermod -a -G www-data $USER
sudo chmod g+s .
sudo chmod o-rwx .
{% endcodeblock %}

Finally, per Laravel's requirements, you can set write permissions to the folders that the app uses for cache and storage.

{% codeblock lang:bash line_number:false %}
sudo chmod -R ug+rwx storage bootstrap/cache
{% endcodeblock %}

## Configure MySQL Database

When we installed all our packages, we also installed MySQL server and PDO, which allows PHP to communicate with databases. In order to create a table and user for our app, we will have to use the MySQL CLI. In Ubuntu, root account is locked, but it's also the only user with privileges to make any changes, so you'll have to use `sudo`:

{% codeblock lang:bash line_number:false %}
sudo mysql
{% endcodeblock %}

This will drop you into the MySQL CLI with privileges so you can create a database for your app and a user to access it. In the following commands, replace `app_user` and `db_name` with your app user name and database name, and `a-secure-password` with your DB user's password. These should be the same values as referenced in your `.env` file for your Laravel project. 

{% codeblock lang:sql line_number:false %}
    CREATE USER 'app_user'@'%' IDENTIFIED BY 'a-secure-password';
    CREATE DATABASE db_name;
    GRANT ALL PRIVILEGES ON db_name.* TO 'app_user'@'%';
{% endcodeblock %}

At this point, you can migrate your database and run your seeders! You've done it. If your app continues to give you issues about permissions, you may need to assign ownership of all the files to your `www-data` user in order to serve them.

{% codeblock lang:bash line_number:false %}
sudo chown -R www-data:www-data .
{% endcodeblock %}