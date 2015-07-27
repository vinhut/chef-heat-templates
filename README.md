# chef OpenStack Heat Orchestration Templates (HOT)

There are two templates here to help you create your chef server infrastructure.

## single-stack chef server

The first is a single-stack instance which is designed to build out a server, router,
and it's own network with a floating IP.

```
source openrc
SEC_ID=$(nova secgroup-list | awk '/ default / { print $2 }')
NET_ID=$(nova net-list | awk '/ ext-net / { print $2 }')
CHEFSERVER_CORE=chef-server-core_12.1.2-1_amd64.deb
CHEFSERVER_USERNAME=admin
CHEFSERVER_FIRSTNAME=Admin
CHEFSERVER_LASTNAME=Jacob
CHEFSERVER_EMAIL=admin@fake-email.org
CHEFSERVER_PASSWORD=P@$$w0rd
CHEFSERVER_PEM=/tmp/admin.pem
CHEFSERVER_SHORTNAME=defaultinc
CHEFSERVER_FULLNAME="Default Inc."
IMAGE_ID=ubuntu-trusty
KEY_NAME=admin
heat stack-create -f single_chef_server-HOT.yml
                  -P public_net=$NET_ID \
                  -P secgroup_id=$SEC_ID \
                  -P chefserver-core=$CHEFSERVER_CORE \
                  -P chefserver-username=$CHEFSERVER_USERNAME \
                  -P chefserver-firstname=$CHEFSERVER_FIRSTNAME \
                  -P chefserver-lastname=$CHEFSERVER_LASTNAME \
                  -P chefserver-email=$CHEFSERVER_EMAIL \
                  -P chefserver-password=$CHEFSERVER_PASSWORD \
                  -P chefserver-pem=$CHEFSERVER_PEM \
                  -P chefserver-shortname=$CHEFSERVER_SHORTNAME \
                  -P chefserver-FULLNAME=$CHEFSERVER_FULLNAME \
                  -P image_id=$IMAGE_ID \
                  -P key_name=$KEY_NAME \
                  chefserver_single-stack
```

The above is the command's you'll need to run in order to boot the single-stack instance. I strongly suggest
you change them from the defaults there.


## HA chef server

The second on is the build out of the DRBD reference architecture from the
[install ha chef server with drbd](http://docs.chef.io/install_server_ha_drbd.html) documentation.
With this build, the stack creates the machines and networks you need for the setup,
but still requires you to run and build out the disks. Every machine that requires chef-server
has it downloaded to `/tmp/` and does a `dpkg` install of it. For the backend machines it already
installs the `drbd8-utils`. On my cluster it took about 1 hour and 40 minutes for the whole thing to
come up and running.

This build also assumes you are running on Ubuntu 14.04. If your glance image for Ubuntu 14.04 is not
called `ubuntu-trusty` you'll have to override the default, on the `heat stack-create`

Also you need to make sure you have ssh-agent working. Otherwise you won't be able to ssh into one of the
machines with a floating IP and ssh into the `be` machines. Add your key to the agent if you haven't
already: `ssh-add -K` and add something like the following to .ssh/config

```
Host *
  ForwardAgent yes
```

Because of heat, ec2-user is the login, the shell is sh, not bash, keep this in mind.

```
source openrc
SEC_ID=$(nova secgroup-list | awk '/ default / { print $2 }')
NET_ID=$(nova net-list | awk '/ ext-net / { print $2 }')
heat stack-create -f high_availability_chef_server-HOT.yml -P public_net=$NET_ID -P secgroup_id=$SEC_ID chefserver_HA-stack
```

The above will build out your HA stack. You should have the
[install ha chef server with drbd](http://docs.chef.io/install_server_ha_drbd.html) documentation, open and be familiar with it.
