---
layout: post
title: "Deploy to Amazon EC2 using GIT"
date: 2013-05-19 22:22
comments: true
categories: [amazon, aws, ec2, git]
---

[Git](http://git-scm.com/ "Git‎") is an amazing and indispensable tool. We use it for tracking all of our development work. It just becomes more cooler when you are able to push your code to a production or sandbox server with a simple [git push](http://git-scm.com/book/ch2-5.html‎).

Below listed are simple yet powerful steps to get you pushing your code to an Amazon EC2 server or any other linux server, where you have [SSH](http://en.wikipedia.org/wiki/Secure_Shell‎) access.

## Install git on your Server

From you local machine

```
ssh -i /path/to/my-amazon-key.pem ec2-user@example-webserver.amazonaws.com

// on your ec2 server
ubuntu@ip-xxx-xxx-xx-xxx:~$ sudo apt-get install git
```

## Setup SSH Access

You need SSH access to the server in order to push your code.
If you have more than one servers to push to, you need to manage more than one SSH keys. You can configure the SSH config file to do that job. Open up ``~/.ssh/config`` file and add this code for each of your hosts. Replace Host, Hostname, User and IdentityFile values with yours.

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

You need one git repo to push to from you work station. Let's set it up.
In directory /home/ubuntu/

```
mkdir -p git-repo/my-git-repo.git
cd git-repo/my-git-repo.git
git init --bare
```
Now, you remote git repo on EC2 is setup.

## Add Amazon EC2 git repo as remote in your local git repo

On your local machine's git repo, add Amazon EC2 git repo as a remote.

```
git remote add production my-prod-server:/home/ubuntu/git-repo/my-git-repo.git
```
Now, a remote git repo is setup and you can also use it as a private git repo as an alternative to Github.

Let's make it work to push the code to the webserver.

## Post-receive Hook on Amazon EC2 git repo

Up till now you've setup a working remote git repo on Amazon EC2 server. To push your code to the apache web directory ``/var/www/example.com/public_html/``, we'll need to add a [post-receive hook](http://git-scm.com/book/en/Customizing-Git-Git-Hooks).

Go to your git repo on Amazon EC2 ``/home/ubuntu/git-repo/my-git-repo.git``

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

The first ``if`` block will copy the working directory code of ``production`` or ``master`` branch of the repo, to the web server path whenever you push the code to this remote git repo.

Similarly, if you push a ``smoke`` named branch to the Amazon ec2 git repo, it will copy the code to smoke test directory ``/var/www/example.com/public_html/smoke/``.

So effectively, there are two purposes getting solved here. First, you can user the server for hosting your code's git repo. Second, for pushing the code to server directory without doing any FTP.

Happy Git-ting!