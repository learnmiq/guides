## Developer Setup

### Install System Packages

#### CentOS 7

* Install CentOS 7 mimimum.

  * update and reboot.
  ```
  [root@centos7 ~]# yum update -y && reboot
  <snipped>
    Replaced:
    NetworkManager.x86_64 1:1.4.0-14.el7_3
    grub2.x86_64 1:2.02-0.44.el7.centos
    grub2-tools.x86_64 1:2.02-0.44.el7.centos
    pygobject3-base.x86_64 0:3.14.0-3.el7
    rdma.noarch 0:7.3_4.7_rc2-5.el7
  
  Complete!
  [root@centos7 ~]#
  ```  

  * OS info
  ```
  [root@centos7 ~]# uname -a ;cat /etc/redhat-release
  Linux centos7 3.10.0-514.6.2.el7.x86_64 #1 SMP Thu Feb 23 03:04:39 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
  CentOS Linux release 7.4.1708 (Core)
  [root@centos7 ~]#
  ```  

* Install Packages

  ```
  sudo yum -y group install "Development Tools"                  # For unf Gem and noi4r Gem
  sudo yum -y install git-all                                    # Git and components
  sudo yum -y install memcached                                  # Memcached for the session store
  sudo yum -y install bzip2 libffi-devel readline-devel          # For rbenv install 2.3.1 (might not be needed with other Ruby setups)
  sudo yum -y install libxml2-devel libxslt-devel patch          # For Nokogiri Gem
  sudo yum -y install sqlite-devel                               # For sqlite3 Gem
  sudo yum -y install nodejs                                     # For ExecJS Gem, bower, npm, yarn, webpack.. - needs at least 6.0.0
  sudo yum -y install libcurl-devel                              # For Curb
  rpm -q --whatprovides npm || sudo yum -y install npm           # For CentOS 7, Fedora 23 and older
  sudo yum -y install openssl-devel                              # For rubygems
  sudo yum -y install cmake                                      # For rugged Gem
  sudo yum -y install openscap                                   # Optional, for openscap Gem for container SSA
  rpm -q --whatprovides ruby || sudo yum -y install ruby
  ```

* Enable Memcached

  ```
  sudo systemctl start  memcached &&   sudo systemctl enable memcached
  ```

