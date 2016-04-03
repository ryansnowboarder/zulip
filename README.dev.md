
Installing the Zulip Development environment
============================================

You will need a machine with at least 2GB of RAM available (see
https://github.com/zulip/zulip/issues/32 for a plan for how to
dramatically reduce this requirement). The recommended approach for
all platforms is to use Vagrant, which  will install the environment
inside a VM or container on any platform that supports Vagrant. 

Start by cloning the Zulip repositiory: 

non-Windows:
------------
`git clone https://github.com/zulip/zulip.git`

Windows:
--------
`git clone https://github.com/zulip/zulip.git  -c core.autocrlf=false` 

This is to remove Windows line endings, which cause weird errors.

Using Vagrant
=============

The best performing way to run the Zulip development environment is
using an LXC container on a Linux host, but we support other platforms
such as Mac/Windows via Virtualbox (but everything will be 2-3x slower).

Ubuntu 15.04 or newer
---------------------
* install and configure the LXC Vagrant provider directly using apt:
  ```
  sudo apt-get install vagrant lxc lxc-templates cgroup-lite redir
  vagrant plugin install vagrant-lxc
  ```

Ubuntu 14.04
------------
*  [download a newer  version of Vagrant][vagrant-dl], and then do the following:
  ```
  sudo apt-get install lxc lxc-templates cgroup-lite redir
  sudo dpkg -i vagrant*.deb # in directory where you downloaded vagrant
  vagrant plugin install vagrant-lxc
  ```

Other Linux (kernal above 3.12):
--------------------------------
* [follow the Vagrant LXC installation instructions][vagrant-lxc] to get 
  Vagrant with LXC for your platform.


OS X or older Linux (kernal at or below 3.12):
----------------------------------------------
* [download VirtualBox][vbox-dl], [download Vagrant][vagrant-dl], and install
  them both.

* Note: If you're on OS X and have VMWare, it should be possible to patch
  Vagrantfile to use the VMWare vagrant provider, which should perform
  much better than Virtualbox.  Patches to do this by default if
  VMWare is available are welcome!

Windows:
--------
* use Vagrant and Virtualbox/VMWare on Windows with Cygwin, similar to the Mac 
  setup. 

[vagrant-dl]: https://www.vagrantup.com/downloads.html
[vagrant-lxc]: https://github.com/fgrehm/vagrant-lxc
[vbox-dl]: https://www.virtualbox.org/wiki/Downloads

