# Mumble Build, Operate, Maintain
Mumble is a voice over IP (VoIP) application primarily designed for use by gamers and is similar to programs such as TeamSpeak.

Mumble uses a client–server architecture which allows users to talk to each other via the same server. It has a very simple administrative interface and features high sound quality and low latency. All communication is encrypted to ensure user privacy.

Mumble is free and open-source software, is cross-platform, and is released under the terms of the new BSD license.

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
Browse to `http://[CAPES-system]` and click the "CyberChef" from the "Services" dropdown.

Deploying manually:
```
mumble_passphrase=$(date +%s | sha256sum | base64 | head -c 32)
USER_HOME=$(getent passwd 1000 | cut -d':' -f6)
for i in mumble_passphrase; do echo "$i = ${!i}"; done > $USER_HOME/capes_credentials.txt
sudo yum install -y docker
sudo groupadd docker
sudo usermod -aG docker "$USER"
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo docker run -d --restart unless-stopped --name capes-mumble -p 64738:64738 -p 64738:64738/udp -v /var/lib/docker/volumes/mumble-data/_data:/data:z -e SUPW=$mumble_passphrase extra/mumble:latest
```

### Connect to the Mumble Server
1. Download the client of your choosing from the [Mumble client page](https://www.mumble.com/mumble-download.php)
1. Install
1. There will be considerable menus to navigate, just accept the defaults unless you have a reason not to and need a custom deployment.
1. Start it up and connect to
  1. Label: Whatever you want your channel to be called...maybe "CAPES" or something?
  1. Address: CAPES server IP address
  1. Port: 7000
  1. Username: whatever you want
  1. Password: this CAN be blank...but it shouldn't be **ahem**
  1. Click "OK"
  1. Select the channel you just created and click "Connect"
1. Right click on your name and select "Register"

### Delegating Permissions
Once a user has created an account and Registered, you can add them to the `admin` role.

1. Click on the Globe and select the channel that you created and click "Edit"
1. For the username, use the `SuperUser` account with the passphrase in `~/capes_credentials.txt` (the passphrase box will show up once you type `SuperUser`).
1. Right-click on main channel (likely "CAPES - Mumble Server") and select `Edit`
1. Go to the Groups tab
1. Select the `admin` role from the drop down
1. Type the user account you want to delegate admin functions to in the box
1. Click `Add` and then `Ok`
1. Click on the Globe and select the channel that you created and click "Edit"
1. Enter your username (not `SuperUser`) and your passphrase, and you can log in and perform administrative functions

### Creating Channels
1. Right-click on the main channel (likely "CAPES - Mumble Server") and select `Add`
1. Name and add the channel

#### Note
If the `Temporary` box is checked and greyed out, you do not have not been delegated rights. See [Delegating Permissions](#delegating-permissions) above.

## Troubleshooting
In the event that you have any issues, here are some things you can check to make sure they're operating as intended.

Is Docker running?
```
sudo systemctl status docker.service
```
Check to make sure it's active, if it isn't, try starting it with `sudo systemctl start docker.service`

Is the Cyberchef container running
```
sudo docker ps -a
```
Check to make sure that it isn't exited. Try `sudo docker start capes-mumble` or `sudo docker logs capes-mumble` to get a closer look.

Check with the Mumle project maintainers at https://wiki.mumble.info/wiki/ and https://wiki.mumble.info/wiki/Murmurguide

If you're still unable to access Mumble, [please file an issue](https://github.com/capesstack/capes-docker/issues).
