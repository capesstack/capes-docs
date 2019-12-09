# Rocketchat Build, Operate, Maintain
Rocket.Chat is an open source team collaboration platform that enables banks, NGOs, startups, and governmental organizations to have their own chat tool, customize its look and feel, choose their users, and securely manage data.

_Rocket.Chat is a fast-growing platform that is now installed on over 180k servers and counts over 10m users worldwide as well as having an active, passionate community of over 700 developer-contributors who help Rocket.Chat’s core team of developers to constantly improve the product. Rocket.Chat’s long-term vision is to replace email with a real-time federated communications platform and offer services to enable businesses to be built using Rocket.Chat._

## Build

### Installation
Run the [CAPES deployment script](../deploy_capes.sh) or deploy manually:

Deploying with CAPES (recommended):
```
sudo yum install -y git
git clone https://github.com/capesstack/capes-docker.git
cd capes-docker
sudo sh deploy_capes.sh
```
Browse to `https://[CAPES-system]` and click on "Rocketchat" from the "Services" dropdown.

Deploying manually:
```
sudo yum install -y docker
sudo groupadd docker
sudo usermod -aG docker "$USER"
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo docker run -d --network capes --restart unless-stopped --name capes-rocketchat-mongo -v /var/lib/docker/volumes/rocketchat/_data:/data/db:z -v /var/lib/docker/volumes/rocketchat/dump/_data:/dump:z mongo:latest mongod --smallfiles
sudo docker run -d --network capes --restart unless-stopped --name capes-rocketchat --link capes-rocketchat-mongo -e "MONGO_URL=mongodb://capes-rocketchat-mongo:27017/rocketchat" -e "ROOT_URL=http://localhost:3000" -p 3000:3000 rocketchat/rocket.chat:latest
```
Browse to `http://[CAPES-system]:3000`

## Operate
1. Browse to `https://[CAPES-system]` and click on "Rocketchat" from the "Services" dropdown or `http://[CAPES-system]:3000`
1. Complete the `Admin Info`
1. Complete the `Organization Info`
1. Complete the `Server Info`
1. In the `Register Server` section, select `Keep standalone`

## Maintain

### Package Locations
Rocketchat location - https://hub.docker.com/_/rocket-chat

### Update Rocketchat
When it's time to update CyberChef, you can just grab the newest image and rerun the container build.
```
sudo docker pull rocketchat/rocket.chat:latest
sudo docker stop capes-rocketchat
sudo docker rm capes-rocketchat
sudo docker run -d --network capes --restart unless-stopped --name capes-rocketchat --link capes-rocketchat-mongo -e "MONGO_URL=mongodb://capes-rocketchat-mongo:27017/rocketchat" -e "ROOT_URL=http://localhost:3000" -p 3000:3000 rocketchat/rocket.chat:latest
```

## Troubleshooting
In the event that you have any issues, here are some things you can check to make sure they're operating as intended.

Is Docker running?
```
sudo systemctl status docker.service
```
Check to make sure it's active, if it isn't, try starting it with `sudo systemctl start docker.service`

Is the Rocketchat and associated MongoDB container running
```
sudo docker ps -a
```
Check to make sure that they are't exited. Try `sudo docker start capes-rocketchat` or `sudo docker logs capes-rocketchat` to get a closer look.

Is the site accessible locally?
```
curl [capes_IP]:3000
or, check from inside the container with
sudo docker exec -it capes-rocketchat bash
curl localhost:3000
```

Check with the Rocketchat project maintainers at https://github.com/RocketChat.

If you're still unable to access the CyberChef page from a web browser, [please file an issue](https://github.com/capesstack/capes-docker/issues).
