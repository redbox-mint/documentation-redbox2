---
title: "Installation on CentOS"
icon: "fas fa-info-circle"
layout: "installationDocumentationTemp"
isInstallationDocumentation: true
menuIndex: 3
---

Preparation 
===========
* This install guide has been tested on VM instances running Centos 7.
* This is a straight forward source install. Docker container based installation is documented [here](./dockerInstallation.html)
* Assumes commands are entered in the VM shell using Putty or similar.
* Also assumes you are not logged in as root but have sudo access.
* Get your VM’s IP address (e.g. hostname -I) or assigned URL.


# Dependencies

## Install Node.js 8.x

First you need to add yum repository of node.js to your system from the nodejs’ website using the following commands.

```
sudo yum install -y gcc-c++ make

curl -sL https://rpm.nodesource.com/setup_8.x | sudo -E bash -

sudo yum install -y nodejs
```
## Install Yarn

To install the Yarn package manager, run:
```
curl -sL https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo

sudo yum install yarn
```

## Install Java 8

```
sudo yum install java-1.8.0-openjdk
```

## Install MongoDB 4.0

Instructions on installing MongoDB for Ubuntu can be found [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/). Warning, these instructions change frequently, so rather than us documenting it here it is best to use their installation commands. Just follow the instructions for *installing the .rpm packages*.

**Start MongoDB.** 

Issue the following command to start mongod:

```
sudo service mongod start
```

**Verify that MongoDB has started successfully**

Verify that the mongod process has started successfully by checking the contents of the log file at /var/log/mongodb/mongod.log for a line reading: [initandlisten] waiting for connections on port 27017

## Install nginx 

Additional information on installing the NGINX web server can be found [here](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

**Install nginx**

```
sudo yum install nginx
```

**Update the config file for reverse proxy configuration**

Substitute your name or suitably meaningful name in the following

```
sudo su

cat << EOF >> /etc/nginx/conf.d/default.conf
server {
listen 80;
server_name  [yourname].redboxresearchdata.com.au;

#charset koi8-r;
#access_log  /var/log/nginx/log/host.access.log  main;

# redirect server error pages to the static page /50x.html
#
error_page   500 502 503 504  /50x.html;
location = /50x.html {
root   /usr/share/nginx/html;
}

add_header   Strict-Transport-Security "max-age=31536000; includeSubdomains";
add_header   X-Content-Type-Options nosniff;
add_header   X-Frame-Options DENY;

location / {
proxy_pass http://localhost:1337;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto http;

}

location /mint {
proxy_pass http://localhost:9001/mint;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto http;

}

}
EOF

exit
```

## Install GIT

```
sudo yum install git-core
```

## Install Yarn and PM2 dependencies

Once you have a Node.js environment installed install the following dependencies globally using the npm install -g <dependency> command:

```
sudo npm install -g yarn

sudo npm install -g pm2
```

## Install Mint

While Mint is a component of ReDBox, it gets installed and runs separately.

```
cd /opt

sudo wget "http://dev.redboxresearchdata.com.au/nexus/service/local/artifact/maven/redirect?r=releases&g=com.googlecode.redbox-mint&a=mint-distro&v=LATEST&e=tar.gz" -O "mint.tar.gz"

sudo tar -xvzf mint.tar.gz

sudo chown -R $USER:$(id -gn $USER) /opt/mint/

sudo rm mint.tar.gz
```

Edit the environment variables if required in */opt/mint/server/tf_env.sh*

Edit tf_env.sh and change line 6 *http://localhost:9001/mint/* replacing localhost:9001 with your server ip address or URL.

**Start Mint**

```
cd /opt/mint/server/

./tf.sh start
```

Checking it is working properly
View */opt/mint/home/logs/main.log* and check that it finishes with a message about all Queues Starting Successfully. If not check /opt/mint/home/logs/stdout.out and /opt/mint/home/logs/spring.log

## Install ReDBox

```
cd /opt

sudo wget "http://dev.redboxresearchdata.com.au/nexus/service/local/artifact/maven/redirect?r=snapshots&g=com.googlecode.redbox-mint&a=redbox-distro&v=2.0-SNAPSHOT&c=build&e=tar.gz" -O "redbox.tar.gz"

sudo tar -xvzf redbox.tar.gz

sudo chown -R $USER:$(id -gn $USER) /opt/redbox/

sudo rm redbox.tar.gz
```

Edit the environment variables if required in */opt/redbox/server/tf_env.sh*

**Start ReDBox** 

```
cd /opt/redbox/server/

./tf.sh start
```

Checking it is working properly

View * /opt/redbox/home/logs/main.log * and check that it finishes with a message about all Queues Starting Successfully. If not check * /opt/redbox/home/logs/stdout.out * and * /opt/redbox/home/logs/spring.log *

## Install RedBox-Portal

Currently the portal is installed using git clone, but the release will use NPM.

Make sure git clones with https.

```
sudo git config --global url."https://".insteadOf git://

git config --global url."https://".insteadOf git://
```

**Clone the repo** 

```
cd /opt

sudo git clone -b dev_build https://github.com/redbox-mint/redbox-portal.git
```

**Overlay any custom build files** 

If you have QCIF custom build files (or your own), they can be overlayed now.

**Populate the dependencies**

```
cd /opt/redbox-portal

sudo yarn
```

**Set Permissions**

```
sudo chown -R $USER:$(id -gn $USER) /opt/redbox-portal/
```

**Run the ReDBox configuration script** 

```
node configureInstall.js
```

When the configuration script asks you which apikey to use, let it default as it will generate the apikey and place it both into the ReDBox Storage * /opt/redbox/data/security/apiKeys.json*  and RedBox-Portal * /opt/redbox-portal/ecosystem.json *
When the configuration script asks you which URL the application will be accessed from, enter the IP address of the server you are installing to or URL assigned.

**Restart ReDBox Storage**

```
cd /opt/redbox/server/

./tf.sh restart
```
 
(check logs)

**Start ReDBox Portal**

```
cd /opt/redbox-portal/
```

Setup PM2 (Process Manager for Node.JS) to run on init  

```
pm2 startup systemd
```

and follow the instructions (if any) for pasting startup commands, e.g: * sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ec2-user --hp /home/ec2-user *

```
pm2 start ecosystem.json
```

**Save the PM2 configuration**

```
pm2 save
```

**Check for errors**

```
pm2 logs
```

Message should say “App lifted successfully”.
