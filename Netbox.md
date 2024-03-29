# Netbox Introduction

## **Febuary 2024 Update:** I actually use Nautobot now as it supports a significant amount of automation for your infrastructure and was forked from Netbox. I'll make a post in the future about the advantages and features that come with Nautobot

## Original 2020 post

======
I use Netbox for a frontend interface to a PostgreSQL database that holds any DCIM (Data Center Infrastructure Management) and IPAM (IP Address Management) data. The goal of implementing netbox should be to ensure a “source of truth” to check against in the event of a problem. A source of truth/database of record should reflect what the reality is for your ecosystem at all times.

Netbox is what I would call a next generation source of truth as it is capable of provisioning and deploying as well as importing and documenting any changes that need to be made.

[Read the Docs](https://netbox.readthedocs.io/en/stable/)

[Demo Enviornment](https://demo.netbox.dev/)

Overview
--------

This is the homepage for Netbox from here you can navigate through sub menus.

This Organization section is likely designed/ideal for MSP’s and large carrier grade ISP’s. The terms Sites and Tenants are just vague “boxes” that I interpret as vague variables.

I tried to use Sites as a macroscopic view of big picture things. For my situation I could go with “Lab” and “Production” or divide it out by different LAN’s in my house.

Tenants are more intrinsic to DCIM related functions. If I was renting out VPS resources I could use this more. Right now I could tag it with who the box works for (Plex for house, ESXI for me).

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/organization.png)

This section is for storing information about “physical things” from servers/switches to physical racks. In a fresh netbox install this is the section you will live in for a while.

**Racks** store a topology of equipment and in addition to this you can colour code whats on the rack and see a visual representation of what is where (switches at top, then routers, etc.). You can also upload photos of what the rack looks like and that will be stored here as well.

**Device Types** are the models of devices. Here you would have things like a Cisco 1850 vs a Cisco Catalyst 1000, etc.

**Devices**  are the physical boxes and what they are called. Here you would have things like Core Router 01 vs Core Router 02.

**Connections** are pretty granular for what I’ve been doing. You can add specific types of interfaces or cables here to be available when documenting things down the road.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/devices.png)

The IPAM section documents all layer 3 information.

VRF’s are not in place for my current setup so undocumented

Aggregates would document EtherChannel links.

Prefixes are the subnets you have (x.x.x.x/x) and will automatically nest inside parent prefixes.

IP Addresses are the individual host addresses you have entered.

VLANs will list what VLAN’s you have and what devices they are on.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/ipam.png)

Providers would be useful for documenting upstream’s with their ASN’s.

Circuits would be where you can tag circuit ID’s on an interface.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/circuits.png)

**Secrets** are text encrypted with RSA. The Enterprise use case for this is to tag each devices unique local login to the entry in Netbox. I use it as a pseudo password vault and have a device specifically made for storing all passwords on (not best practice).

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/Secrets.png)

Change Log obviously tracks all changes, you can scroll down to the bottom here and see a complete list of all changes. You can use Ansible here to automatically write any update from the change log to a slack channel or something which would be invaluable if the database became inaccessible and you couldn’t view the change log normally. Changelog can or can not be migrated over with a SQL migration at the discretion of the maintainer.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/changelog.png)

Add a Device
------------

While you can and should add the overall prefxies that you own (192.168.0.0/24) you should make it a priority to document what devices/interfaces have what IP’s. This means you will have to create your base layer of devices before getting into the IPAM portion.

Start by signing in via the login located in the upper right.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login.png)

You will need to add a Site before anything else. You can use the dropdown or  navigate to the Sites page from the Organization section on the homepage.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sites1.png)

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sites2.png)

Fields in Bold are required but anything else is optional. I would always recommend using a description whenever available. Click the blue create button at the very bottom of the page to continue (not pictured).

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-2.png)

You will automatically be in your new site. In Netbox there are many ways to do the same thing. For the next step I will recommend using the devices drop down and using the green add button next to manufacturers.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-4.png)

You will need to list whatever device vendor you have in the name field and then hit the blue create button.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/manufacturers.png)

I would now recommend using the Devices dropdown to add a device type under the previous vendor.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-3.png)

Here you would specify if it was a Catalyst 1000 or in this case an IBM 5100 (not a Sun box, forgive the reference to early 2000’s conspiracies).

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-5.png)

Use the dropdown to create a device role that fits what can define your device. Core Router, Ring Switch, Datacenter, etc.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/device-make.png)

You finally have all the required fields to populate/create a device. Use the devices dropdown and fill out all bolded fields.

