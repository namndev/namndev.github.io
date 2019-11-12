---
layout: post
title: How to Install MongoDB on Ubuntu 14.04 LTS
image: https://stackjava.com/wp-content/uploads/2018/07/mongodb.png
# subtitle: A collection of awesome gists. Feel free to contribute.
# gh-repo: daattali/beautiful-jekyll
# gh-badge: [star, fork, follow]
tags: [MongoDB, Linux]
comments: true
---

[__MongoDB__](https://www.mongodb.com/what-is-mongodb) is a `NoSQL` database intended for storing large amounts of data in document-oriented storage with dynamic schemas. NoSQL refers to a database with a data model other than the tabular format used in relational databases such as __MySQL__, __PostgreSQL__, and __Microsoft SQL__. 

`MongoDB`'s features include:

- `Full index support`
- `Replication`
- `High availability`
- `Auto-sharding`

## PRE-FLIGHT CHECK

- These instructions are intended for installing `MongoDB` on a single `Ubuntu 14.04 LTS` node.
- I’ll be working from a Liquid Web Core Managed `Ubuntu 14.04 LTS` server, and I’ll be logged in as a non-root user, but with sudo access.

## SETUP A THE PACKAGE DATABASE
First we’ll import the `MongoDB` public key used by the package management system:

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
```

### Create a list file for MongoDB.

Create the `/etc/apt/sources.list.d/mongodb-org-3.2.list` list file using the command appropriate for your version of Ubuntu:

__Ubuntu 12.04__

```bash
echo "deb http://repo.mongodb.org/apt/ubuntu precise/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
```

__Ubuntu 14.04__

```bash
echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
```

Now reload the package database:

```bash
sudo apt-get update
```

### INSTALL LATEST STABLE VERSION MONGODB

At this point, installing MongoDB is as simple as running just one command:

```bash
sudo apt-get install -y mongodb-org
```

If you’d like MongoDB to auto-update with apt-get than you’re done with the installation. But, it’s possible to ‘pin’ the version of MongoDB you just installed to prevent apt-get from auto-updating.

```bash
echo "mongodb-org hold" | sudo dpkg --set-selections
echo "mongodb-org-server hold" | sudo dpkg --set-selections
echo "mongodb-org-shell hold" | sudo dpkg --set-selections
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```

### GET MONGODB RUNNING

_Start-Up MongoDB:_

```bash
sudo service mongod start
```

_Check MongoDB Service Status:_

```bash
sudo service mongod status
```

_Summary List of Status Statistics (Continuous):_

```bash
mongostat
```

_Summary List of Status Statistics (5 Rows, Summarized Every 2 Seconds):_

```bash
mongostat --rowcount 5 2
```

_Enter the MongoDB Command Line:_

```bash
mongo
```

By default, running this command will look for a `MongoDB` server listening on port `27017` on the localhost interface.
 
If you’d like to connect to a MongoDB server running on a different port, then use the `–port` option. For example, if you wanted to connect to a local `MongoDB` server listening on port `22222`, then you’d issue the following command:

```bash
mongo --port 22222
```

_Shutdown MongoDB:_

```bash
sudo service mongod stop
```

_Restart MongoDB:_

```bash
sudo service mongod restart
```

## MongoDB Munin Plugin

A [plugin](https://github.com/comerford/mongo-munin) is available that provide metrics for:

- B-Tree stats
- Current connections
- Memory usage
- Database operations (inserts, updates, queries etc.)

The plugin can be installed on each node where MongoDB. For requirements and instructions to installing the MongoDB Munin Plugin.

### CHECK YOUR SETUP

After installing the plugin and making the configuration changes, force the server to update the information to check that your setup is correct using the following:

```bash
sudo -u munin /usr/share/munin/munin-update
```

If everything is set up correctly, you will get a chart like this:

<p align="center">
    <img src="/img/2016/munin-configuration-screen-shot.png" />
</p>

__PLUGINS__
- mongo_ops : operations/second
- mongo_mem : mapped, virtual and resident memory usage
- mongo_btree : btree access/misses/etc...
- mongo_conn : current connections
- mongo_lock : write lock info
- mongo_docs : number of documents (inserted, updated...)

__REQUIREMENTS__

- MongoDB 2.4+
- python/pymongo

### INSTALLATION (UBUNTU)

__Install pip:__

```bash
sudo apt-get install build-essential python-dev python-setuptools
sudo easy_install pip
sudo pip install --upgrade virtualenv 
```

### Install pymongo:

```bash
sudo pip install pymongo
```

### Install plugins:

``bash
git clone https://github.com/comerford/mongo-munin.git /tmp/mongo-munin
sudo cp /tmp/mongo-munin/mongo_* /usr/share/munin/plugins
sudo ln -sf /usr/share/munin/plugins/mongo_btree /etc/munin/plugins/mongo_btree
sudo ln -sf /usr/share/munin/plugins/mongo_conn /etc/munin/plugins/mongo_conn
sudo ln -sf /usr/share/munin/plugins/mongo_lock /etc/munin/plugins/mongo_lock
sudo ln -sf /usr/share/munin/plugins/mongo_mem /etc/munin/plugins/mongo_mem
sudo ln -sf /usr/share/munin/plugins/mongo_ops /etc/munin/plugins/mongo_ops
sudo ln -sf /usr/share/munin/plugins/mongo_docs /etc/munin/plugins/mongo_docs
sudo chmod +x /usr/share/munin/plugins/mongo_*
sudo service munin-node restart
```

_Check if plugins are running:_

```bash
munin-node-configure | grep "mongo_"
```

_Test plugin output:_

```bash
munin-run mongo_ops
```

### CONFIGURATION
__How to configure custom db connection?__

munin-node can set env value in below file: `/etc/munin/plugin-conf.d/munin-node`

```bash
[mongo_*]
env.MONGO_DB_URI mongodb://user:password@host:port/dbname
```

> Thanks,