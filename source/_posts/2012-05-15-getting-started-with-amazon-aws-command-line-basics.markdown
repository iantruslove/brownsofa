---
author: Ian
comments: true
date: 2012-05-15 03:10:55+00:00
layout: post
slug: getting-started-with-amazon-aws-command-line-basics
title: Getting Started with Amazon AWS Command Line Basics
wordpress_id: 346
categories:
- aws
- ec2
- programming
- sysadmin
- web services
---

I didn't even realize there's a set of command line tools for working with Amazon's Web Services.  It only took a few mins to get started, but it wasn't immediately apparent what needed to happen.

_I'm using OSX Lion.  If you're using another *nix this is all fairly easy to translate. I have no idea how this could be Windows-ified..._


## Download and Install


I downloaded the tools as a zip archive from [http://aws.amazon.com/developertools/351](http://aws.amazon.com/developertools/351).  I like to keep this kind of package within a `~/local` folder, so:

    
    mkdir ~/local
    cd Downloads
    unzip ec2-api-tools.zip
    mv ec2-api-tools-1.5.3.1 ~/local


To make upgrades easier, create a symlink to that directory:

    
    cd ~/local
    ln -s ec2-api-tools-1.5.3.1 ec2-api-tools




## Security


You need your username and password to get into the AWS web interface.  You'll still need to authenticate with the command line tools, but you have a more convenient mechanism to do this - X.509 keys.

Log into the AWS web page, and go to **My Account** > **Security Credentials**.  Go to the **X.509 Certificates** tab and hit **Create A New Certificate**.  Download the private key and the certificate and save them somewhere safe - since it already has appropriate permissions and contains keys, I save mine to `~/.ssh/pk.pem` and `~/.ssh/cert.pem` respectively.


## Configure


The tools require you to set `JAVA_HOME` and `EC2_HOME` environment variables, and there are a couple of other variables that will save you a whole lot of typing. Add the following to the end your `~/.bash_profile`:

    
    export PATH=$HOME/local/ec2-api-tools/bin:$PATH
    export JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home
    export EC2_HOME=$HOME/local/ec2-api-tools
    export EC2_PRIVATE_KEY=~/.ssh/pk_ec2.pem
    export EC2_CERT=~/.ssh/cert_ec2.pem
    export EC2_URL=https://ec2.us-west-1.amazonaws.com


Be careful with the last of these.  I host my instances in the US-West-1 region - you will likely need to figure out which URL to put here based on the region your instances or volumes are at.  If you have machines in multiple regions, use the `--region` switch in the commands instead.

Source the file to have the changes take effect:

    
    . ~/.bash_profile


and carry on.


## Run


Now, it's easy.  For example, run `ec2-describe-instances` - it should show you a list of all your EC2 instances, and a whole bunch of info about them.  I leave it to you to work out the other 280-odd commands.  Looks like you can do pretty much do any operation to manage EC2 instances, images, volumes, networking, snapshots, ...
