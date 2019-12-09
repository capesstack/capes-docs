# Etherpad Build, Operate, Maintain
This project is created and maintained by the Etherpad team. All credit for the service goes to their talented team.

_Etherpad allows you to edit documents collaboratively in real-time, much like a live multi-player editor that runs in your browser. Write articles, press releases, to-do lists, etc. together with your friends, fellow students or colleagues, all working on the same document at the same time._

_All instances provide access to all data through a well-documented API and supports import/export to many major data exchange formats. And if the built-in feature set isn't enough for you, there's tons of plugins that allow you to customize your instance to suit your needs._

## Build

### Installation
Run the [CAPES deployment script](../deploy_capes.sh) or deploy manually.

Deploying with CAPES (recommended):
```
sudo yum install -y git
git clone https://github.com/capesstack/capes-docker.git
cd capes-docker
sudo sh deploy_capes.sh
```
Browse to `https://[CAPES-system]` and click the "Etherpad" from the "Services" dropdown.

Deploying manually:
```
etherpad_user_passphrase=$(date +%s | sha256sum | base64 | head -c 32)
sleep 1
etherpad_mysql_passphrase=$(date +%s | sha256sum | base64 | head -c 32)
sleep 1
etherpad_admin_passphrase=$(date +%s | sha256sum | base64 | head -c 32)
sleep 1
USER_HOME=$(getent passwd 1000 | cut -d':' -f6)
for i in {etherpad_user_passphrase,etherpad_mysql_passphrase,etherpad_admin_passphrase}; do echo "$i = ${!i}"; done > $USER_HOME/capes_credentials.txt
sudo yum install -y docker
sudo groupadd docker
sudo usermod -aG docker "$USER"
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo docker network create capes
sudo docker run -d  --network capes --restart unless-stopped --name capes-etherpad-mysql -v /var/lib/docker/volumes/mysql/etherpad/_data:/var/lib/mysql:z -e "MYSQL_DATABASE=etherpad" -e "MYSQL_USER=etherpad" -e MYSQL_PASSWORD=$etherpad_mysql_passphrase -e "MYSQL_RANDOM_ROOT_PASSWORD=yes" mysql:5.7
sudo docker run -d --network capes --restart unless-stopped --name capes-etherpad -e "ETHERPAD_TITLE=CAPES" -e "ETHERPAD_PORT=9001" -e ETHERPAD_ADMIN_PASSWORD=$etherpad_admin_passphrase -e "ETHERPAD_ADMIN_USER=admin" -e "ETHERPAD_DB_TYPE=mysql" -e "ETHERPAD_DB_HOST=capes-etherpad-mysql" -e "ETHERPAD_DB_USER=etherpad" -e ETHERPAD_DB_PASSWORD=$etherpad_mysql_passphrase -e "ETHERPAD_DB_NAME=etherpad" -p 5000:9001 tvelocity/etherpad-lite:latest
```
Browse to `https://[CAPES-system]:5000`

## Operate
Once you have completed the installation, you can start by creating simple pad's by visiting the page.

Additional functionality and extensibility of Etherpad is controlled with nodejs plugins.

### Administrative Functionality
Once you have installed Etherpad, you will want to browse to the administrative path and add some plugins for management.

#### adminpads
A plugin for Etherpad allowing you to list, search, and delete pads.

To install adminpads:
1. Browse to `https://[CAPES-system]:5000/admin`
1. Authenticate with `admin` and the credentials from the `~capes_credentials.txt` file
1. Select `Plugin Manager`
1. Search for `adminpads`
1. Click `Install`
1. Wait for it to install, refresh your browser, and you will notice a `Manage pads` tab, or browse to `https://[CAPES-system]:5000/admin/pads`

## Maintain

### Package Locations
Etherpad location - https://hub.docker.com/r/tvelocity/etherpad-lite/

### Update Etherpad
When it's time to update Etherpad, you can just grab the newest image and rerun the container build.
```
sudo docker pull tvelocity/etherpad-lite:latest
sudo docker stop capes-etherpad
sudo docker rm capes-etherpad
sudo docker run -d --network capes --restart unless-stopped --name capes-etherpad -e "ETHERPAD_TITLE=CAPES" -e "ETHERPAD_PORT=9001" -e ETHERPAD_ADMIN_PASSWORD=[from ~/capes_credentials.txt] -e "ETHERPAD_ADMIN_USER=admin" -e "ETHERPAD_DB_TYPE=mysql" -e "ETHERPAD_DB_HOST=capes-etherpad-mysql" -e "ETHERPAD_DB_USER=etherpad" -e ETHERPAD_DB_PASSWORD=[from ~/capes_credentials.txt] -e "ETHERPAD_DB_NAME=etherpad" -p 5000:9001 tvelocity/etherpad-lite:latest
```

## Troubleshooting
In the event that you have any issues, here are some things you can check to make sure they're operating as intended.

Is Docker running?
```
sudo systemctl status docker.service
```
Check to make sure it's active, if it isn't, try starting it with `sudo systemctl start docker.service`

Are the Etherpad and Etherpad MySQL containers running
```
sudo docker ps -a
```
Check to make sure that it isn't exited. Try `sudo docker start capes-etherpad` or `sudo docker logs capes-etherpad` to get a closer look. A lot of the issues with Etherpad usually have to do with the MySQL database not being up or failed logons. Check all of the creds, even if you have to manually enter them.

Is the site accessible locally?
```
curl [capes_IP]:5000
or, check from inside the container with
sudo docker exec -it capes-gitea bash
curl localhost:9001
```

Check with the Etherpad project maintainers at http://etherpad.org/

If you're still unable to access the Etherpad page from a web browser, [please file an issue](https://github.com/capesstack/capes-docker/issues).
