## Build and App server creation with ansible

#### BUILD SERVER

* Created a completely NEW buildserver vm on the FreeBSD host
* Gave that VM a static IP of 192.168.98.15 (see `HOW Static IP on a ubuntu`)
* Added that as a DNS entry to the DNS server
* Added the user Steve (during install) - configured that user for SSH and added my mac pub key to it's authorized_keys (see `HOW: SSH client setup`)
* Added the authorized_keys to the root account on the build server
* Now the complete BUILDSERVER ansible playbook should run on this box
* run the build server with: `./play apps/build_server/`


#### APP SERVER

* created a completely new appserver vm on the FreeBSD host
* gave that a static IP of 192.168.98.16
* added a DNS entry to the DNS server
* added a CNAME for present-me so that we can use that as the hostname in ansible
* Added the user Steve (during install) - configured that user for SSH and added my mac pub key to it's authorized_keys (see `HOW: SSH client setup`)
* Added the authorized_keys to the root account on the build server
* Added a user called `deployer` to the server and also added the SSH config and pub key and sudo etc.
* Create a `vault` for the db password (see `How: create a vault = DB password`)
* run the app server with: `./play apps/present_me/` - i.e. this is for our current apps


#### How: create a vault = DB password

ansible-vault encrypt_string --name db_password "YOUR_DB_PASSWORD"


#### App server SYSTEMD startup refers to an environment file

* called this: `home/{{ username }}/config/{{ app_name }}.conf` (ansible..)
* NB: we don't `export` these like in a standard linux environment file

```
#
# PRODUCTION ENV VARS
#
# DB
#
PRESENT_ME_PROD_DB_USERNAME=deployer
PRESENT_ME_PROD_DB_PASSWORD=REPLACE
PRESENT_ME_PROD_DB_DATABASE=present_me_prod
PRESENT_ME_PROD_DB_HOST=localhost
#
PRESENT_ME_PROD_SECRET_KEY_BASE=REPLACE
#
# URL settings
#
PRESENT_ME_PROD_URL_SCHEME=http
#PRESENT_ME_PROD_URL_HOST=p2u.cloud
PRESENT_ME_PROD_URL_HOST=present-me.forkin.local
PRESENT_ME_PROD_URL_PORT=80
#
# AWS configuration settings
#
PRESENT_ME_DEV_AWS_ACCESS_KEY_ID=REPLACE
PRESENT_ME_DEV_AWS_SECRET_ACCESS_KEY=REPLACE
PRESENT_ME_PROD_AWS_BUCKET=presentme-production
#
# MAILGUN settings
#
PRESENT_ME_PROD_MAILGUN_DOMAIN=https://api.mailgun.net/v3/mg.netflakes.co
PRESENT_ME_PROD_MAILGUN_KEY=REPLACE
#
PORT=5010
```

#### Initial app server steps

* Via `psql` create the DB
* First run the app once without the `DomainNameList`
* Then `mix edeliver migrate` and then via `remote-console` run `PresentMe.MigrateTenants.run()`
* Finally turn the `DomainNameList` back on - TODO: add some config code to be able turn this on/off


#### Since edeliver uses the old upstart if fails to restart systemd!

* create this file `/etc/profile.d/00-aliases.sh`
* add this content:

```
alias start='systemctl enable'
alias stop='systemctl disable'
```

#### HOW: SSH client setup

```
mkdir ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t rsa
```

#### HOW Static IP on a ubuntu

>Replace the `dhcp` entry with the `static` one

```
# The primary network interface
#auto enp0s2
#iface enp0s2 inet dhcp

# The primary network interface
auto enp0s2
iface enp0s2 inet static
	address 192.168.98.16
	network 192.168.98.0
	netmask 255.255.255.0
	broadcast 192.168.98.255
	gateway 192.168.98.1
	dns-nameservers 192.168.98.9
```
