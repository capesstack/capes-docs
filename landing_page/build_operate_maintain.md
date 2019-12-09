# CAPES Land Page Build, Operate, Maintain
The CAPES landing page was developed to give a singular place for operators to go to access all of the CAPES toolsets. Feel free to customize it to meet your environment. The only exception to this is Mumble as it is not a web application.

**This will distill the basic installation and configuration of the HTTP server, nginx, as it relates to the CAPES project.**

**Yes, believe me, I know there are a lot of ways to do this, and that when you're running more than 1 web service, there are individual `.conf` files to use - I get it :) That said, as it relates to CAPES, we only need a single `index.html` page and just allow `nginx` to do it's thing there without making a configuration file for the entire CAPES web application. I'm certainly open to reasons to do this in a more complex way, but as it sits right now, I didn't see the need. PR and Issues are gleefully welcome.**

**For additional configuration and deployment options, see the official [nginx wiki](https://www.nginx.com/resources/wiki/).**

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
Browse to `https://[CAPES-system]`

Deploying manually:
```
sudo yum install -y docker
sudo groupadd docker
sudo usermod -aG docker "$USER"
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo docker run -d --restart unless-stopped --name capes-landing-page -v $(pwd)/landing_page:/usr/share/nginx/html:z -p 80:80 nginx:latest
```
Browse to `https://[CAPES-system]`

## Operate
The landing page runs as the `nginx` user, it has no shell and cannot be logged on as.

### Cosmetics
There is an included `favicon.ico` file for the little image that shows up on browser tabs, you can update this with your own logo in `/usr/share/nginx/html/favicon.ico`. Its dimensions should be `32x32`.

You can mount the location of the `favicon.ico` file into the container like this:
```
sudo docker run -d --restart unless-stopped --name capes-landing-page -v $(pwd)/landing_page:/usr/share/nginx/html:z -v [location-of-local-favicon.ico]:/usr/share/nginx/html/favicon.ico:z -p 80:80 nginx:latest
```

To update the `Your Logo Here` graphic, place your logo in the `/usr/share/nginx/html/images` directory. Its dimensions should be `250x250`. You will also need to update `/usr/share/nginx/html/index.html` with the logo name:

You can mount the location of your logo file into the container like this:
```
sudo docker run -d --restart unless-stopped --name capes-landing-page -v $(pwd)/landing_page:/usr/share/nginx/html:z -v [location-of-local-local-logo.png]:/usr/share/nginx/html/images/your-logo.png:z -p 80:80 nginx:latest
```

## Maintain
You can make changes to the nginx configuration or update your environment as necessary.

### Package Location
CAPES nginx landing page location - https://hub.docker.com/_/nginx

### Update Configuration
In the event that you just want to update some of the content on the page, you can simply do that locally on the host and then run `sudo docker restart capes-landing-page`.

### Update nginx
When it's time to update the Nginx image, you can just grab the newest image and rerun the container build.
```
sudo docker pull nginx:latest
sudo docker stop capes-landing-page
sudo docker rm capes-landing-page
sudo docker run -d --restart unless-stopped --name capes-landing-page -v $(pwd)/landing_page:/usr/share/nginx/html:z -p 80:80 nginx:latest
```

## Troubleshooting
In the event that you have any issues, here are some things you can check to make sure they're operating as intended.

Is Docker running?
```
sudo systemctl status docker.service
```
Check to make sure it's active, if it isn't, try starting it with `sudo systemctl start docker.service`

Is the Nginx container running
```
sudo docker ps -a
```
Check to make sure that it isn't exited. Try `sudo docker start capes-landing-page` or `sudo docker logs capes-landing-page` to get a closer look.

Is the site accessible locally?
```
curl [capes_IP]
or, check from inside the container with
sudo docker exec -it capes-landing-page bash
curl localhost
```
If you're still unable to access the CAPES page from a web browser, [please file an issue](https://github.com/capesstack/capes/issues).