* Configure PostgreSQL
  * Required PostgreSQL version is 9.4, 9.5
    * See [here](developer_setup/postgresql_software_collection.md) how to install
      it in Linux distributions like CentOS 7, using _SoftwareCollections.org_.
    * Or follow the directions [here](https://www.postgresql.org/download/linux/redhat/#yum)
      to install it from the PostgreSQL Global Development Group repositories.

  ```

  sudo postgresql-setup initdb
  sudo grep -q '^local\s' /var/opt/rh/rh-postgresql94/lib/pgsql/data/pg_hba.conf || echo "local all all trust" | sudo tee -a /var/opt/rh/rh-postgresql94/lib/pgsql/data/pg_hba.conf
  sudo sed -i.bak 's/\(^local\s*\w*\s*\w*\s*\)\(peer$\)/\1trust/' /var/opt/rh/rh-postgresql94/lib/pgsql/data/pg_hba.conf
  sudo systemctl enable postgresql
  sudo systemctl start postgresql
  sudo -u postgres psql -c "CREATE ROLE root SUPERUSER LOGIN PASSWORD 'smartvm'"
  # This command can return with a "could not change directory to" error, but you can ignore it
  ```

* check the status

we see it is only avaible to from local host connection.

```
[me@centos7 manageiq]$ sudo netstat -antulp |grep 5432
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      23867/postgres
[me@centos7 manageiq]$
```

* Make postgres acept connection from all hosts.

```
cd /var/opt/rh/rh-postgresql94/lib/pgsql/data/

[root@centos7 data]# egrep -v '^#|^$'  pg_hba.conf                                          |
host    all             all             0.0.0.0/0               md5 
[root@centos7 data]#         

```
* Make postgres acept connection from all hosts

```
cd /var/opt/rh/rh-postgresql94/lib/pgsql/data/
egrep -v '^#|^$'  postgresql.conf | egrep '^listen_address'
# we should see folloing
listen_addresses = '*'    # what IP address(es) to listen on; 
```

* Use pgadmin4 to get a GUI with our PG db.

### Install Ruby and Bundler

* CentOS come with older version of ruby
  ```
  [root@centos7 ~]# rpm -q --whatprovides ruby
  ruby-2.0.0.648-33.el7_4.x86_64
  [root@centos7 ~]#
  ```

* Remove older version of ruby
  ```
  [root@centos7 ~]# yum remove -y ruby-\* rubygem* && rpm -qa |grep ruby
  Loaded plugins: fastestmirror
  No Match for argument: ruby-*
  No Match for argument: rubygem*
  No Packages marked for removal
  [root@centos7 ~]#
  ```
* Install a newer version ruby from SCL

```
yum --enablerepo=centos-sclo-rh -y install rh-ruby24\*
scl enable rh-ruby24 bash
[root@centos7 ~]# ruby --version
ruby 2.4.0p0 (2016-12-24 revision 57164) [x86_64-linux]
[root@centos7 ~]#
[root@dlp ~]# vi /etc/profile.d/rh-ruby24.sh
# create new
 #!/bin/bash

source /opt/rh/rh-ruby24/enable
export X_SCLS="`scl enable rh-ruby24 'echo $X_SCLS'`"

```

* build and install pg gem with ruby 2.4 and /opt/rh/rh-postgresql94 from SCL

```
gem install pg -- --with-pg-config=/opt/rh/rh-postgresql94/root/usr/bin/pg_config
```
* Use a Ruby version manager (choose one)
  * [chruby](https://github.com/postmodern/chruby) and [ruby-install](https://github.com/postmodern/ruby-install)
  * [rvm](http://rvm.io/)
  * [rbenv](https://github.com/rbenv/rbenv)
* Required Minimum Ruby version is 2.3.1+
* Required Minimum Bundler version is 1.8.7+

### Setup Git and Github

* The most reliable authentication mechanism for git uses SSH keys.
  * [SSH Key Setup](https://help.github.com/articles/generating-ssh-keys) Set up a SSH keypair for authentication.  Note: If you enter a passphrase, you will be prompted every time you authenticate with it.
* Github account setup [Account Settings](https://github.com/settings).
  * [Profile](https://github.com/settings/profile): Fill in your personal information, such as your name.
  * [Profile](https://github.com/settings/profile): Optionally set up an avatar at gravatar.com.  When you set up your gravatar, be sure to have an entry for the addresses you plan to use with git / Github.
  * [Emails](https://github.com/settings/emails): Enter your e-mail address and verify it, click the Verify button and follow the instructions.
  * [Notification Center](https://github.com/settings/notifications) / Notification Email / Custom Routing: Change the email address associated with ManageIQ if desired.
  * If you are a member of the ManageIQ organization:
    * Go to [the Members page](https://github.com/ManageIQ?tab=members).
      * Verify you are listed.
      * Optionally click Publicize Membership.

* Fork ManageIQ/manageiq:

  * Go to [ManageIQ/manageiq](https://github.com/ManageIQ/manageiq)
  * Click the Fork button and choose "Fork to \<yourname\>"

* Git configuration and default settings.

  ```
  git config --global user.name "Joe Smith"
  git config --global user.email joe.smith@example.com
  git config --global --bool pull.rebase true
  git config --global push.default simple
  ```

  If you need to use git with other email addresses, you can set the local user.email from within the clone using:

  ```
  git config user.name "Joe Smith"
  git config user.email joe.smith@example.com
  ```

### Clone the Code using git protocol if you are in manageiq team

```
git clone git@github.com:JoeSmith/manageiq.git # Use "-o my_fork" if you don't want the remote to be named origin
cd manageiq
git remote add upstream git@github.com:ManageIQ/manageiq.git
git fetch upstream
```


### Clone the Code in https readonly mode

```
git clone git@github.com:LearningManageIQ/manageiq.git # Use "-o my_fork" if you don't want the remote to be named origin
cd  cd manageiq
git remote add upstream https://github.com/ManageIQ/manageiq.git
git fetch upstream
```

You can add other remotes at any time to collaborate with others by running:

```
git remote add other_user git@github.com:OtherUser/manageiq.git
git fetch other_user
```

### Javascripty things

  Make sure your node version is at least 6.0.0. If not, you can use [nvm](https://github.com/creationix/nvm) to install a node version locally (similar to `rbenv`).

  Note: you may need to run the `npm install -g` commands using sudo if you get permission errors,
  but if your node environment is up to date you should be able to install without sudo.
  If you do get these errors and you don't want to use sudo,
  [you can configure npm to install packages globally for a given user](https://github.com/sindresorhus/guides/blob/master/npm-global-without-sudo.md).

* make sure your account can sudo without password
  ```
  /etc/group
  /etc/sudoer
  adm 
  ```
* Install the [_Bower_](https://bower.io/) package manager

  ```
  sudo npm install -g bower
  ```

* Install the [_Yarn_](https://yarnpkg.com/en/) package manager

  Follow [official instructions](https://yarnpkg.com/lang/en/docs/install/#linux-tab) or

  ```
  sudo npm install -g yarn
  ```

* Install the _Gulp_(https://gulpjs.com/) and [_Webpack_](https://webpack.github.io/) build system

  ```
  sudo npm install -g gulp-cli &&  sudo npm install -g webpack
  ```

### Get the Rails environment up and running

```
bin/setup                  # Installs dependencies, config, prepares database, etc
gem install pg -v '0.18.4' -- --with-pg-config=/opt/rh/rh-postgresql94/root/usr/bin/pg_config
bundle install 
bundle exec rake evm:start # Starts the ManageIQ EVM Application in the background
```

* You can now access the application at `http://localhost:3000`. The default username is `admin` with password `smartvm`.
* There is also a minimal mode available to start the application with fewer services and workers for faster startup or
targeted end-user testing. See the [minimal mode guide](developer_setup/minimal_mode.md) for details.
* As an alternative to minimal mode, individual workers can also be started using [Foreman](https://ddollar.github.io/foreman/)
  * See the [Foreman guide](developer_setup/foreman.md) for details.
* To run the test suites, see [the guide on that topic](developer_setup/running_test_suites.md).

### Update dependencies and migrate db

* You can update ruby and javascript dependencies as well as run migrations using one command

```
bin/update                # Updates dependencies using bundler and bower, runs migrations, prepares test db.
```

For provider, UI or other plugin development, see [the guide on that topic](developer_setup/plugins.md).


# Some troubleshooting notes

* First login fails

Make sure you have memcached running. If not stop the server with `bundle exec rake evm:stop`,
start memcached and retry.

* `bin/setup fails` while trying to load the gem 'sprockets/es6'

If this happens check the log for
`ExecJS::RuntimeUnavailable: Could not find a JavaScript runtime` a few lines down.
When this message is present, then the you need to install `node.js` and re-try


* `bin/setup` fails to install the 'sys-proctable' gem, or installs the wrong version.

If this happens it may be a Bundler issue. Try running `bundle config specific_platform true`
and re-try.

* Can't install any gems during bundle install

```
# install all dependencies to a local path
bundle install --path vendor/bundle
```

* Can't install `sys-proctable` on a Mac - a package missing even after bundle install succeeded

```
bundle config specific_platform true
bundle install
```

* FATAL:  database "vmdb_development" does not exist

```
me@centos7 manageiq]$ bundle exec rake evm:start
** Using session_store: ActionDispatch::Session::MemCacheStore
Starting EVM...
rake aborted!
ActiveRecord::NoDatabaseError: FATAL:  database "vmdb_development" does not exist
<snipped>
me@centos7 manageiq]$ 

```
