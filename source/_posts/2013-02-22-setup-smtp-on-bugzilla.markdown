---
layout: post
title: "Setup SMTP on Bugzilla"
date: 2013-02-22 23:33
comments: true
categories: bugzilla
---

Our startup just celebrated its 1st anniversary and we marked this by making [bugzilla](http://www.bugzilla.org) our primary bug tracker. The bugzilla [docs](http://www.bugzilla.org/docs/4.2/en/html/index.html) will help you setting it up.

We are hosting it on Ubuntu 12.04 LTS 64bit Amazon EC2 micro instance and using Amazon RDS mysql as backend. For sending bugzilla emails alerts, we choose SMTP.

We followed this [guide](http://dawood.in/bugzilla-alerts-using-gmail/) but ran into some problems on the first step of installing perl module [Email::Send::SMTP::TLS](http://search.cpan.org/~fayland/Email-Send-SMTP-TLS-0.04/lib/Email/Send/SMTP/TLS.pm). The error we got was

_Building and testing Net-SSLeay-1.52 ... FAIL_

This probably means that your Ubuntu is missing the Perl module for Secure Sockets Layer (SSL). You install it like this.

```
sudo apt-get install libnet-ssleay-perl
```

Next, we go ahead with the installation of Email::Send::SMTP::TLS module.

```
sudo apt-get install cpanminus
sudo cpanm Email::Send::SMTP::TLS
```

Now, continue following this [guide](http://dawood.in/bugzilla-alerts-using-gmail/)