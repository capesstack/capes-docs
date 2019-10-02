# Documentation
Please see below for Build, Operate, Maintain specifics on the different web applications
* [Post Installation](https://github.com/capesstack/capes-docs/tree/master/docs#post-installation)
* [CAPES Landing Page](landing_page/build_operate_maintain.md)  
* [CyberChef](cyberchef/build_operate_maintain.md)
* [Gitea](gitea/build_operate_maintain.md)  
* [Etherpad](etherpad/build_operate_maintain.md)  
* [Rocketchat](rocketchat/build_operate_maintain.md)  
* [TheHive](thehive/build_operate_maintain.md)  
* [Cortex](thehive/build_operate_maintain.md)  
* [Mumble](mumble/build_operate_maintain.md)  
* [Kibana](kibana/build_operate_maintain.md)  
* [Portainer](portainer/build_operate_maintain.md)  

## Requirements
While the OS version isn't a hard requirement, all testing and development work has been done with `CentOS 7.6.1810 (Core)`.

## Secure by Design

### Secure Deployment
There is an inherent risk to deploying web applications to the Internet or to a contested networking enclave -- basically, somewhere the bad guys can get to.

To address this, the CAPES project has done a few things to help protect these web applications and left a few things for you, the operator, to close in on as your need requires.

### Operating System
While there are a lot of projects that are developed using Ubuntu (many of these service creators still follow that path), CAPES chose to use CentOS because of a few different reasons:  
1. CentOS is the open source version of Red Hat Enterprise Linux (RHEL)
    - Many enterprises use RHEL as their Linux distribution of choice because you can purchase support for it
    - We wanted to use a distribution that could easily be ported from an open source OS to the supported OS (RHEL)
1. CentOS uses Security Enhanced Linux (SELinux) instead of AppArmor
    - SELinux uses context to define security controls (for example, I know a text editor shouldn't talk to the Internet because it's a text editor, not a browser)
    - AppArmor uses prescripted rules to define security controls (for example, I know that a text editor shouldn't talk to the Internet because someone **told** me it shouldn't)

### Implementation
While the `firewalld` service is running on CAPES and the only ports listening have services attached to them, you should still consider using a Web Application Firewall (WAF), an Intrusion Detection System (IDS), or a Network Security Monitor (like [RockNSM](rocknsm.io) - which has an IDS integrated on top of a litany of other goodies) to ensure that your CAPES stack isn't being targeted.

If possible, CAPES, just like a passive NSM, should **not** be on the contested network. This will prevent it from being targeted by aggressors.

## Installation
Generally, the CAPES ecosystem is meant to run as a whole, so the preferred usage will be to install CAPES with the `deploy_capes.sh` script in the root directory of this repository. Additionally, if there is a service that you do not want, you can comment that service out of the `deploy_capes.sh` file as they are documented with service tags.

### Build your OS (CentOS 7.6)
This is meant to help those who need a step-by-step build of CentOS, securing SSh, and getting ready to grab CAPES. If you don't need this guide, skip on down to [Get CAPES](#get-capes).
1. Download the latest version of [CentOS Minimal](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso)
1. Build a VM or a physical system with 4 cores, 8 GB of RAM, and a 20 GB HDD at a minimum
    - Don't use any of the "easy install" options when setting up a VM, we're going to make some config changes in the build process
    - I recommend removing all the things that get attached that you're not going to use (speakers, BlueTooth, USB, printer support, floppy, web camera, etc.)
1. Fire up the VM and boot into Anaconda (the Linux install wizard)
1. Select your language
1. Start at the bottom-left, `Network & Host Name`
    - There is the `Host Name` box at the bottom of the window, you can enter a hostname here or we can do that later...in either case, `localhost` is a poor choice
    - Switch the toggle to enable your NIC
      - Click `Configure`
      - Go to `IPv6 Settings` and change from `Automatic` to `Ignore`
      - Click `Save`
    - Click `Done` in the top left
1. Next click `Installation Destination`
    - Select the hard disk you want to install CentOS to, likely it is already selected unless you have more than 1 drive
    - Click `Done`
1. Click `kdump`
    - Uncheck `Enable kdump`
    - Click `Done`
1. `Installation Source` should say `Local media` and `Software Selection` should say `Minimal install` - no need to change this
1. Click `Date & Time`
    - `Region` should be changed to `Etc`
    - `City` should be changed to `Coordinated Universal Time`
    - `Network Time` should be toggled on
    - Click `Done`
    - Note - the beginning of these install scripts configures Network Time Protocol (NTP). You just did that, but it's included just to be safe because time, and DNS, matter.
1. Click `Begin Installation`
1. We're not going to set a Root passphrase because CAPES will never, ever need it. Ever. Not setting a passphrase locks the Root account.
1. Create a user, but ensure that you toggle the `Make this user administrator` checkbox
1. Once the installation is done, click the `Reboot` button in the bottom right to...well...reboot
1. Login using the account you created during the Anaconda setup
1. Secure Ssh by removing username and password as an authentication option and don't allow the root account to log in at all
  - On your management system, create an ssh key pair
    ```
    ssh-keygen -t rsa #accept the defaults, but enter a passphrase for your keys
    ssh-copy-id your_capes_user@[ip of CAPES]
    ssh your_capes_user@[ip of CAPES]
    # You are now logged into the shell of CAPES
    sudo sed -i 's/\#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config #enter password for the CAPES user you created
    sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
    sudo systemctl restart sshd.service
    ```
## Get CAPES
Finally, here we go.

### CentOS 7.6
```
sudo yum -y install git
git clone https://github.com/capesstack/capes-docker.git
cd capes-docker
sudo sh deploy_capes.sh
```

### Build Process
The build is automated and the shell script will start the build of:
* Install Docker
* Install Docker containers for
- Mumble
- Rocketchat
- Gita
- TheHive
- Cortex
- Etherpad
- Cyberchef
- Nginx
- Mysql
- Mongodb
- Elasticsearch (for TheHive)
- Elasticsearch (for monitoring CAPES)
- Kibana (for monitoring CAPES)
- Heartbeat (for monitoring CAPES)
* Setup the CAPES landing page
* Make firewall changes

## Post Installation
While the CAPES deploy script attempts to get you up and running, there are a few things that need to be done after installation.

After the CAPES installation, you should be able to browse to `http://capes_hostname` (or `http://capes_IP` if you don't have DNS set up) to get to the CAPES landing page and start setting up services.

### Portainer
1. Browse to the Portainer UI and create your admin account
1. When it starts, click on the `Local` option

### Gitea
1. When you browse to your Gitea instance, you'll need to click on `Register`, set your database passphrase, your SSH Server Domain, your base URL, and disable Gravatar.
1. Set your database type to `MYSQL`, the host will be `capes-gitea-mysql:3306`, and the user will be `gitea` and the passphrase is in `~/capes_credentials.txt`
1. For the `SSH Server Domain`, use your host IP address (not the container IP)
1. For the `Gitea Base URL`, enter `http://[your_host_IP_address]:4000/`
1. Go to `Server and Third-Party Service Settings` and check the `Disable Gravatar` box
1. Click to setup Gitea
1. Click on `Login` to set up your administrative account. Pick a username, email address, and passphrase (note, you cannot use `admin`)
1. Click `Register Account`
1. Login in as the user you just created

### Etherpad
1. Browse to the Etherpad UI
1. I also recommend going to [capesIP]:5000/admin
1. Login with the admin account in `~/capes_credentials.txt` as `etherpad_admin_passphrase`
1. Click on `Plugins` and install `AdminPads`

### Rocketchat
1. Browse to the Rocketchat UI
1. Complete the `Admin Info`
1. Complete the `Organization Info`
1. Complete the `Server Info`
1. In the `Register Server` section, select `Keep standalone`

### TheHive
1. Browse to TheHive's UI
1. Click "Update Database" and create an administrative account

#### Upload Configuration
To get the custom fields with the templates, you'll need to upload the whole configuration file (which is recommended). After this configuration file is uploaded you can make any additional changes that you'd like. The below steps should be performed on your system, not CAPES:

1. Ensure you have [Python3](https://www.python.org/) installed
1. Log into TheHive as an administrator
1. Click on the `Admin` dropdown and select `Users`
1. Either create a new account with `admin` permissions or use an existing account, create and reveal the API key, copy this down
1. Collect the [capes-config.conf](thehive/capes-config.conf) file
```
git clone https://github.com/TheHive-Project/TheHive-Resources.git
cd TheHive-Resources/contrib/ManageConfig
sudo pip3 install requests
python3 submit_config.py -k <API key> -u http://CAPES-IP:9000 -c capes-config.conf
```

### Mumble
1. Download the client of your choosing from the [Mumble client page](https://www.mumble.com/mumble-download.php)
1. Install
1. There will be considerable menus to navigate, just accept the defaults unless you have a reason not to and need a custom deployment.
1. Start it up and connect to
  - Label: Whatever you want your channel to be called...maybe "CAPES" or something?
  - Address: CAPES server IP address
  - Port: 7000
  - Username: whatever you want
  - Password: this CAN be blank...but it shouldn't be **ahem**
  - Click "OK"
  - Select the channel you just created and click "Connect"
1. Right click on your name and select "Register". Once a user has created an account and Registered, you can add them to the `admin` role.
1. Click on the Globe and select the channel that you created and click "Edit"
1. For the username, use the `SuperUser` account with the passphrase from `~/capes_credentials.txt` as `mumble_passphrase` (the passphrase box will show up once you type `SuperUser`).
1. Right-click on main channel (likely "CAPES - Mumble Server") and select `Edit`
1. Go to the Groups tab
1. Select the `admin` role from the drop down
1. Type the user account you want to delegate admin functions to in the box
1. Click `Add` and then `Ok`
1. Click on the Globe and select the channel that you created and click "Edit"
1. Enter your username (not `SuperUser`) and your passphrase, and you can log in and perform administrative functions

I **strongly** recommend that you look at the `Build, Operate, Maintain` guides for these services before you get going. A few of the services launch a configuration pipeline that is hard to restart if you don't complete it the first time (I'm looking at you TheHive, Cortex, and Gitea).
