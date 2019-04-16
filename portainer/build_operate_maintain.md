# Portainer Build, Operate, Maintain
Portainer Community Edition is the foundation of the Portainer world. With over 700 million downloads throughout its history, it’s a powerful, open-source management toolset that allows you to easily build, manage and maintain Docker environments. And it’s completely free. By choosing from a growing range of extensions (available through a simple, low cost in-app subscription) Portainer’s core functionality can be extended to become the ideal toolset to manage docker based environments for organisations of any size.

_Portainer was developed to help customers adopt Docker container technology and accelerate time-to-value._

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
Browse to `http://[CAPES-system]` and click on "Portainer" from the "Services" drop down.

Deploying manually:
```
sudo yum install -y docker
sudo groupadd docker
sudo usermod -aG docker "$USER"
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo docker run -d --restart unless-stopped --name capes-portainer -v /var/lib/docker/volumes/portainer/_data:/data:z -v /var/run/docker.sock:/var/run/docker.sock -p 2000:9000 portainer/portainer:latest
```
Browse to `http://[CAPES-system]:2000`

## Operate
1. Browse to the Portainer UI and create your admin account
1. When it starts, click on the `Local` option and select `Connect`

## Maintain

### Package Locations
Portainer location - https://portainer.io

### Update CyberChef
When it's time to update CyberChef, you can just grab the newest image and rerun the container build.
```
sudo docker pull portainer/portainer:latest
sudo docker stop capes-portainer
sudo docker rm capes-portainer
sudo docker run -d --restart unless-stopped --name capes-portainer -v /var/lib/docker/volumes/portainer/_data:/data:z -v /var/run/docker.sock:/var/run/docker.sock -p 2000:9000 portainer/portainer:latest
```

## Troubleshooting
In the event that you have any issues, here are some things you can check to make sure they're operating as intended.

Is Docker running?
```
sudo systemctl status docker.service
```
Check to make sure it's active, if it isn't, try starting it with `sudo systemctl start docker.service`

Is the Portainer container running
```
sudo docker ps -a
```
Check to make sure that it isn't exited. Try `sudo docker start capes-portainer` or `sudo docker logs capes-portainer` to get a closer look.

Is the site accessible locally?
```
curl [capes_IP]:2000
or, check from inside the container with
sudo docker exec -it capes-portainer bash
curl localhost:2000
```

Check with the Portainer project maintainers at https://portainer.io

If you're still unable to access the CyberChef page from a web browser, [please file an issue](https://github.com/capesstack/capes-docker/issues).
