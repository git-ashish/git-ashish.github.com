---
layout: post
title: "Deploy to Amazon EC2 using GIT"
date: 2013-12-02 18:04
comments: true
categories: [amazon, aws, ec2, git]
---

[Git](http://git-scm.com/ "Git‎") is an amazing and indispensable tool. We use it for tracking all of our development work. It just becomes more cooler when you are able to push your code to a production or sandbox server with a simple [git push](http://git-scm.com/book/ch2-5.html‎).

Below listed are simple yet powerful steps to get you pushing your code to an Amazon EC2 server or any other linux server, where you have [SSH](http://en.wikipedia.org/wiki/Secure_Shell‎) access.

## Install git on your Server

First SSH into your remote server from you local machine. Use the private key file (.pem key) that you've downloaded from the Amazon EC2 server setup process on aws console. If you don't have it yet, Go to your aws console -> EC2 server -> Network & Security -> Key Pairs -> Create Key Pair

Once you've created the private key, it'll automatically be downloaded to your machine. Move it a folder where you wish to keep the key. Also, change its permission to 0400

```
chmod 0400 /path/to/my-amazon-key.pem
```

Now, lets use this private key to connect to your newly created ec2 server.

Open console and type:


```
ssh -i /path/to/my-amazon-key.pem ec2-user@example-webserver.amazonaws.com
```

If you've downloaded and set key's permission correctly, you should now be on the ec2 server's console. Now, install git on the ec2 server.

```
// on your ec2 server
ubuntu@ip-xxx-xxx-xx-xxx:~$ sudo apt-get install git
```

Perfect! You're one step closer to push your local git repo to the ec2 server.


## Setup SSH Access for different ec2 servers

Say, you've more than one ec2 servers. To make your life easier, it is desirable to set up a SSH config file which basically holds information needed to SSH into your remote ec2 server.

On your local machine, open file [or create one] ``~/.ssh/config`` and add this code for each of your ec2 servers. Replace Host, Hostname, User and IdentityFile values with yours.

```
Host my-prod-server
Hostname example-webserver.amazonaws.com
User ec2-user
IdentityFile /path/to/my-amazon-key.pem

Host my-sandbox-server
Hostname example-webserver.amazonaws.com
User ec2-user
IdentityFile /path/to/my-sandbox-amazon-key.pem
```

## Test the connection
```
ssh my-prod-server
```

Hurray, if you get connected to your server otherwise debug using this.
```
ssh -Tvvv my-prod-server
```
<!-- more -->

## Setup Git Repo on Amazon EC2 Server

By now, you should have access to your ec2 server from your local machine and Git installed on your ec2 server. The next step is to create a Git repository on your remote ec2 server, to which we'll be pushing all your code.

Let's set it up.

SSH into your remote ec2 server and create a directory for holding all such git repos. 
Assuming you're creating this directory under /home/ubuntu/ which is the home directory of default user ubuntu.

```
mkdir -p git-repo/my-git-repo.git
cd git-repo/my-git-repo.git
git init --bare
```

That's it! Now, your remote Git repo on the ec2 server is setup.

## Add Amazon EC2 git repo as remote in your local git repo

We now need to make this Git repo on the ec2 server as a remote repository of our local Git repo.

On your local machine's git repo, add Amazon EC2 git repo as a remote.

```
// on your local machine

git remote add production my-prod-server:/home/ubuntu/git-repo/my-git-repo.git
```

Beauty! Your remote Git repo is setup and you can also use it as a private git repo, an alternative to Github paid plan ;)

Let's make this remote git repo to push the code to the http server on the same ec2 server machine.

## Post-receive Hook on Amazon EC2 git repo

Up till now you've setup a working remote git repo on Amazon EC2 server. To push your code to the apache web directory ``/var/www/example.com/public_html/``, we'll need to add a [post-receive hook](http://git-scm.com/book/en/Customizing-Git-Git-Hooks).

Go to your git repo on Amazon EC2 server ``/home/ubuntu/git-repo/my-git-repo.git``

```
cd /home/ubuntu/git-repo/my-git-repo.git

vi hooks/post-receive
```

Add following code in the file

```
#!/bin/bash

while read oldrev newrev ref
do
  branch=`echo $ref | cut -d/ -f3`
  if [ "production" == "$branch" -o "master" == "$branch" ]; then
    
    git --work-tree=/var/www/example.com/public_html/ checkout -f $branch
    
    echo 'Changes pushed to Amazon EC2 PROD.'
  fi

  if [ "smoke" == "$branch" ]; then
    
    git --work-tree=/var/www/example.com/public_html/smoke/ checkout -f $branch
    
    echo 'Changes pushed to Amazon EC2 PROD smoke box.'
  fi
done
```

The first ``if`` block will copy the working directory code of ``production`` or ``master`` branch of the repo, to the web server path whenever you do a push to this remote git repo from your local git repo.

Similarly, if you push a ``smoke`` named branch to the Amazon ec2 git repo, it will copy the code to smoke test directory ``/var/www/example.com/public_html/smoke/`` on the ec2 server.

So effectively, there are two purposes getting solved here. First, you can user the server for hosting your code's git repo. Second, for pushing the code to web server directory without doing any FTP anytime you make some changes to reflect on the ec2 server.

Happy Git-ting!