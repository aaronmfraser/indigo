---
layout: post
title: ssh and you
description: "how to make ssh easier"
modified: 2014-02-24
category: posts
tags: 
- sysops
- ssh
blog: true
---

Recently been spending alot of time working on side projects for friends and past employers. The number of servers that needs my attention has increased and each of them have separate usernames and ssh keys. To manage this I was using shell aliases 

{% highlight bash %}
alias mysrvr="ssh -i ~/.ssh/mysrvr.priv root@mysrvr.afraser.io"
{% endhighlight %}

## ssh configs
These worked great and all but I still didnt have an easy way to scp files. Trying to make an alias dynamic enough to be able to scp was a pain.
In exploring this issue and I found ssh configs and they proved to be far more powerful. Soon I replaced  my alias habit with an ssh config

File is located in your home dir ~/.ssh/config. 

{% highlight bash %}
Host c1-dev
   HostName client1-server01.test.co
   User root
   IdentityFile ~/.ssh/client1.pem
Host c2-dev01
   HostName client2-server01.anothertest.co
   User root
   IdentityFile ~/.ssh/client2.pem
Host mysrvr
   HostName mysrvr.afraser.io
   User root
   IdentityFile ~/.ssh/mysrvr.priv
{% endhighlight %}

## sshpass
I have found some situations were having an ssh config wasnt going to work out. If you have servers that are dont support ssh keys (Load balancers for example) there is a tool for that, SSHPASS.

<br />
First you'll need to install it:
<br />
{% highlight bash %}sudo apt-get install sshpass{% endhighlight %}

To use ssh pass, I find it best to have a sourced file with an environment variable:
{% highlight bash %}
$ cat bastion-login
  SSH_PASS="some-horribly-insecure-password"

$ source bastion-login
{% endhighlight %}

Once the file is sourced you can then simply prepend sshpass to your standard ssh commands like so:
{% highlight bash %}sshpass -e ssh -o StrictHostKeyChecking=no  cloud-user@mysrvr.afraser.io {% endhighlight %}

In the above example you will see ```-o StrictHostKeyChecking=no```.  This simply auto-accepts/ignores the Host Key check you would normally acknowledge for the first connection to the destination server.

Host key errors typically look like this:
{% highlight bash %}
$ ssh cloud-users@mysrvr.afraser.io
The authenticity of host 'mysrvr.afraser.io (208.9.150.2)' can't be established.
RSA key fingerprint is cf:1b:f4:1f:c5:aa:b1:1e:bf:4e:5e:cf:53:fa:a8:73.
Are you sure you want to continue connecting (yes/no)? 
{% endhighlight %}


Hopefully this can help make sshing a more pleasurable experience.
