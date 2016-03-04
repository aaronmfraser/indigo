---
layout: post
title: "Nagios behind Nginx"
description: "Why do I really need to use apache :|"
modified: 2013-05-31
category: posts
tags: 
- nagios
- nginx
blog: true
---


Recently changed jobs, Disney Interactive, and their default web server is Nginx. So to better support nginx I decided to migrate all of the services that I play with to work behind it and today was Nagios's turn. I followed a few other blogs help in getting the configs straighted out but below is another rendition of this exercise.


## Specs:
System: Amazon AMI   
Server: Nginx 1.0.15   
Nagios: 3.3.1   


### Install nagios & plugins:
First lets install the packages needed:

{% highlight bash %}
yum install nagios nagios-plugins-all
{% endhighlight %}
Before we start services we need to head over to change the ownership of a few files. These files are important for nginx to access on startup:
{% highlight bash %}
chown root:nginx /etc/nagios/passwd
chown root:nginx /usr/share/nagios/html/config.inc.php
{% endhighlight %}
Also we need to add nginx to nagios group. This is necessary for nginx to be able to access certain files for FastCGI:
{% highlight bash %}
nagios:x:497:nginx
{% endhighlight %}
We can now go ahead and start services as we wont be changing any of the nagios configs from here on out:
{% highlight bash %}
# service nagios start
Starting nagios: done.

# chkconfig --level 3 nagios on
{% endhighlight %}
### Configure FastCGI and PHP scripts/services
To start you have it install epel. Many of the required packages arent provided with Amazons version of epel
{% highlight bash %}
wget http://mirror.steadfast.net/epel/6/i386/epel-release-6-7.noarch.rpm
rpm -Uvh epel-release-6-7.noarch.rpm

yum install fcgi spawn-fcgi fcgi-devel
{% endhighlight %}
##### Install fcgiwrap
We will need to install fcgiwrap. It is a Simple server for running CGI applications over FastCGI (http://nginx.localdomain.pl/wiki/FcgiWrap). Below are the steps to install fcgiwrap:
{% highlight bash %}
cd /tmp
git clone git://github.com/gnosek/fcgiwrap.git
cd fcgiwrap/
autoreconf -i
make
make install
{% endhighlight %}
##### Configure spawn-fcgi
Once fcgiwrap is installed, what we want to do is enable FastCGI and enable us to pass requests.
To begin lets set up spawn-fcgi config located, /etc/sysconfig/spawn-fcgi:
{% highlight nginx %}
OPTIONS="-u nginx -g nginx -a 127.0.0.1 -p 9001 -f /usr/local/sbin/fcgiwrap -P /var/run/spawn-fcgi.pid"
{% endhighlight %}
##### Configure spawn-fcgi-php
Next lets set up spawn-fcgi-php config located, /etc/sysconfig/spawn-fcgi-php:
{% highlight bash %}
cp /etc/sysconfig/spawn-fcgi /etc/sysconfig/spawn-fcgi-php
{% endhighlight %}
Add the following line to the newly created file:
{% highlight nginx %}
OPTIONS="-u nginx -g nginx -a 127.0.0.1 -p 9002 -f /usr/bin/php-cgi -P /var/run/spawn-fcgi-php.pid"
{% endhighlight %}
### Install & Configure nginx
First we will need to install nginx.
{% highlight bash %}
yum install nginx
{% endhighlight %}
Next lets configure the VirtualHost
{% highlight nginx %}
server {
    server_name  <servername>;

    location / {
        auth_basic "Access to the web interface is restricted";
        auth_basic_user_file /etc/nagios/passwd;
	index index.php;
        rewrite ^/nagios/(.*) /$1 break;

        root /usr/share/nagios/html;
        fastcgi_index  index.php;
        include /etc/nginx/fastcgi_params;
	fastcgi_param SCRIPT_FILENAME /usr/share/nagios/html$fastcgi_script_name;
        if ($uri ~ "\.php"){
	   fastcgi_pass   127.0.0.1:9002;
        }
    }

    location ~ ^/nagios/cgi-bin/ {
	root /usr/lib64/;        
	include /etc/nginx/fastcgi_params;
        auth_basic "Restricted";
        auth_basic_user_file /etc/nagios/passwd;
        fastcgi_param  AUTH_USER $remote_user;
        fastcgi_param  REMOTE_USER $remote_user;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        if ($uri ~ "\.cgi$"){
            fastcgi_pass   127.0.0.1:9001;
        }
    }
}
{% endhighlight %}