Some optional fields like Regions we did not create but you can easily add as needed.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-6-1024x432.png)

Adding an Interface and IP address
----------------------------------

By default if you created a device yourself (steps above) then there will be no interfaces or specifics listed. You can import a template to preload 48 interfaces on your switch instead of manually adding each one but I am not covering templates in this tutorial.

If you just created a device then you will be on your devices page already. You will want to use the add components drop down in the upper right and selecting interface.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/components.png)

Fill out required fields here. The description field is not required but should be. You can mass add a ton of interfaces by doing something like \[fe\] – 0/0/1.1\[1-9\] which would add \[fe\] – 0/0/1.11 through \[fe\] – 0/0/1.19. I’m just adding one interface here FastEthernet – 0/0/1. As this is a copper connection I have to change the dropdown from the default type of virtual  and select a copper connection under the Ethernet category with the correct speed.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/add-interface.png)

You should have arrived back on the original device page after creating the interface. At the bottom of the page use the green add button to apply an ip address to the interface using cidr notation.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-7-1024x89.png)

JSON Constraints
----------------

This is a VERY niche use case and most of what you would do with JSON constraints will likely be implemented in future versions of Netbox. For the sake of showcasing what it can do though I will demonstrate a very simple profile that I made.

As it stands now any user that is authorized to use an RSA key to decrypt something can unlock any secret across netbox. You cannot restrict things by RSA key as all RSA keys are cryptographically linked once theyre in netbox. My goal was to make it so that my test user “IPAM” couldn’t unlock something under my “tavishr” account but could unlock anything under the “Operations” role.

To apply a JSON constraint you will have to go to the admin panel, then under the USERS group on the left there should be a permissions section.  You enable or disable permissions here in addition to creating new ones. You can click the green add button in the lower left or click the grey add permission button in the upper right. These can also be added when making a group in the section above this.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-8.png)

I am only allowing view permissions but you could grant full access and a JSON constraint would still pear things out.

Additional actions from what I’ve seen have to deal with things like NAPALM permissions.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/json-permission.png)

This is a two part process to filter out passwords that don’t belong to you. I am going to add the first rule for just accessing secrets. Without this step you will either not be able to go into the secret category from the main netbox page or you will get into secrets and not have anything filtered. Essentially Secret>Secret is your ability to click on the secrets page or use the secrets dropdown and access it that way.

Most of the object types are actually database tables. You can access the netbox postgresql database and open one of these tables.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/Object-types.png)

You will have to apply the rules to somebody at this point. Your first option is to do it by group, the only group I have made so far is operations. If you don’t apply to a group you will have to apply to a user, in this case IPAM. IPAM is already in the operations group so

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-9.png)

A Constraint acts as a filter/lens to where you can only see things allowed by the constraint. You would typically do something like “in devices category only show user A machines that have a status of up and that are in this ip range” or whatever type of filtering you see fit. In a big organization this of course could be used to keep people in different regions from seeing/manipulating  things outside of their jurisdiction.

The syntax for Constraints is

{“field” : “query”}

Here I’m using {“name : “ops pwd1”} which means that this account should only list something with ops pwd1 as the field name. This isnt quite the case which is why we need another rule to reinforce this.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-10.png)

The Second rule should look the same except for the object type will change to secret role

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-11.png)

And the new constraint will be {“name”:”operations”}. Since IPAM has the secret role named “Operations” this should prevent this user from seeing any secret role that isnt tagged as operations. This is not the case by itself but with rule 1 in place as well it is effective.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/netbox-login-12.png)

Setup via terminal - Linux
--------------------------

During any ubuntu install its always best practice to open with an: apt-get update and apt-get upgrade

apt-get install -y postgresql libpq-dev

systemctl start postgresql-9.6

systemctl enable postgresql 9.6

sudo -u postgres psql

CREATE DATABASE netbox;

CREATE USER netbox WITH PASSWORD ‘Netbox!’;

GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;

\\q

psql –username netbox –password –host localhost netbox

\\q

apt-get install -y redis-server

 apt-get install -y python3.6 python3-pip python3-venv python3-dev build-essential libxml2-dev libxslt1-dev libffi-dev libpq-dev libssl-dev zlib1g-dev

pip3 install –upgrade pip

apt-get install -y git

mkdir -p /opt/netbox/ && cd /opt/netbox/

