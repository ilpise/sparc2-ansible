<!-- <p align="center">
  <h1>sparc2-ansible</h1>
</p> -->

# sparc2-ansible

## Requirements
- Vagrant

	The

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

- Ansible
	Installation of ansible from [packages](http://docs.ansible.com/ansible/latest/intro_installation.html#latest-releases-via-apt-ubuntu)

- VirtualBox

# How to restore a vagrant box using vmdk disk

In order to restore an old sparc2-ansible machine from a vmdk disk proceed this way

1. Import the machine
Go to ~/VirtualBox VMs/<name of the machine>/ and double click on the file with vbox extension. This should open VirtualBox GUI and import the machine.

2. Create the box file using the virtual machine you just imported
Use the following command to create a box file.
```
vagrant package --base <name of running virtual machine> --output ubuntu.box
```

3. Verify the names of the boxes you have in vagrant with
```
vagrant box list
```

4. Import the base box giving it a unique name not in the box list (This way you are not going to override boxes)
For example to import the ubuntu.box into sparc  
```
vagrant box add "sparc" ./ubuntu.box
```
After the import you can verify the new box is present with
```
vagrant box list
```
You can see a box named sparc

5. Now in the sparc2-ansible.git directory open the Vagrant file and sobstitute the default box config.vm.box = "ubuntu/trusty64"
with the name of the new imported box
```
config.vm.box = "sparc"
```

6. you can now start your box in the usual way – using `vagrant up` command from inside the sparc2-ansible directory.

If during the boot process you experience this Warning
```
 default: Warning: Authentication failure. Retrying...
 default: Warning: Authentication failure. Retrying...
 default: Warning: Authentication failure. Retrying...
 default: Warning: Authentication failure. Retrying...
 default: Warning: Authentication failure. Retrying...
 default: Warning: Authentication failure. Retrying...
 default: Warning: Authentication failure. Retrying...
```

Shared folders will not be linked.

To solve the problem ssh in the guest machine remove the `.ssh/authorized_keys` file and reset it with
```
wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O /home/vagrant/.ssh/authorized_keys  
```

In order to verify the box you are running is the correct one use
```
vagrant global-status
```
it will output the status of the boxes
```
id       name    provider   state    directory                              
----------------------------------------------------------------------------
9de5394  default virtualbox poweroff /home/hostmachineusername/wfp/sparc2-ansible.git    
9d2bb89  default virtualbox running  /home/hostmachineusername/wfpose/sparc2-ansible.git
```


# Installation

Install ansible using pip and use the version 2.0.0.1

Create a local directory (e.g. wfp) to store the satellite projects around sparc-ansible.
All the project are copied in a directory with .git 'extension' according with the Vagrant file provided in sparc-ansible project.
These directories are __synced-folders__  in respect to Vagrant [see documentation](https://www.vagrantup.com/docs/synced-folders/).

>Synced folders are configured within your Vagrantfile using the config.vm.synced_folder method. Usage of the configuration directive is very simple:

```ruby
Vagrant.configure("2") do |config|
  # other config here

  config.vm.synced_folder "src/", "/srv/website"
end
```
The first parameter is a path to a directory on the host machine. If the path is relative, it is relative to the project root. The second parameter must be an absolute path of where to share the folder within the guest machine. This folder will be created (recursively, if it must) if it does not exist.

Looking to the Vagrant file in sparc2-ansible we have to sync the following folders

```ruby
config.vm.synced_folder "~/wfp/geodash-base.git", "/home/vagrant/geodash-base.git"
config.vm.synced_folder "~/wfp/public/geodash-framework-django.git", "/home/vagrant/geodash-framework-django.git"
config.vm.synced_folder "~/wfp/sparc2-core.js.git", "/home/vagrant/sparc2-core.js.git"
config.vm.synced_folder "~/wfp/sparc2-pipeline.git", "/home/vagrant/sparc2-core.js.git"
config.vm.synced_folder "~/wfp/sparc2-plugin-calendar.git", "/home/vagrant/sparc2-plugin-calendar.git"
config.vm.synced_folder "~/wfp/sparc2-plugin-sidebar.git", "/home/vagrant/sparc2-plugin-sidebar.git"
config.vm.synced_folder "~/wfp/public/sparc2.git", "/home/vagrant/sparc2.git"
```

Then from your home on the host machine create a wfp directory change dir into it and clone the packages

```
cd wfp

git clone https://github.com/wfp-ose/geodash-base.git geodash-base.git
git clone https://github.com/wfp-ose/geodash-framework-django.git geodash-framework-django.git
git clone https://github.com/wfp-ose/sparc2-core.js.git sparc2-core.js.git
git clone https://github.com/wfp-ose/sparc2-pipeline.git sparc2-pipeline.git
git clone https://github.com/wfp-ose/sparc2-plugin-calendar.git sparc2-plugin-calendar.git
git clone https://github.com/wfp-ose/sparc2-plugin-sidebar.git sparc2-plugin-sidebar.git
git clone https://github.com/wfp-ose/sparc2.git sparc2.git
```
Then clone the sparc2-ansible project
```
git clone https://github.com/karakostis/sparc2-ansible.git sparc2-ansible.git
```

We have to set the right branch for geodash-framework-django to sparc2
```
cd geodash-framework-django.git
git checkout sparc2
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
```
vagrant ssh
```
> Inside the virtual machine run the following command

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

enter the sparc2 virtualenvironment

```
workon sparc2
```

>Run the following commands in order to install geodash-framework-django
```
/home/vagrant/.venvs/sparc2/bin/pip install -e git+https://github.com/wfp-ose/geodash-framework-django.git@sparc2#egg=geodash-framework-django
```

Install the geodash-framework-django from local downloaded repository (be shure you are in the sparc2 branch)
```
(sparc2)vagrant@vagrant-ubuntu-trusty-64:~$ pip install -e git+file:///home/vagrant/geodash-framework-django.git#egg=geodash-framework-django
Obtaining geodash-framework-django from git+file:///home/vagrant/geodash-framework-django.git#egg=geodash-framework-django
  Cloning file:///home/vagrant/geodash-framework-django.git to ./.venvs/sparc2/src/geodash-framework-django
Installing collected packages: geodash-framework-django
  Running setup.py develop for geodash-framework-django
Successfully installed geodash-framework-django
```

### Postgresql configuration

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

### Supervisord

The supervisord is started using a configuration file inside the sparc2.git project.

Verify the supervisord Status
```
supervisorctl status
```
If you have an error about `http://127.0.0.1:9001 refused connection` stop all the supervisord process and restart it.

Before restart verify the configuration file in

```
/home/vagrant/sparc2.git/supervisord.conf
```

the line about the command related with gunicorn   
```
command=/home/vagrant/.venvs/sparc2/bin/gunicorn sparc2.wsgi -c /home/vagrant/sparc2.git/gunicorn.conf.py
```

Restart supervisord passing the sparc2.git configuration file
```
supervisord -c /home/vagrant/sparc2.git/supervisord.conf
```

you can verify the status of the program launched from supervisord using the `supervisorctl stasus` command that have to return something like

```
sparc2:gunicorn                  RUNNING    pid 2780, uptime 0:16:12
sparc2:memcached                 RUNNING    pid 2728, uptime 0:16:20
```

### collectstatic
Change directory in `/home/vagrant/sparc2.git` where the `manage.py` file is located and run collectstatic to create Django static files
```
sudo /home/vagrant/.venvs/sparc2/bin/python manage.py collectstatic --noinput -i gulpfile.js -i package.json -i temp -i node_modules
```

### Run the application
Be sure have the sparc2 virtualenv active.

Start the application on all the network interfaces port 8000
```
python manage.py runserver 0.0.0.0:8000
```
> ImportError: cannot import name GEOMETRY_TYPE_TO_OGR

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
