---
layout: post
title:  "PostgreSQL setup with Vagrant"
date:   2015-03-22 17:40:57 +0000
categories: TECHNOTES POSTGRE
---

# Setting up PostgreSQL and accepting remote connections
## Vagrant VM and PostgreSQL setup
```
vagrant init
```
Places the vagrant file in the directory.
```
vagrant box add ubuntu/trusty64
```
Just if the box isn't already installed. Now we need to edit Vagrantfile in order to specify the kind of operating
system that we want to use. For Ubuntu Server 64 bit use ubuntu/trusty64. This can be convinient to mantain your
testing environment as similar as possible with production.

```
vagrant up
```

In order to run the VM. Limitations over the physical size of the machine can be stablished via Vagrantfile.

```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.provider "virtualbox" do |v|
    v.memory = 256
    v.cpus = 1
  end
end
```

Install PostgreSQL
```
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

Among other things, a new cluster will be created:
```
Creating new cluster 9.3/main ...
  config /etc/postgresql/9.3/main
  data   /var/lib/postgresql/9.3/main
  locale en_US.UTF-8
  port   5432
```

## Creating users and databases

First, create a new operating system user (you'll need to set the password too):
```
sudo adduser mydb_user
```
Change to the special postgres super user with:
```
sudo -i -u postgres
```
The user has special privileges and let's you access to the postgres client, called **psql**. Template1 is the
default system database:
```
psql template1
```
Now, create the user for the database and the database itself:
```
CREATE USER mydb_user WITH PASSWORD 'XXXXXX';
CREATE DATABASE mydb;
GRANT ALL PRIVILEGES ON DATABASE mydb to mydb_user;
\q
su - mydb_user
$ psql -d mydb -U mydb_user
```
## Remote connections
```
`initialize': could not connect to server: Connection refused (PG::ConnectionBad)
```
Yo should add to the Vagrantfile the following line that will open the default port and will allow us access from
outside:
```
#Assuming that you want to mantain the same port number
config.vm.network :forwarded_port, host: 5432,  guest: 5432
```
Localize PostgreSQL configuration:
```
sudo vim /etc/postgresql/9.3/main/postgresql.conf
# You need to find the line listen_addresses='localhost'. Using Vim, you can type "/listen_addresses".
# Now you need to change it from
# #listen_addresses = 'localhost'         # what IP address(es) to listen on;
# to:
listen_addresses = '*'         # what IP address(es) to listen on;
# VERY IMPORTANT: Remove the dash (which makes the line a comment)
# Restart the service:
sudo service postgresql restart
```
At this point, the error message when trying to connect should have changed:
```
FATAL:  no pg_hba.conf entry for host "XX.XX.XX.XX", user "mydb_user", database "mydb",
SSL on (PG::ConnectionBad)
```
In the VM, let's go to pg_hba.conf, which is located on the same folder than postgresql.conf:
```
sudo vim /etc/postgresql/9.3/main/pg_hba.conf
#Add the following line before the line that accepts 127.0.0.1
host    all             all             XX.XX.XX.XX/32             md5
#Restart
sudo service postgresql restart
```
