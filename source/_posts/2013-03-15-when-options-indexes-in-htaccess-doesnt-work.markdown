---
layout: post
title: "When Options -Indexes in .htaccess doesn't work"
date: 2013-03-15 19:48
comments: true
categories: [apache2, ec2]
---

We've our sandbox server running on Amazon EC2. There are a lot of projects hosted there with each project's .htaccess taking care of it. Apart from the various tasks that the .htaccess does, it restricts directory browsing using simple directive

```
<IfModule mod_autoindex.c>
  Options -Indexes
</IfModule>
```

But somehow on the root folder of the sandbox, this directive didn't work. This implies, a directory specific directive was somewhere being issues. These are a few places to look for

```
/etc/apache2/httpd.conf
/etc/apache2/apache.conf
/etc/apache2/sites-enabled/000-default
```

In our case, the below directives were present in the ``/etc/apache2/sites-enabled/000-default`` file.

```
DocumentRoot /var/www
<Directory />
        Options FollowSymLinks
        AllowOverride None
</Directory>
<Directory /var/www/>
        Options Indexes FollowSymLinks MultiViews Includes
        AllowOverride None
        Order allow,deny
        allow from all
</Directory>
```

This was changed to
```
DocumentRoot /var/www
<Directory />
        #Options FollowSymLinks
        Options -Indexes
        #AllowOverride None
</Directory>
<Directory /var/www/>
        #Options Indexes FollowSymLinks MultiViews Includes
        #AllowOverride None
        #Order allow,deny
        #allow from all
        Options -Indexes
</Directory>
```

and it worked!