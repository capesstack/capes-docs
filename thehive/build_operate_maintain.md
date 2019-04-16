# TheHive Build, Operate, Maintain
A scalable, open source and free Security Incident Response Platform, tightly integrated with MISP (Malware Information Sharing Platform), designed to make life easier for SOCs, CSIRTs, CERTs and any information security practitioner dealing with security incidents that need to be investigated and acted upon swiftly.

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
Browse to `http://[CAPES-system]` and click the "TheHive" from the "Services" drop down.

Deploying manually:
```
sudo yum install -y docker
sudo groupadd docker
sudo usermod -aG docker "$USER"
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo docker network create capes
sudo mkdir -p /var/lib/docker/volumes/elasticsearch/thehive/_data
sudo chown -R 1000:1000 /var/lib/docker/volumes/elasticsearch
sudo docker run -d --network capes --restart unless-stopped --name capes-thehive-elasticsearch -v /var/lib/docker/volumes/elasticsearch/thehive/_data:/usr/share/elasticsearch/data:z -e "http.host=0.0.0.0" -e "transport.host=0.0.0.0" -e "xpack.security.enabled=false" -e "cluster.name=hive" -e "script.inline=true" -e "thread_pool.index.queue_size=100000" -e "thread_pool.search.queue_size=100000" -e "thread_pool.bulk.queue_size=100000" docker.elastic.co/elasticsearch/elasticsearch:5.6.13
sudo docker run -d --network capes --restart unless-stopped --name capes-thehive -e CORTEX_URL=capes-cortex -p 9000:9000 thehiveproject/thehive:latest --es-hostname capes-thehive-elasticsearch --cortex-hostname capes-cortex
```
Browse to `http://[CAPES-system]:9000`

## Operate
1. Browse to TheHive's UI
1. Click "Update Database" and create an administrative account

## Maintain

### Package Locations
TheHive location - https://blog.thehive-project.org/tag/docker/

### Update TheHive
When it's time to update CyberChef, you can just grab the newest image and rerun the container build.
```
sudo docker pull thehiveproject/thehive:latest
sudo docker stop capes-thehive
sudo docker rm capes-thehive
sudo docker run -d --network capes --restart unless-stopped --name capes-thehive -e CORTEX_URL=capes-cortex -p 9000:9000 thehiveproject/thehive:latest --es-hostname capes-thehive-elasticsearch --cortex-hostname capes-cortex
```

## Troubleshooting
In the event that you have any issues, here are some things you can check to make sure they're operating as intended.

Is Docker running?
```
sudo systemctl status docker.service
```
Check to make sure it's active, if it isn't, try starting it with `sudo systemctl start docker.service`

Is the TheHive container running
```
sudo docker ps -a
```
Check to make sure that it isn't exited. Try `sudo docker start capes-thehive` or `sudo docker logs capes-thehive` to get a closer look.

Is the site accessible locally?
```
curl [capes_IP]:9000
or, check from inside the container with
sudo docker exec -it capes-thehive bash
curl localhost:9000
```

Check with the CyberChef project maintainers at https://github.com/TheHive-Project

If you're still unable to access the CyberChef page from a web browser, [please file an issue](https://github.com/capesstack/capes-docker/issues).
