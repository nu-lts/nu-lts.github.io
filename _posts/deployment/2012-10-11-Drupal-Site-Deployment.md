---
layout: post
category : drush
tags : [drupal, drush, git, mamp, developer tools]
author: Steven Bassett
tagline: Less time, more beer.
---
{% include JB/setup %}


This is a short article on how to use a local Linux, Apache, MySQL and PHP environment for website development and deployment to the staging and/or production server. I please leave comments if you have any additional suggestions.

##Installing the required software locally.
You will need to make sure that that you have the software installed and configured before you can try moving a site locallly.

### Download and install [MAMP or MAMP Pro](http://www.mamp.info/en/index.html).
I like to configure my development enviroment to point to the `~/Sites/` directory to serve the sites.
###Installing git version control
Download and install the latest version of [git](http://git-scm.com/).
Follow the instructions.
If you’re looking for some enticing reading, check out [Pro Git](http://git-scm.com/book) for detailed infomation about git version control.
###Installing Drush

Please read the following Administration Guide from Drupal.org for the most up to date information on installing drush [Specific instructions for installing Drush on different platforms](https://drupal.org/node/1791676).

####Drush with MAMP Pro
*Please read this article for the most up to date information* 
[Original Article](http://brianfisher.name/content/drupal-development-environment-os-x-mamp-pro-eclipse-xdebug-and-drush)
These instructions are somewhat specific to MAMP PRO 2.1.1. [Download](http://www.mamp.info/en/downloads/index.html), install, and launch MAMP Pro.

Under the PHP tab, select php 5.x from version pulldown. Click
"Apply". Click "Start".

Add the MAMP executables to your path.

	chmod +x /Applications/MAMP/bin/php/php5.3.14/bin/*
	echo "export PATH=/Applications/MAMP/bin/php/php5.3.14/bin:/Applications/MAMP/Library/bin:\$PATH" >> ~/.profile
	. ~/.profile


Add a symlink to the MAMP Mysql socket to allow drush to connect

	sudo mkdir /var/mysql
	sudo ln -s /Applications/MAMP/tmp/mysql/mysql.sock /var/mysql/mysql.sock


##### Performace
Please change the default php and mysql settings so that there are enough resources for both drush and drupal to run.

###### MYSQL

You may get "MySql server has gone away" errors. To fix, in MAMP, edit the
mysql configuration by going to file->edit template->mysql. Change
`max_allowed_packet` to `max_allowed_packet =  16M`

######PHP

Increase both the MAMP PHP memory limit. In MAMP, edit the php configuration
file->edit template->php->php5.3
change `memory_limit` to `memory_limit =  256M`


Increase the CLI PHP memory limit. To figure out where your CLI php.ini file
is, use: `$ which php` then `php -i | grep  php.in`. Then edit the file and change `memory_limit` to `memory_limit =  -1`.

### Cloning production site to your local environment. 
#### Clone the Git repository.

Open Terminal. Use git to clone the repository for the project you’re working on from our [Bitbucket Account](https://bitbucket.org/nu_lts). For example use,  `git clone git@bitbucket.org:nu_lts/dmc.git ~/sites/dmc.neu.edu/`, _clone from remote to this directory_. Note that this will only bring down the code that is being tracked by the git repository. It will not bring down any files or directories not tracked by Git.


###Setting Up Drush Site Aliases
Go to your root apache htdocs folder, in my case `cd ~/Sites`.
Set up a file structure such as: `~/Sites/library.neu.edu/` to act as the local root for your Drupal directory.
This would let you access your site from[localhost:8888/library.neu.edu/](localhost:8888/library.neu.edu/) in a default MAMP environment.

##### Where drush looks for aliases.

I have used the following location by using: `cp /path/to/drush/examples/example.aliases.drushrc.php ~/.drush/aliases.drushrc.php` to move the example file with documentation to my home directory that drush will look at to find aliases.

Defualt Locations
* /etc/drush
* $HOME/.drush
* The sites/all/drush folder for the current Drupal site

Edit the `aliases.drushrc.php` file by configuring the production, staging and local environment you would like to work with eg:
	
	$aliases['dmc.local'] = array(
	'uri' => 'localhost:8888/dmc.neu.edu',
	'root' => '/Users/steven/Sites/dmc.neu.edu', // This doesn't work with ~/Sites/dev.local
	'db_url' => 'mysql://root:root@localhost:8889/dmc',
	'path-aliases' => array(
	  '%files' => 'sites/defualt/files',
	  '%dump' => '/Users/steven/tmp/drush/sql-sync-dev-local.sql', // Arbitrary location for temp files
	  ),
	);
	$aliases['dmc.prod'] = array(
	'uri' => 'defualt',
	'root' => '/var/www/html',
	'databases' =>
	  array (
	   'default' =>
	   array (
	   'default' =>
	   array (
	   'database' => 'dmc',
	   'username' => 'dmc1',
	   'password' => ‘****’,
	   'host' => 'localhost',
	   'port' => '',
	   'driver' => 'mysql',
	   'prefix' => '',
	   ),
	   ),
	  ),
	'remote-user' => 'sbassett',
	'remote-host' => '155.33.22.226',
	'path-aliases' => array(
	  '%drush' => '/usr/local/share/drush',
	  '%site' => 'sites/default',
	  '%files' => 'sites/defualt/files',
	  '%dump' => 'tmp', // Arbitrary location for temp files
	  ),
	);

###Finding the site information for the aliases

Tip from [Using drush to synchronize and deploy sites](https://drupal.org/node/670460). Creating the site-alias config array is tedious by hand. If you
have a working site. Change into the site dir of a working site and run `drush site-alias --with-db --show-passwords --with-optional @self`
I often go `drush site-alias --with-db --show-password --with-optional @self >
/etc/drush/mysqit.alias.drushrc.php`
and then **importantly** edit the resulting new files and  A: add a `<?php` tag to the top!  B: relabel it from @self to your preferred nickname - which
must match the filename you used. Those extra connection details are required for remote aliases. If you want, you can also split out the component parts of an alias and use
inheritance to construct the peer alias.

###SSH Key Pairs
**Required for Drush to work with remote servers**

Go to your .ssh directory: `$ cd ~/.ssh`
Check for keys: 

	$ ls
	id_rsa		id_rsa.pub

If there are no keys like that above try creating the keys using the directions from github:

	$ ssh-keygen -t rsa -C "your_email@youremail.com"
	# Creates a new ssh key using the provided email
	# Generating public/private rsa key pair.
	# Enter file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]

Do not enter a passphrase:

	Enter passphrase (empty for no passphrase): [Press enter]
	Enter same passphrase again: [Press enter]

Should return:

	Your identification has been saved in /Users/you/.ssh/id_rsa.
	Your public key has been saved in /Users/you/.ssh/id_rsa.pub.
	The key fingerprint is:
	01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@youremail.com

Make sure to create the `touch ~/.ssh/authorized_keys` file. Try to create the permissions to be like the following for your environment:

	$ cd ~/.ssh
	$ ls -al
	drwx------ 2 user user 4096 Oct 10 12:31 .
	drwx------ 3 user user 4096 Oct 10 12:51 ..
	-rw------- 1 user user  808 Oct 10 12:50 authorized_keys
	-rw------- 1 user user 1675 Oct 10 12:31 identity
	-rw------- 1 user user 1675 Oct 10 12:16 id_rsa
	-rw-r--r-- 1 user user  398 Oct 10 12:16 id_rsa.pub

SSH into the remote server  `$ ssh user@example.neu.edu`, then create a keypair for the remote server using the same method above. **(Don’t forget to check the permissions!)**
When you have completed the steps on both your local and remote machines, then you can add your key to the remote server. From your **local terminal**:
	$ login=username@hostname
	$ ssh $login "echo '`cat ~/.ssh/id_rsa.pub`' >> ~/.ssh/authorized_keys"

To debug what might be happening you can run:
	$ ssh -v user@server.edu
***Need more assistance making a bootstrap way of creating key pairs.***

###Using Drush commands on remote servers
To use drush commands on a remote server you can simply use the alias for. For example I can look and see the basic site status for the DMC website.  `$ drush @dmc.prod status` should return the basic overview of the remote site.           


### Using rsync to fetch files from the remote server.

Go fetch the files from our production server to local server: `drush rsync @dmc.prod:%files/ @dmc.local:%files/
`
### Using drush to run a site install

Before running the command please make sure to look at `drush site-install —help` for instructions.
To install the local site into mamp you can run the following command:

	$ drush @dmc.local use
	$ drush si

If the aliases are configured correctly, you can go the the local url eg localhost:8888/dmc.neu.edu/ and it should install a basic site. Nothing special.

### Using drush to sync the databases
*Note: you will probably want to clear the cache on both sites before the transfer* `$ drush cc all`.
Use the drush `sql-sync` command to send from source to target. For example to create your local enviromnet, you will want to pull down the sql database using: `$ drush sql-sync @dmc.prod @dmc.local`

### Notes for trouble shooting White-Screen-Of-Death (WSOD)

Try turning off clean urls: `$ drush vset --exact clean_url 0`

Clear the cache `$ drush cc all`

Look at the settings.php file in the sites/default/ directory.

Look at the .htaccess file.

Refer to the php.ini files to make sure that MAMP is configured to give enough memory for both drush and drupal to run.

Check your mysql and php error logs, with MAMP they are located  `/Applications/MAMP/logs/`

##Articles##
* [Using drush to synchronize and deploy sites](https://drupal.org/node/670460)
* [A Drupal Development Environment on OS X with MAMP Pro, Eclipse, xDebug, and Drush](http://brianfisher.name/content/drupal-development-environment-os-x-mamp-pro-eclipse-xdebug-and-drush)
* [getting drush to work on a MAMP setup](https://drupal.org/node/601604)
* [Drush Official Site](http://drush.ws/)