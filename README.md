# chef OpenStack Heat Orchestration Templates (HOT)

There are two templates here to help you create your chef server infrastructure.

## Standalone chef Server

The first is a standalone instance which is designed to build out a chef server
on the network you declare to go on. It also assigns a floating IP to the instance,
so other machines outside your tenant network can get to it.

```
source openrc
NET_ID=$(nova net-list | awk '/ ext-net / { print $2 }')
TENANT_ID=$(nova net-list | awk '/ local-net / { print $2 }')
TENANT_SUBNET=$(neutron subnet-list | awk '/ local-subnet / { print $2 }')
CHEFSERVER_CORE=chef-server-core_12.1.2-1_amd64.deb
CHEFSERVER_USERNAME=admin
CHEFSERVER_FIRSTNAME=Admin
CHEFSERVER_LASTNAME=Jacob
CHEFSERVER_EMAIL=admin@fake-email.org
CHEFSERVER_PASSWORD=123456
CHEFSERVER_SHORTNAME=defaultinc
CHEFSERVER_FULLNAME="Default Inc."
IMAGE_ID=ubuntu-trusty
KEY_NAME=admin
heat stack-create -f single_chef_server-HOT.yml
                  -P public_net=$NET_ID \
                  -P tenant_subnet=$TENANT_SUBNET \
                  -P tenant_net=$TENANT_NET \
                  -P chefserver-core=$CHEFSERVER_CORE \
                  -P chefserver-username=$CHEFSERVER_USERNAME \
                  -P chefserver-firstname=$CHEFSERVER_FIRSTNAME \
                  -P chefserver-lastname=$CHEFSERVER_LASTNAME \
                  -P chefserver-email=$CHEFSERVER_EMAIL \
                  -P chefserver-password=$CHEFSERVER_PASSWORD \
                  -P chefserver-shortname=$CHEFSERVER_SHORTNAME \
                  -P chefserver-FULLNAME=$CHEFSERVER_FULLNAME \
                  -P image_id=$IMAGE_ID \
                  -P key_name=$KEY_NAME \
                  chefserver_standalone_server
```

The above are the command's you'll need to run in order to boot the standalone instance. I strongly suggest
you change them from the defaults there.

After you log in the web-ui via `https://<floating-ip>`, you should go to `https://<floating-ip>/organizations/<CHEFSERVER_SHORTNAME>/getting_started`
and pull that down. Go ahead and unzip the `.zip` file it gives you. Change directory into the `chef-repo` that it should have created for you.
`knife` should work talking to your new chef server. A good test is either a `knife status` or `knife client list` in your `chef-repo`. With `status`
nothing should be returned, and when you use the `client` command should see your validator.pem file name come back.

## HA chef server

*NOTE*: Please read this section completely if you are planning on attempting this. Then read the referenced
[install ha chef server with drbd](http://docs.chef.io/install_server_ha_drbd.html) documentation.

The HA chef server template will build out the  [DRBD](http://drbd.linbit.com/) reference architecture from the
[install ha chef server with drbd](http://docs.chef.io/install_server_ha_drbd.html) documentation.
With this build, the stack creates the machines and networks you need for the setup,
but still requires you to run and build out the disks and configure chef-server(s). Every machine that requires chef-server package
has it downloaded to `/tmp/` and does a `dpkg` install of it for you. On the backend machines (be-1 and be-2), the template already
installs the `drbd8-utils` for you

This process could take as long as 2 hours depending on your cluster and machine resources. It should be completed with the following line:

```
Cloud-init v. 0.7.5 finished at Mon, 27 Jul 2015 21:48:34 +0000. Datasource DataSourceOpenStack [net,ver=2].
```

in the bootup log in the instance otherwise it's still in the process of building.

This build also assumes you are running on Ubuntu 14.04. If your glance image for Ubuntu 14.04 is not
called `ubuntu-trusty` you'll have to override the default, on the `heat stack-create`

An example of this would be something like:

```
source openrc
SEC_ID=$(nova secgroup-list | awk '/ default / { print $2 }')
NET_ID=$(nova net-list | awk '/ ext-net / { print $2 }')
IMAGE_ID=ubuntu-trusty
KEY_NAME=admin
heat stack-create -f high_availability_chef_server-HOT.yml
                  -P public_net=$NET_ID \
                  -P secgroup_id=$SEC_ID \
                  -P image_id=$IMAGE_ID \
                  -P key_name=$KEY_NAME \
                  chefserver_HA-stack
```

The above script will build out your HA stack. You should have the
[install ha chef server with drbd](http://docs.chef.io/install_server_ha_drbd.html) documentation, open and be familiar with it.

Because of heat, ec2-user is the login, the shell is sh, not bash, keep this in mind.

Also you need to make sure you have ssh-agent working. Otherwise you won't be able to ssh into one of the
machines with a floating IP and ssh into the `be` machines. Add your key to the agent if you haven't
already: `ssh-add -K` and add something like the following to `.ssh/config`

```
Host *
  ForwardAgent yes
```

## License and Authors

Author:: Chef Partner Engineering (<partnereng@chef.io>)

Copyright:: Copyright (c) 2015 Chef Software, Inc.

License:: Apache License, Version 2.0

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the License at

```
http://www.apache.org/licenses/LICENSE-2.0
```

Unless required by applicable law or agreed to in writing, software distributed under the
License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
either express or implied. See the License for the specific language governing permissions
and limitations under the License.
