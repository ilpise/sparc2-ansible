# Sparc2 project

# Architecture

- [sparc2-ansible](#sparc2-ansible) - is the virtual machine implementation of the project. It is a Vagrant ????? then it can be run from scratch resulting in the installation of the machine with it's packages or to restart the machine after the provisioning

- [sparc2](#sparc2) - Is a mix of python Django environment and a geodash based application

- [geodash](https://github.com/geodashio) - Is a web framework implemented using angular.js openlayers3 on the client side. It is composed by a lot of packages to build a rich interface.

# sparc2-ansible

This is the starting point. With this project we are going to create a virtual machine with all the needed packages installed.

It is the point to start the virtual machine we are going to work on. (Switching environment)

In the Vagrant file is possible to share folders in order to make changes to local files and recompile the project.


## Requirements
- Vagrant

	The 	Installation of Vagrant from binary packages https://www.vagrantup.com/downloads.html

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


## Install new version of node
If starting from an old implementation of the machine we have to upgrade nodejs.
This steps explain how to unistall the nodejs and install a new nodejs environment with nvm.

First we have to remove the old node/nodejs provided from the chrislea repository

unistall nodejs
```
$ sudo apt-get remove nodejs
```
remove the chrislea repositories from the sources.list.d folder
```
vagrant@vagrant-ubuntu-trusty-64:/etc/apt/sources.list.d$ sudo rm -rf ppa_chris_lea_node_js_trusty.list*
```
remove the chrislea apt key from the list

get the list
```
$ sudo apt-key list
```
it return
```
/etc/apt/trusted.gpg
--------------------
pub   1024D/437D05B5 2004-09-12
uid                  Ubuntu Archive Automatic Signing Key <ftpmaster@ubuntu.com>
sub   2048g/79164387 2004-09-12

pub   1024D/FBB75451 2004-12-30
uid                  Ubuntu CD Image Automatic Signing Key <cdimage@ubuntu.com>

pub   4096R/C0B21F32 2012-05-11
uid                  Ubuntu Archive Automatic Signing Key (2012) <ftpmaster@ubuntu.com>

pub   4096R/EFE21092 2012-05-11
uid                  Ubuntu CD Image Automatic Signing Key (2012) <cdimage@ubuntu.com>

pub   1024R/314DF160 2009-05-10
uid                  Launchpad ubuntugis-stable

pub   1024R/C7917B12 2010-05-17
uid                  Launchpad chrislea
```
to remove the chrislea key use
```
$ sudo apt-key del C7917B12
```

## Installation using nvm

1. Get and install the latest tagged version of nvm (0.33.6) actually
`vagrant@vagrant-ubuntu-trusty-64:~$ sudo wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.6/install.sh | bash`
2. Activate nvm
`vagrant@vagrant-ubuntu-trusty-64:~$ source ~/.bash_profile`
or you can exit the actuall shell and login again to activate
3. Get a list of nodejs pachages
`vagrant@vagrant-ubuntu-trusty-64:~$ nvm ls-remote --lts`
4. Install the latest LTS Boron
```
vagrant@vagrant-ubuntu-trusty-64:~$ nvm install v6.12.0
Downloading and installing node v6.12.4...
Downloading https://nodejs.org/dist/v6.12.0/node-v6.12.0-linux-x64.tar.xz...
######################################################################## 100,0%
Computing checksum with sha256sum
Checksums matched!
Now using node v6.12.0 (npm v3.10.10)
Creating default alias: default -> v6.12.0
```
5. Set the default node to the installed LTS
```
vagrant@vagrant-ubuntu-trusty-64:~$ nvm use v6.12.0
Now using node v6.12.0 (npm v3.10.10)
vagrant@vagrant-ubuntu-trusty-64:~$ nvm alias default v6.12.0
default -> v6.12.0
```

# sparc2
sparc2 is composed with two main parts

- In the root of the project we have the __manage.py__ file used to run the web interface and the collectstatic.

- In the `sparc2/static/sparc2` directory we have a [geodash.io based application](#geodashio-sparc2-application).

The geodash.io sparc2 application can be compiled using gulp.

## compile geodash.io sparc2 application
This paragraph explain how to compile the sparc2 application with gulp on a working machine. (You need nodejs version 6 to compile with gulp)

You need gulp installed globally. To install gulp globally
```
vagrant@vagrant-ubuntu-trusty-64:~$ npm install gulp -g
```

Compile sparc2 with gulp
	> If you have not installed the node packages you have to run 
	```
	vagrant@vagrant-ubuntu-trusty-64:~/sparc2.git/sparc2/static/sparc2$ npm install
	```
Compilation 

```
vagrant@vagrant-ubuntu-trusty-64:~/sparc2.git/sparc2/static/sparc2$ gulp default
```

Once compiled remember to move the static files from the build directory to the www directory using collectstatic to create static files
```
vagrant@vagrant-ubuntu-trusty-64:~/sparc2.git$ sudo /home/vagrant/.venvs/sparc2/bin/python manage.py collectstatic --noinput -i gulpfile.js -i package.json -i temp -i node_modules
```

## geodash.io sparc2 application
The geodash.io sparc2 application is in `/sparc2/static/sparc2` directory which contain the `gulpfile.js` and the `config.yml` - this is the core of the sparc2 project.
- __gulpfile.js__ is used to run tasks to compile and create files inside the build directory
- __config.yml__ file is used to configure path about the compilation.

Inside the `config.yml` file we have the inclusion of the following sections
- name, version, description sections
```
name: sparc2
version: "0.0.1"
description: "SPARC 2.x"
```
This parts describe the name version and description of the project.

- <a id="path">path</a> section
```
path:
  geodash: "./src/geodash"
```
The path instruction define a name (geodash) and it's relative path. This path is the path related to the [plugins](#plugins) section

- less section
	TBD

- bootstrap section
	TBD

- <a id="plugins">plugins</a> section
	
	When a plugin is required the gulp process look recursively inside the for it's config.yml file.
	
	If inside plugins section we have a *single name* the path where to find the code is given from the previous [path](#path) section inside the plugins directory.

	If we have a *file* the path is fully defined

	if we have a *git url* that is the code we are going to bring for the plugin

	Here is an example from sparc2 config.yml
	```
	plugins:
	  - sparc2 # The code is in ./src/geodash/plugins
	  - file:///home/vagrant/geodash-plugin-map-map.git # The code is in the vagrant user home directory
	  - { branch: master, url: https://github.com/wfp-ose/sparc2-plugin-welcome.git } # The code is taken from github. NOTE sometimes the `branch` name is substitute with `version`
	```	

	The plugins loaded from sparc2 are the following
	- sparc2
	- [geodash-plugin-map-map](#geodash-plugin-map-map)
	- [sparc2-plugin-sidebar](#sparc2-plugin-sidebar)
	- [sparc2-plugin-welcome](#sparc2-plugin-welcome)
	- [geodash-plugin-main](#geodash-plugin-main)

- <a id="dependencies">dependencies</a> section

```
dependencies:
  production:
    javascript:
      - "./src/js/main/*.js"
      - "./build/meta/meta.js"
      - "./build/templates/templates.js"
    templates:
      - "./src/templates/*.html"
    project:
      - "~/geodash-base.git/config.yml" # the ~ path is referred to the vagrant user home directory. the config.yml file contains references to (see it)
  test:
    javascript:
      - "./src/js/main/*.js"
      - "./src/js/polyfill/*.js"
```
The `production: project:` element point to the `config.yml` file of [geodash-base](#geodash-base) project where is defined the __geodash__ name used in the `build: monolith.js: src: project:` as explained in the [build](#build) section

- <a id="build">build</a> section
	```
	build:
	  ...
	  monolith.js:
		name: monolith.js
		type: js
		src:
		  - type: resource
		    project: geodash # This refer to the geodash-base project where we can find all the resources and version requested here inside the lib directory
		    version: "1.9.1"
		    name: jquery.js
		  ...	
		  - type: build
		    name: monkeypatch.js
		  - type: resource
		    project: geodash
		    version: "0.0.1"
		    name: geodash.js
		  - type: resource
		    project: sparc2
		    version: "0.0.1"
		    name: sparc2-core.js
		  - type: build
		    name: main.js
		outfile: sparc2.full.js
		dest: ./build/js/

	```
	In this section is defined the compilation chain about css and javascript.

	At the `monolith.js` level are defined `src` of `type: resource` with `project` attribute.

	Here we are including the javascript libraries (jquery openalyers etc.) from the lib folder of the [geodash-base](#geodash-base) (`project: geodash`) project specifying their version.

	The result of the built javascript is the sparc2.full.js file in the `.build/js/` folder

### sparc2 plugins 
<a id="geodash-plugin-main">geodash-plugin-main.git</a> - Implements wrapping angularjs directives and controllers for the map template side.

Here is the template this plugin inject 
```
<div
  class="row no-gutters geodash-main geodash-dashboard geodash-controller">
  <div
    id="geodash-map"
    class="container-fluid geodash-map geodash-controller"
    style="width: 100%;"
    geodash-map>
    <div data-geodash-controllers="GeoDashControllerOverlays" geodash-map-overlays></div> 
		-- here are inserted the map side buttons - then the GeoDashControllerOverlays is reponsible of their usge
		-- [geodash-plugin-overlays](https://github.com/geodashio/geodash-plugin-overlays) project loaded from 
		-- /geodash-base.git/config.yml
    <div data-geodash-controllers="GeoDashControllerMapNavbars" geodash-map-navbars></div>
		-- Bottom navigation navbar with months - in geodash-plugin-navbars project
    <div data-geodash-controllers="GeoDashControllerLegend" geodash-map-legend></div>
		-- On map legend with Population at Risk title - 
    <div id="map" data-geodash-controllers="GeoDashControllerMapMap" geodash-map-map></div>
		-- the GeoDashControllerMapMap is defined in geodash-plugin-map-map project
    <div id="geodash-popups" style="display:none;">
      <div id="popup"></div>
    </div>
  </div>
  <div id="geodash-modals"></div>
</div>
```

<a id="geodash-plugin-map-map">geodash-plugin-map-map.git</a> - It implements angularjs directives and controllers for the map.

This plugin has two branch, master and extent

For sparc2 use the extent branch if you have the ol3 resource set at version 3.17.1 in config.yml (if you want to use the master branch set the ol3 version at 3.20.1 otherwise the base map is not loaded and the browser javascritp console will shoe an error about `geodash.var.map.getView(...).getAnimating is not a function`)

Here is the template this plugin inject.
```
<div id="map" class="geodash-map-map"></div>
```

<a id="sparc2-plugin-sidebar">sparc2-plugin-sidebar.git</a> - It is responsible of the left sidebar it contains texts about graphs in the sidebar. For changes to the graphs see [sparc2-core.js.git](#sparc2-core-js)

<a id="sparc2-plugin-welcome">sparc2-plugin-welcome.git</a> - It is responsible of the modal window at project start

# geodash-base
Inside the lib folder of this project we have all the versions of libraries called in the build section of `config.yml` file of sparc2 project

The `config.yml` file of geodash-base is called like a resource in sparc2 `config.yml`.

## geodash-base plugins

- <a id="geodash-plugin-base">geodash-plugin-base</a> -

- <a id="geodash-plugin-filters">geodash-plugin-filters</a> - This plugin manage the filter tab functionalities inside the left sidebar

- <a id="geodash-plugin-handlers">geodash-plugin-handlers</a> - Here we have a `handlers` directory with javascript files related with [geodash-plugin-overlays](#geodash-plugin-overlays)

- <a id="geodash-plugin-legend">geodash-plugin-legend</a> - 

- <a id="geodash-plugin-navbars">geodash-plugin-navbars</a> - Where are defined directives controllers templates and styles for the months navbars.

- <a id="geodash-plugin-overlays">geodash-plugin-overlays</a> - It implements angularjs directives and controllers for the button on the map. (Zoom in, Zoom out, Reset Extent, Toggle legend, Submit an Issue, Toggle Full Screen, Print Map)

## geodash-base resources

- <a id="sparc2-core-js">sparc2-core.js.git</a> - Here we have the files to manage contents on the Charts Tab and to generate javascript about the c3.js graphs rendered in the left sidebar.

This module is in charge to create graph using d3 and c3 js libraries.

To build the module you'll need to install [browserify](http://browserify.org/), [uglify-js](https://www.npmjs.com/package/uglify-js), and [jshint](https://www.npmjs.com/package/jshint).  You should install globally with:

```
sudo npm install -g browserify
sudo npm install -g uglify-js
sudo npm install -g jshint
```

To run the build, which creates `dist/sparc2-core.js`, `dist/sparc2-core.min.js`, and the docs just run:

```
npm run build
```

To just build the distributable code (`dist/sparc2-core.js`, `dist/sparc2-core.min.js`), run:

```
npm run build:code
```

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

Install the geodash-framework-django from local downloaded repository (be sure you are in the sparc2 branch)
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

you can verify the status of the program launched from supervisord using the `supervisorctl status` command that have to return something like

```
sparc2:gunicorn                  RUNNING    pid 2780, uptime 0:16:12
sparc2:memcached                 RUNNING    pid 2728, uptime 0:16:20
```

### collectstatic
Change directory in `/home/vagrant/sparc2.git` where the `manage.py` file is located and run collectstatic to create Django static files
```
sudo /home/vagrant/.venvs/sparc2/bin/python manage.py collectstatic --noinput -i gulpfile.js -i package.json -i temp -i node_modules
```
`--noinput, --no-input`
Do NOT prompt the user for input of any kind.

`--ignore PATTERN, -i PATTERN`
Ignore files or directories matching this glob-style pattern. Use multiple times to ignore more.

### Run the application
Be sure have the sparc2 virtualenv active.

Start the application on all the network interfaces port 8000
```
(sparc2)vagrant@vagrant-ubuntu-trusty-64:~/sparc2.git$ python manage.py runserver 0.0.0.0:8000
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
