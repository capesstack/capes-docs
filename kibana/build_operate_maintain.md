# Kibana Build, Operate, Maintain
Kibana lets you visualize your Elasticsearch data and navigate the Elastic Stack.

In CAPES, Kibana is used to monitor the stack through the use of Elasticsearch and Beats.

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
Browse to `https://[CAPES-system]` and click the "TheHive" from the "Services" drop down.

Deploying manually:
```
sudo yum install -y docker
sudo groupadd docker
sudo usermod -aG docker "$USER"
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo docker network create capes
sudo mkdir -p /var/lib/docker/volumes/elasticsearch/capes/_data
sudo chown -R 1000:1000 /var/lib/docker/volumes/elasticsearch
sudo chown root: heartbeat.yml
sudo chmod 0644 heartbeat.yml
sudo chown root: metricbeat.yml
sudo chmod 0644 metricbeat.yml
sudo docker run -d --network capes --restart unless-stopped --name capes-elasticsearch-1 -v /var/lib/docker/volumes/elasticsearch-1/capes/_data:/usr/share/elasticsearch/data:z --ulimit memlock=-1:-1 -p 9200:9200 -p 9300:9300 -e "cluster.name=capes" -e "node.name=capes-elasticsearch-1" -e "cluster.initial_master_nodes=capes-elasticsearch-1" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" docker.elastic.co/elasticsearch/elasticsearch:[version]]
sudo docker run -d --network capes --restart unless-stopped --name capes-elasticsearch-2 -v /var/lib/docker/volumes/elasticsearch-2/capes/_data:/usr/share/elasticsearch/data:z --ulimit memlock=-1:-1 -e "cluster.name=capes" -e "node.name=capes-elasticsearch-2" -e "cluster.initial_master_nodes=capes-elasticsearch-1" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "discovery.seed_hosts=capes-elasticsearch-1,capes-elasticsearch-3" docker.elastic.co/elasticsearch/elasticsearch:[version]
sudo docker run -d --network capes --restart unless-stopped --name capes-elasticsearch-3 -v /var/lib/docker/volumes/elasticsearch-3/capes/_data:/usr/share/elasticsearch/data:z --ulimit memlock=-1:-1 -e "cluster.name=capes" -e "node.name=capes-elasticsearch-3" -e "cluster.initial_master_nodes=capes-elasticsearch-1" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "discovery.seed_hosts=capes-elasticsearch-1,capes-elasticsearch-2" docker.elastic.co/elasticsearch/elasticsearch:[version]
sudo docker run -d --network capes --restart unless-stopped --name capes-kibana --network capes -p 5601:5601 --link capes-elasticsearch:elasticsearch docker.elastic.co/kibana/kibana:[version]
sudo docker run -d --network capes --restart unless-stopped --name capes-heartbeat --network capes --user=heartbeat -v $(pwd)/heartbeat.yml:/usr/share/heartbeat/heartbeat.yml:z docker.elastic.co/beats/heartbeat:[version] -e -E output.elasticsearch.hosts=["capes-elasticsearch:9200"]
sudo docker run -d --network capes --restart unless-stopped --name capes-metricbeat --network capes --user=root -v $(pwd)/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:z -v /var/run/docker.sock:/var/run/docker.sock:z -v /sys/fs/cgroup:/hostfs/sys/fs/cgroup:z -v /proc:/hostfs/proc:z -v /:/hostfs:z docker.elastic.co/beats/metricbeat:[version] -e -E output.elasticsearch.hosts=["capes-elasticsearch-1:9200"]
```
Browse to `https://[CAPES-system]:5601`

## Operate
1. Browse to Kibana's UI
1. On the left side, click on the Management cog wheel
1. Click on Index Patterns
1. Click on Create index pattern
1. Type `heartbeat-`
1. Select `@timestamp`
1. Repeat for `metricbeat-`
1. You should now be able to select the Uptime app on the left side (1/2 circle with an arrow) to monitor CAPES uptime and the Infrastructure app on the left side to monitor the System and Docker resources

## Maintain

### Package Locations
Kibana location - https://www.elastic.co/guide/en/kibana/current/docker.html
Elasticsearch location - https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
Heartbeat location - https://www.elastic.co/guide/en/beats/heartbeat/current/running-on-docker.html
Metricbeat location - https://www.elastic.co/guide/en/beats/metricbeat/current/running-on-docker.html

