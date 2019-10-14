---
layout:     post
title:      Hadoop Installation Guidance for Dummies
subtitle:   Step by Step Showing How To Install Hadoop in Fully Distributed Mode on AWS
date:       2019-10-13
author:     Susu
header-img: img/posts20191013/header_bg.jpg
catalog: 	 true
tags:
    - Big Data
    - Hadoop
    - Hadoop Installation
---

## Before Start

Having some virtual machines (VM) launched on Amazon Web Service (AWS).  

I launched four VMs, you can choose to launch more.  And the Hadoop Version I installed is Hadoop-2.6.5, this configuration works for Hadoop-2.X.X versions.

You can take my blog as an example and configure your VMs.

**Note: recently I noticed that if you keep your Hadoop idling on AWS, AWS may consider the connection between your VMs as DOS Attack and shut down network ports used by Hadoop.**

### Objectives

Build fully distributed Hadoop on the cluster formed by these four VMs.

## SSH Passphrase Free Connection

**For each VM**, type the following command in the terminal to generate a pair of RSA keys (i.e., public key and private key):

`ssh-keygen`

Then press "enter key" for file and passphrase, which means setting these values as default.             

![](https://github.com/Pollyanna-Ye/Pollyanna-Ye.github.io/blob/master/img/posts20191013/001.jpg)

Then type the following in the Terminal of each VM:

`cd ~/.ssh`

By typing this, you enter the default directory of ssh, where should have at least two files now: `id_rsa` and `id_rsa.pub`, you may also have another file named `know_hosts` or `authorized_keys`.

**RSA Encryption Algorithm** allows us to use public and private key pairs to safely connect two hosts.  Private key, which saves in `id_rsa` should be kept in your own computer, and public key, which saves in `id_rsa.pub`, are shared with other hosts (computers).  By doing this, other computers can use this public key to communicate with your computer safely.  Surely, you have to make sure only give your public key to the computers you trust.

And the file `known_hosts` or `authorized_keys` are used to save other computers' public keys.    So if computer A has computer B's public key save in A's `authorized_keys` file, then computer A and computer B can communicate with each other safely.

**So, obviously, we have to share public keys among all VMs, so that they can communicate with each other.** Because Hadoop is a parallel computing engine, all the works are done by computers in the cluster.  If these computers can not communicate with each other, how can they cooperate and complete their jobs.  Just in case you may need to use localhost, you can also save one copy of each computer's public key to its `authorized_keys`.

Assuming you are not familiar with command line, you can just use copy and paste to do this public key sharing.  **After doing that, for each computer, it should have four (I have four VMs) public keys save in `authorized_keys` files.**

After doing that, now you have successfully configured SSH passphrase free connection between all VMs.  Now I can SSH to the computer with private ip address "172.31.11.234" using the following command:

`ssh 172.31.11.234`

Certainly, I can also use its public address.

**However, typing IP address is tedious and prone to make mistakes.**. So we can save the name of each computer and their corresponding IP addresses in a "phone book", and connect to each other using their name instead of typing ip addresses.

**Do the following for all VMs.** We can type the following command in the Terminal to edit  "phone book" of each computer:

`sudo vim /etc/hosts`

Add the following entries into it:

![](https://github.com/Pollyanna-Ye/Pollyanna-Ye.github.io/blob/master/img/posts20191013/002.jpg)

These four lines are the contact info for each computer.  For instance, the IP address "172.31.11.234" is for the computer named "master".

After adding these four contact info in each computer's phone book.  We can use the following command to connect to another computer:

`ssh master (or slave1, slave2, slave3)`

## Install JAVA

**For each VM, you have to install JAVA. So the following commands are necessary for all VMs.**

First of all, type the following command in the Terminal to update the repository of apt-get (which can be considered as App Store in your iphone):

`sudo apt-get update`

Then type the following command to 