git clone -b master [https://github.com/netbox-community/netbox.git](https://github.com/netbox-community/netbox.git) .

adduser –system –group netbox

chown –recursive netbox /opt/netbox/netbox/media/

cd /opt/netbox/netbox/netbox/

cp configuration.example.py configuration.py

python3 ../generate\_secret\_key.py

copy the generated key

nano configuration.py

ensure:

ALLOWED\_HOSTS=\[\*\]DATABASE = {

1. ‘NAME’ : ‘netbox’,
2. ‘USER’ : ‘netbox’,
3. ‘PASSWORD’ : ‘netbox!’,

REDIS = {

PORT: 6379

SSL: False

SECRET\_KEY:

input your copy pasted code

/opt/netbox/upgrade.sh

source /opt/netbox/venv/bin/activate

cd /opt/netbox/netbox

python3 manage.py createsuperuser

run through user generation, skip email

python3 manage.py runserver 0.0.0.0:8000 –insecure

cd /opt/netbox

cp  contrib/gunicorn.py /opt/netbox/gunicorn.py

cp contrib/\*.service /etc/systemd/system/

systemctl daemon-reload

systemctl start netbox netbox-rq

systemctl enable netbox netbox-rq

systemctl status netbox.service

openssl req -x509 -nodes -days 365 -newkey rsa:2048 \\ -keyout /etc/ssl/private/netbox.key \\ -out /etc/ssl/certs/netbox.crt

apt-get install -y nginx

cp /opt/netbox/contrib/nginx.conf /etc/nginx/sites-available/netbox

cd /etc/nginx/sites-enabled/

rm default

ln -s /etc/nginx/sites-available/netbox

cd /etc/nginx/sites-available/netbox

nano netbox

replace netbox.com with custom ip address

service nginx restart

Setup via Docker - Linux
------------------------

all of the following commands are sudo minus the CD:

1. apt-get update
2. apt-get upgrade
3. apt-get install docker.io
4. usermod -aG docker $USER
    1. re log
5. curl -L “[https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)](https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname%20-s)-$(uname%20-m))” -o /usr/local/bin/docker-compose.
6. chmod +x /usr/local/bin/docker-compose
7. systemctl start docker
8. systemctl enable docker
9. git clone -b release [https://github.com/netbox-community/netbox-docker.git](https://github.com/netbox-community/netbox-docker.git)
10. cd netbox-docker
11. nano env/netbox.env
    1. here I just set netbox/netbox logins for superuser  – do not use this on a production box with a public ip. MUCH better ways to secure on a production box but this is only a backup on an internal ip.
12. nano docker-compose.override.yml

version: ‘3.4’

services:

  nginx:

    ports:

    -8000:8080

1. docker-compose pull
2. docker-compose up
3. in browser go to  [http://0.0.0.0:8000/](http://0.0.0.0:8000/) – netbox/netbox for mine API Token: 0123456789abcdef0123456789abcdef01234567
4. to shutdown netbox: docker-compose stop

Take a database backup

docker-compose exec -T postgres sh -c ‘pg\_dump -cU $POSTGRES\_USER $POSTGRES\_DB’ | gzip > db\_dump.sql.gz

Restore that database:

gunzip -c db\_dump.sql.gz | docker-compose exec -T postgres sh -c ‘psql -U $POSTGRES\_USER $POSTGRES\_DB’

SQL Dump/Restore
----------------

 The command to dump postgresql to a file is as follows:

pg\_dump yourdatabase > yourdatabase.sql

This command is to be run from the filesystem/bash NOT from inside the database. The account you are using MUST be superuser and have all rights to the database in question. On ubuntu the authentication should be IDENT so it will automatically check your system user against a list of database users. You will want these to match so create either a system user to match a known database user or a database user to match a known system user.

on ubuntu your backdoor into the database will always be:

sudo -u postgres psql

alternatively you can login to your database the long way

psql –username yourusername –password –host localhost netbox

use:

\\du

when inside the database to see a list of users and what access they have. If you need to line up a database user with your system user:

CREATE USER yourusername WITH PASSWORD ‘yourpassword’;

GRANT ALL PRIVILEGES ON DATABASE yourdatabase TO yourusername;

ALTER USER yourusername WITH SUPERUSER;

remember to always end your statements in a database with a semicolon or you will get errors.

you should be able to use pg\_dump from your bash terminal with yourusername.

the yourdatabase.sql file should be in /home/yourusername unless you specified otherwise.

Restore from backup:

Go to your home directory and make sure that you paste your backup in here (anywhere)

systemctl stop netbox

sudo su postgres

psql -c ‘drop database netbox’

psql -c ‘create database netbox’

psql netbox < netbox.sql