### Update The Elastic Stack
When it's time to update Kibana (or any of the Elastic Stack), you can just grab the newest image and rerun the container build.
```
sudo docker pull docker.elastic.co/beats/heartbeat:[version]
sudo docker pull docker.elastic.co/beats/metricbeat:[version]
sudo docker pull docker.elastic.co/kibana/kibana:[version]
sudo docker pull docker.elastic.co/elasticsearch/elasticsearch:[version]
sudo docker stop capes-heartbeat
sudo docker stop capes-metricbeat
sudo docker stop capes-kibana
sudo docker stop capes-elasticsearch
sudo docker rm capes-heartbeat
sudo docker rm capes-metricbeat
sudo docker rm capes-kibana
sudo docker rm capes-elasticsearch
sudo docker run -d --network capes --restart unless-stopped --name capes-elasticsearch-1 -v /var/lib/docker/volumes/elasticsearch-1/capes/_data:/usr/share/elasticsearch/data:z --ulimit memlock=-1:-1 -p 9200:9200 -p 9300:9300 -e "cluster.name=capes" -e "node.name=capes-elasticsearch-1" -e "cluster.initial_master_nodes=capes-elasticsearch-1" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" docker.elastic.co/elasticsearch/elasticsearch:[version]]
sudo docker run -d --network capes --restart unless-stopped --name capes-elasticsearch-2 -v /var/lib/docker/volumes/elasticsearch-2/capes/_data:/usr/share/elasticsearch/data:z --ulimit memlock=-1:-1 -e "cluster.name=capes" -e "node.name=capes-elasticsearch-2" -e "cluster.initial_master_nodes=capes-elasticsearch-1" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "discovery.seed_hosts=capes-elasticsearch-1,capes-elasticsearch-3" docker.elastic.co/elasticsearch/elasticsearch:[version]
sudo docker run -d --network capes --restart unless-stopped --name capes-elasticsearch-3 -v /var/lib/docker/volumes/elasticsearch-3/capes/_data:/usr/share/elasticsearch/data:z --ulimit memlock=-1:-1 -e "cluster.name=capes" -e "node.name=capes-elasticsearch-3" -e "cluster.initial_master_nodes=capes-elasticsearch-1" -e "bootstrap.memory_lock=true" -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" -e "discovery.seed_hosts=capes-elasticsearch-1,capes-elasticsearch-2" docker.elastic.co/elasticsearch/elasticsearch:[version]
sudo docker run -d --network capes --restart unless-stopped --name capes-kibana --network capes -p 5601:5601 --link capes-elasticsearch:elasticsearch docker.elastic.co/kibana/kibana:[version]
sudo docker run -d --network capes --restart unless-stopped --name capes-heartbeat --network capes --user=heartbeat -v $(pwd)/heartbeat.yml:/usr/share/heartbeat/heartbeat.yml:z docker.elastic.co/beats/heartbeat:[version] -e -E output.elasticsearch.hosts=["capes-elasticsearch:9200"]
sudo docker run -d --network capes --restart unless-stopped --name capes-metricbeat --network capes --user=root -v $(pwd)/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:z -v /var/run/docker.sock:/var/run/docker.sock:z -v /sys/fs/cgroup:/hostfs/sys/fs/cgroup:z -v /proc:/hostfs/proc:z -v /:/hostfs:z docker.elastic.co/beats/metricbeat:[version] -e -E output.elasticsearch.hosts=["capes-elasticsearch-1:9200"]
```

## Troubleshooting
In the event that you have any issues, here are some things you can check to make sure they're operating as intended.

Is Docker running?
```
sudo systemctl status docker.service
```
Check to make sure it's active, if it isn't, try starting it with `sudo systemctl start docker.service`

Are the Elastic containers running
```
sudo docker ps -a
```
Check to make sure that it isn't exited. Try `sudo docker start capes-kibana` or `sudo docker logs capes-kibana` to get a closer look. Try the same for `capes-elasticsearch` and/or `capes-heartbeat` and/or `capes-metricbeat`.

Is the site accessible locally?
```
curl [capes_IP]:5601
or, check from inside the container with
sudo docker exec -it capes-kibana bash
curl localhost:5601
```

Check with the Elastic project maintainers at https://github.com/elastic

If you're still unable to access the CyberChef page from a web browser, [please file an issue](https://github.com/capesstack/capes-docker/issues).