Universal Vagrant instructions:
-------------------------------
Once that's done, simply change to your zulip directory and run
`vagrant up` in your terminal to install the development server.  This
will take a long time on the first run and also fail to fully execute
on Windows (we're working on it). Vagrant needs to download the Ubuntu
Trusty base image, but later you can run `vagrant destroy` and then
`vagrant up` again to rebuild the environment and it will be much faster. 

Once that finishes, you can run the development server as follows:

```
vagrant ssh -- -L9991:localhost:9991
# Now inside the container
cd /srv/zulip
source /srv/zulip-venv/bin/activate
./tools/run-dev.py --interface=''
```

To get shell access to the virtual machine running the server to run
lint, management commands, etc., use `vagrant ssh`.

(A small note on tools/run-dev.py: the `--interface=''` option will
make the development server listen on all network interfaces.  While
this is correct for the Vagrant guest sitting behind a NAT, you
probably don't want to use that option when using run-dev.py in other
environments).

At this point you should [read about using the development
environment][using-dev].

[using-dev]: #using-the-development-environment


Using provision.py without Vagrant
----------------------------------

If you'd like to install a Zulip development environment on a server
that's already running Ubuntu 14.04 Trusty, you can do that by just
running:

```
sudo apt-get update
sudo apt-get install -y python-pbs
python /srv/zulip/provision.py

cd /srv/zulip
source /srv/zulip-venv/bin/activate
./tools/run-dev.py
```

Note that there is no supported uninstallation process without Vagrant
(with Vagrant, you can just do `vagrant destroy` to clean up the
development environment).

By hand
-------
If you really want to install everything by hand, the below
instructions should work.

Install the following non-Python dependencies:
 * libffi-dev — needed for some Python extensions
 * postgresql 9.1 or later — our database (client, server, headers)
 * nodejs 0.10 (and npm)
 * memcached (and headers)
 * rabbitmq-server
 * libldap2-dev
 * python-dev
 * redis-server — rate limiting
 * tsearch-extras — better text search
 * libfreetype6-dev — needed before you pip install Pillow to properly generate emoji PNGs

### On Debian or Ubuntu systems:

```
sudo apt-get install closure-compiler libfreetype6-dev libffi-dev \
    memcached rabbitmq-server libldap2-dev redis-server \
    postgresql-server-dev-all libmemcached-dev python-dev \
    hunspell-en-us nodejs nodejs-legacy npm git yui-compressor \
    puppet gettext

# If on 12.04 or wheezy:
sudo apt-get install postgresql-9.1
wget https://dl.dropboxusercontent.com/u/283158365/zuliposs/postgresql-9.1-tsearch-extras_0.1.2_amd64.deb
sudo dpkg -i postgresql-9.1-tsearch-extras_0.1.2_amd64.deb

# If on 14.04:
sudo apt-get install postgresql-9.3
wget https://dl.dropboxusercontent.com/u/283158365/zuliposs/postgresql-9.3-tsearch-extras_0.1.2_amd64.deb
sudo dpkg -i postgresql-9.3-tsearch-extras_0.1.2_amd64.deb

# If on 15.04 or jessie:
sudo apt-get install postgresql-9.4
wget https://dl.dropboxusercontent.com/u/283158365/zuliposs/postgresql-9.4-tsearch-extras_0.1_amd64.deb
sudo dpkg -i postgresql-9.4-tsearch-extras_0.1_amd64.deb
```

Now continue with the "All systems" instructions below.

### On Fedora 22 (experimental):

These instructions are experimental and may have bugs; patches
welcome!

```
sudo dnf install libffi-devel memcached rabbitmq-server \
    openldap-devel python-devel redis postgresql-server \
    postgresql-devel postgresql libmemcached-devel freetype-devel \
    nodejs npm yuicompressor closure-compiler gettext
```

Now continue with the Common to Fedora/CentOS instructions below.

### On CentOS 7 Core (experimental):

These instructions are experimental and may have bugs; patches
welcome!

```
# Add user zulip to the system (not necessary if you configured zulip
# as the administrator user during the install process of CentOS 7).
useradd zulip

# Create a password for zulip user
passwd zulip

# Allow zulip to sudo
visudo
# Add this line after line `root    ALL=(ALL)       ALL`
zulip   ALL=(ALL)       ALL

# Switch to zulip user
su zulip

# Enable EPEL 7 repo so we can install rabbitmq-server, redis and
# other dependencies
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# Install dependencies
sudo yum install libffi-devel memcached rabbitmq-server openldap-devel
    python-devel redis postgresql-server postgresql-devel postgresql \
    libmemcached-devel wget python-pip openssl-devel freetype-devel \
    libjpeg-turbo-devel zlib-devel nodejs yuicompressor \
    closure-compiler gettext

# We need these packages to compile tsearch-extras
sudo yum groupinstall "Development Tools"

# clone Zulip's git repo and cd into it
cd && git clone https://github.com/zulip/zulip && cd zulip/

## NEEDS TESTING: The next few DB setup items may not be required at all.
# Initialize the postgres db
sudo postgresql-setup initdb

# Edit the postgres settings:
sudo vi /var/lib/pgsql/data/pg_hba.conf

# Change these lines:
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident
# to this:
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

Now continue with the Common to Fedora/CentOS instructions below.

### On OpenBSD 5.8 (experimental):

These instructions are experimental and may have bugs; patches
welcome!

```
doas pkg_add sudo bash gcc postgresql-server redis rabbitmq \
    memcached node libmemcached py-Pillow py-cryptography py-cffi

# Get tsearch_extras and build it (using a modified version which
# aliases int4 on OpenBSD):
git clone https://github.com/blablacio/tsearch_extras
cd tsearch_extras
gmake && sudo gmake install

# Point environment to custom include locations and use newer GCC
# (needed for Node modules):
export CFLAGS="-I/usr/local/include -I/usr/local/include/sasl"
export CXX=eg++

# Create tsearch_data directory:
sudo mkdir /usr/local/share/postgresql/tsearch_data


# Hack around missing dictionary files -- need to fix this to get the
# proper dictionaries from what in debian is the hunspell-en-us
# package.
sudo touch /usr/local/share/postgresql/tsearch_data/english.stop
sudo touch /usr/local/share/postgresql/tsearch_data/en_us.dict
sudo touch /usr/local/share/postgresql/tsearch_data/en_us.affix
```

Now continue with the All Systems instructions below.

### Common to Fedora/CentOS instructions

```
# Build and install postgres tsearch-extras module
wget https://launchpad.net/~tabbott/+archive/ubuntu/zulip/+files/tsearch-extras_0.1.3.tar.gz
tar xvzf tsearch-extras_0.1.3.tar.gz
cd ts2
make
sudo make install

# Hack around missing dictionary files -- need to fix this to get the
# proper dictionaries from what in debian is the hunspell-en-us
# package.
sudo touch /usr/share/pgsql/tsearch_data/english.stop
sudo touch /usr/share/pgsql/tsearch_data/en_us.dict
sudo touch /usr/share/pgsql/tsearch_data/en_us.affix

# Edit the postgres settings:
sudo vi /var/lib/pgsql/data/pg_hba.conf

# Add this line before the first uncommented line to enable password
# auth:
host    all             all             127.0.0.1/32            md5

# Start the services
sudo systemctl start redis memcached rabbitmq-server postgresql

# Enable automatic service startup after the system startup
sudo systemctl enable redis rabbitmq-server memcached postgresql
```

Finally continue with the All Systems instructions below.

### All Systems:

```
pip install -r requirements.txt
npm install
./tools/download-zxcvbn
./tools/emoji_dump/build_emoji
./scripts/setup/generate_secrets.py -d
if [ $(uname) = "OpenBSD" ]; then sudo cp ./puppet/zulip/files/postgresql/zulip_english.stop /var/postgresql/tsearch_data/; else sudo cp ./puppet/zulip/files/postgresql/zulip_english.stop /usr/share/postgresql/9.3/tsearch_data/; fi
./scripts/setup/configure-rabbitmq
./tools/postgres-init-dev-db
./tools/do-destroy-rebuild-database
./tools/postgres-init-test-db
./tools/do-destroy-rebuild-test-database
```

To start the development server:

```
./tools/run-dev.py
```

… and visit [http://localhost:9991/](http://localhost:9991/).

Using Docker
-------------

You can also use Docker to develop, first you need to install Docker
in your development machine following the
[instructions][docker-install].
Some other interesting links for somebody new in Docker are:

* [Get Started](https://docs.docker.com/linux/started/)
* [Understand the architecture](https://docs.docker.com/engine/introduction/understanding-docker/)
* [Docker run reference](https://docs.docker.com/engine/reference/run/)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

[docker-install]: https://docs.docker.com/engine/installation/

Then you should create the Docker image based on Ubuntu Linux, first
go to the directory with the Zulip source code:

```
docker build -t user/zulipdev .
```

Now you're going to install Zulip dependencies in the image:

```
docker run -itv $(pwd):/srv/zulip -p 80:9991 user/zulipdev /bin/bash
$ /usr/bin/python /srv/zulip/provision.py --docker
docker ps -af ancestor=user/zulipdev
docker commit -m "Zulip installed" <container id> user/zulipdev:v2
```

Finally you can run the docker server with:

```
docker run -itv $(pwd):/srv/zulip -p 80:9991 user/zulipdev:v2 \
    /srv/zulip/scripts/start-dockers
```

If you want to connect to the Docker instance to build a release
tarball you can use:

```
docker ps
docker exec -it <container id> /bin/bash
$ source /home/zulip/.bash_profile
$ <Your commands>
$ exit
```

To stop the server use:
```
docker ps
docker kill <container id>
```

If you want to run all the tests you need to start the servers first,
you can do it with:

```
docker run -itv $(pwd):/srv/zulip user/zulipdev:v2 /bin/bash
$ scripts/test-all-docker
```

You can modify the source code in your development machine and review
the results in your browser.


Using the Development Environment
=================================

Once the development environment is running, you can visit
<http://localhost:9991/> in your browser.  By default, the development
server homepage just shows a list of the users that exist on the
server and you can login as any of them by just clicking on a user.
This setup saves time for the common case where you want to test
something other than the login process; to test the login process
you'll want to change AUTHENTICATION_BACKENDS in the not-PRODUCTION
case of `zproject/settings.py` from zproject.backends.DevAuthBackend
to use the auth method(s) you'd like to test.

While developing, it's helpful to watch the `run-dev.py` console
output, which will show any errors your Zulip development server
encounters.

When you make a change, here's a guide for what you need to do in
order to see your change take effect in Development:

* If you change Javascript or CSS, you'll just need to reload the
browser window to see changes take effect.

* If you change Python code used by the the main Django/Tornado server
processes, these services are run on top of Django's [manage.py
runserver][django-runserver] which will automatically restart the
Zulip Django and Tornado servers whenever you save changes to Python
code.  You can watch this happen in the `run-dev.py` console to make
sure the backend has reloaded.

* The Python queue workers don't automatically restart when you save
changes (or when they stop running), so you will want to ctrl-C and
then restart `run-dev.py` manually if you are testing changes to the
queue workers or if a queue worker has crashed.

* If you change the database schema, you'll need to use the standard
Django migrations process to create and then run your migrations; see
the [new feature tutorial][new-feature-tutorial] for an example.
Additionally you should check out the [detailed testing
docs][testing-docs] for how to run the tests properly after doing a
migration.

(In production, everything runs under supervisord and thus will
restart if it crashes, and `upgrade-zulip` will take care of running
migrations and then cleanly restaring the server for you).

[django-runserver]: https://docs.djangoproject.com/en/1.8/ref/django-admin/#runserver-port-or-address-port
[new-feature-tutorial]: http://zulip.readthedocs.org/en/latest/new-feature-tutorial.html
[testing-docs]: http://zulip.readthedocs.org/en/latest/testing.html

Running the test suite
======================

For more details, especially on how to write tests, check out the
[detailed testing docs][tdocs].

[tdocs]: http://zulip.readthedocs.org/en/latest/testing.html

To run all the tests, do this:
```
./tools/test-all
```

For the Vagrant environment, you'll want to first enter the
environment:
```
vagrant ssh
source /srv/zulip-venv/bin/activate
cd /srv/zulip
```

This runs the linter (`tools/lint-all`) plus all of our test suites;
they can all be run separately (just read `tools/test-all` to see
them).  You can also run individual tests which can save you a lot of
time debugging a test failure, e.g.:

```
./tools/lint-all # Runs all the linters in parallel
./tools/test-backend zerver.test_bugdown.BugdownTest.test_inline_youtube
./tools/test-js-with-casper 10-navigation.js
./tools/test-js-with-node # Runs all node tests but is very fast
```

The above setup instructions include the first-time setup of test
databases, but you may need to rebuild the test database occasionally
if you're working on new database migrations.  To do this, run:

```
./tools/postgres-init-test-db
./tools/do-destroy-rebuild-test-database
```

Possible testing issues
=======================

- When running the test suite, if you get an error like this:

  ```
      sqlalchemy.exc.ProgrammingError: (ProgrammingError) function ts_match_locs_array(unknown, text, tsquery) does not   exist
      LINE 2: ...ECT message_id, flags, subject, rendered_content, ts_match_l...
                                                                   ^
  ```

  … then you need to install tsearch-extras, described
  above. Afterwards, re-run the `init*-db` and the
  `do-destroy-rebuild*-database` scripts.

- When building the development environment using Vagrant and the LXC
  provider, if you encounter permissions errors, you may need to
  `chown -R 1000:$(whoami) /path/to/zulip` on the host before running
  `vagrant up` in order to ensure that the synced directory has the
  correct owner during provision. This issue will arise if you run `id
  username` on the host where `username` is the user running Vagrant
  and the output is anything but 1000.
  This seems to be caused by Vagrant behavior; for more information,
  see [the vagrant-lxc FAQ entry about shared folder permissions
  ][lxc-sf].

[lxc-sf]: https://github.com/fgrehm/vagrant-lxc/wiki/FAQ#help-my-shared-folders-have-the-wrong-owner)
