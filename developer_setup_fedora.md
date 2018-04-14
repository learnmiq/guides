## Developer Setup

### Install System Packages

#### Fedora / CentOS 7

* CentOS only - use yum instead of dnf.

* Install Packages

  ```bash
  sudo dnf -y group install "C Development Tools and Libraries"  # For unf Gem and noi4r Gem
  sudo dnf -y install git-all                                    # Git and componentso
  sudo dnf -y install memcached                                  # Memcached for the session store
  sudo dnf -y install postgresql-devel postgresql-server         # PostgreSQL Database server and to build 'pg' Gem
  sudo dnf -y install bzip2 libffi-devel readline-devel          # For rbenv install 2.3.1 (might not be needed with other Ruby setups)
  sudo dnf -y install libxml2-devel libxslt-devel patch          # For Nokogiri Gem
  sudo dnf -y install sqlite-devel                               # For sqlite3 Gem
  sudo dnf -y install nodejs                                     # For ExecJS Gem, bower, npm, yarn, webpack.. - needs at least 6.0.0
  sudo dnf -y install libcurl-devel                              # For Curb
  rpm -q --whatprovides npm || sudo dnf -y install npm           # For CentOS 7, Fedora 23 and older
  sudo dnf -y install openssl-devel                              # For rubygems
  sudo dnf -y install cmake                                      # For rugged Gem
  sudo dnf -y install openscap                                   # Optional, for openscap Gem for container SSA
  ```

* Enable Memcached

  ```bash
  sudo systemctl enable memcached
  sudo systemctl start  memcached
  ```

* Configure PostgreSQL
  * Required PostgreSQL version is 9.4, 9.5
    * See [here](developer_setup/postgresql_software_collection.md) how to install
      it in Linux distributions like CentOS 7, using _SoftwareCollections.org_.
    * Or follow the directions [here](https://www.postgresql.org/download/linux/redhat/#yum)
      to install it from the PostgreSQL Global Development Group repositories.

  ```bash
  sudo postgresql-setup initdb
  sudo grep -q '^local\s' /var/lib/pgsql/data/pg_hba.conf || echo "local all all trust" | sudo tee -a /var/lib/pgsql/data/pg_hba.conf
  sudo sed -i.bak 's/\(^local\s*\w*\s*\w*\s*\)\(peer$\)/\1trust/' /var/lib/pgsql/data/pg_hba.conf
  sudo systemctl enable postgresql
  sudo systemctl start postgresql
  sudo -u postgres psql -c "CREATE ROLE root SUPERUSER LOGIN PASSWORD 'smartvm'"
  # This command can return with a "could not change directory to" error, but you can ignore it
  ```

### Install Ruby and Bundler

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

  ```bash
  git config --global user.name "Joe Smith"
  git config --global user.email joe.smith@example.com
  git config --global --bool pull.rebase true
  git config --global push.default simple
  ```

  If you need to use git with other email addresses, you can set the local user.email from within the clone using:

  ```bash
  git config user.name "Joe Smith"
  git config user.email joe.smith@example.com
  ```

### Clone the Code

```bash
git clone git@github.com:JoeSmith/manageiq.git # Use "-o my_fork" if you don't want the remote to be named origin
cd manageiq
git remote add upstream git@github.com:ManageIQ/manageiq.git
git fetch upstream
```

You can add other remotes at any time to collaborate with others by running:

```bash
git remote add other_user git@github.com:OtherUser/manageiq.git
git fetch other_user
```

### Javascripty things

  Make sure your node version is at least 6.0.0. If not, you can use [nvm](https://github.com/creationix/nvm) to install a node version locally (similar to `rbenv`).

  Note: you may need to run the `npm install -g` commands using sudo if you get permission errors,
  but if your node environment is up to date you should be able to install without sudo.
  If you do get these errors and you don't want to use sudo,
  [you can configure npm to install packages globally for a given user](https://github.com/sindresorhus/guides/blob/master/npm-global-without-sudo.md).

* Install the _Bower_ package manager

  ```bash
  npm install -g bower
  ```

* Install the _Yarn_ package manager

  Follow [official instructions](https://yarnpkg.com/lang/en/docs/install/#linux-tab) or

  ```bash
  npm install -g yarn
  ```

* Install the _Gulp_ and _Webpack_ build system

  ```bash
  npm install -g gulp-cli
  npm install -g webpack
  ```

### Get the Rails environment up and running

```bash
bin/setup                  # Installs dependencies, config, prepares database, etc
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

```bash
bin/update                # Updates dependencies using bundler and bower, runs migrations, prepares test db.
```

For provider, UI or other plugin development, see [the guide on that topic](developer_setup/plugins.md).

#### Some troubleshooting notes

* First login fails

Make sure you have memcached running. If not stop the server with `bundle exec rake evm:stop`,
start memcached and retry.

* `bin/setup fails` while trying to load the gem 'sprockets/es6'

If this happens check the log for
`ExecJS::RuntimeUnavailable: Could not find a JavaScript runtime` a few lines down.
When this message is present, then the you need to install `node.js` and re-try

* `bin/setup` fails while trying to install the 'nokogiri' gem

If this happens, you may be missing developer tools in your OS X. Try to install them with
`xcode-select --install` and re-try.

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

