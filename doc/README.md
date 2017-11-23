<!-- <p align="center">
  <h1>sparc2-ansible</h1>
</p> -->

# sparc2-ansible

## Vagrant
Installation of Vagrant from binary packages https://www.vagrantup.com/downloads.html

Installed version

```
~$ vagrant --version
Vagrant 2.0.1
```

In Sparc2 the vagrant file specify

+ An ubuntu 14.04 server 64 bit __box__

	Boxes are the package format for Vagrant environments. A box can be used by anyone on any platform that Vagrant supports to bring up an identical working environment.

	The easiest way to use a box is to add a box from the [publicly available catalog of Vagrant boxes](href="https://vagrantcloud.com/boxes/search">)

+ An *ansible* __provisioner__

	Provisioners in Vagrant allow you to automatically install software, alter configurations, and more on the machine as part of the vagrant up process.


By default, VirtualBox is the default __provider__ for Vagrant.

## Ansible
Installation of ansible from [packages](http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-via-apt-ubuntu)

# Restore an old version of sparc-ansible

Install ansible using pip and use the version 2.0.0.1

Create a local directory (e.g. wfp) to store the satellite projects around sparc-ansible.
All the project are copied in a directory with .git 'extension' according with the Vagrant file provided in sparc-ansible project.
These directories are __synced-folders__  in respect to Vagrant [see documentation](https://www.vagrantup.com/docs/synced-folders/)
```
cd wfp

git clone https://github.com/wfp-ose/geodash-base.git geodash-base.git
git clone https://github.com/wfp-ose/geodash-framework-django.git geodash-framework-django.git
git clone https://github.com/wfp-ose/sparc2-core.js.git sparc2-core.js.git
git clone https://github.com/wfp-ose/sparc2-pipeline.git sparc2-pipeline.git
git clone https://github.com/wfp-ose/sparc2-plugin-calendar.git sparc2-plugin.calendar.git
git clone https://github.com/wfp-ose/sparc2-plugin-sidebar.git sparc2-plugin.sidebar.git
git clone https://github.com/wfp-ose/sparc2.git sparc2.git
```
Then clone the sparc2-ansible project
```
git clone https://github.com/karakostis/sparc2-ansible.git sparc2-ansible.git
```
change dir in sparc2-ansible.git
```
cd sparc2-ansible.git
```
run
```
vagrant up
```

> __If you have an error related to GDAL:__

> Access the virtual machine running
> ```
  vagrant ssh
  ```
> Inside the virtual machine run the following command
>
  ```
  /home/vagrant/.venvs/sparc2/bin/pip install --no-install GDAL==2.0.0
  cd /home/vagrant/.venvs/sparc2/build/GDAL
  /home/vagrant/.venvs/sparc2/bin/python setup.py build_ext --include-dirs=/usr/include/gdal
  /home/vagrant/.venvs/sparc2/bin/pip install --no-download GDAL==2.0.0
  pip install django-leaflet
  pip install django==1.9.6
  ```

## Install the framework
Access the virtual machine running
```
vagrant ssh
```
Run the following commands
```
/home/vagrant/.venvs/sparc2/bin/pip install -e git+https://github.com/wfp-ose/geodash-framework-django.git@sparc2#egg=geodash-framework-django
```
Modify the postgresql configuration

```
sudo nano /etc/postgresql/9.3/main/pg_hba.conf
```
```
# --------------------------------------------------------------------------------
...
# Database administrative login by Unix domain socket
local   all             postgres                                md5

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5
local   sparc2          sparc2                                  md5
host    sparc2          sparc2           127.0.0.1/32           md5
host    sparc2          sparc2           10.0.2.2/32            md5
host    all             all              all                    password
# --------------------------------------------------------------------------------
```
Restart postgresql
```
brew services restart postgresql
```
Modify the file supervisord.conf
```
nano supervisord.conf
```
Modify the following row this way
```
command=/home/vagrant/.venvs/sparc2/bin/gunicorn sparc2.wsgi -c gunicorn.conf.py
```
Restart supervisord
```
supervisord -c /home/vagrant/sparc2.git/supervisord.conf
```
Verify the supervisord Status
```
supervisorctl status
```
If you have an error about occupied port stop all the supervisord process and restart it

Run collectstatic to create Django static files
```
sudo /home/vagrant/.venvs/sparc2/bin/python manage.py collectstatic --noinput -i gulpfile.js -i package.json -i temp -i node_modules
```
Start the application on all the network interfaces port 8000
```
python manage.py runserver 0.0.0.0:8000
```
Verify the access to the postgresql database ?from the vagrant user?.
Dump the database
```
pg_dump sparc2 > sparc2_backup.sql
```
Became postgres and fill the sparc2 database with the backup

```
sudo su postgres (modificare eventualmente pg_hba.conf mettendo trust al posto di md5 per le connessioni local)

psql -d sparc2 -f sparc2_backup.sql
```

Verify Django static files and recreate them if necessary (e.g. no icons in home page)
```
sudo /home/vagrant/.venvs/sparc2/bin/python manage.py collectstatic --noinput -i gulpfile.js -i package.json -i temp -i node_modules
```
Restart the application
```
python manage.py runserver 0.0.0.0:8000
```
At the first start requests are very slow because chache has to be generated
