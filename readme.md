# Dann's Home Media Server (Public)

A Docker-Compose file that lifts up (almost) all the services needed to run a handy home server. This repo is to serve as a compliment to my (upcoming) blog post on [dannb.org](https://dannb.org).

## What's included

The following services are part of the docker-compose document:

* Traefik1 reverse proxy
* OAuth Authentication
* Plex
* Apache Web Server
* Portainer
* Organizr
* Diskover (w/ES, redis)
* PiGallery2
* Ubooquity Comic Server
* Handbrake
* Watchtower
* Netdata
* Glances
* Dokuwiki
* Tiddlywiki
* Homebridge
* Tautulli

### How it all works together

**Organizr** is the home base. It allows you to access all these services from a single dashboard. Docker visual management is in **Portainer**. **Netdata** and **Glances** let you see detailed analytics about the server. **Watchtower** keeps things up-to-date.

**Plex** streams movies and TV shows, and **Tautulli** provides viewing stats. **Ubooquity** is a comic server, allowing you to keep .cbr/.cbz/.pdf files on your NAS and view them on your comptuer or with a mobile app such as [Chunky](http://chunkyreader.com/). **PiGallery2** is a simple directory-based image gallery for viewing all your pictures.

Finally, **Dokuwiki** is a traditional wiki and **Tiddlywiki** is a non-linear wiki. You can use these to create an knowledge database.

### Managing it all

You can edit the docker-compose file here:

```
nano ~/docker/docker-compose.yml
```

Once you're done editing that file, save it and run:

```
docker-compose -f ~/docker/docker-compose.yml up -d
```

### Issues I've had

When you set up the new Linux server, you should create a new user and then log in as that user. That way you're not doing everything as root. Just make sure that new user has sudo access. :headdesk:

Also, directory permissions! Make sure all the folders you're using, and the folders inside those, have the following permissions:

```
sudo chmod -R 0775 /mnt/nas/Movies
```

## Migrating to a new server

### 1) Install Debian

### 2) connect to Internet

View ip settings using `$ ip a`. Note if eth0 has been renamed (maybe to eno1?). Then edit `/etc/network/interfaces` to add:

```
# The primary network interface
allow-hotplug eno1`
iface eno1 inet dhcp
```

Then use command `$ dhclient -v eno1` to get an IP address from the router. Make note of this local IP. It will probably be something like 192.168.1.[2-99].

### 3) Install Docker/Docker-compose

* [Install Docker](https://docs.docker.com/install/linux/docker-ce/debian/#install-docker-ce)
* [Install Docker-Compose](https://docs.docker.com/compose/install/)

### 4) Login to docker

`docker login --username=[username]`

_Replace [username] with your username, sans brackets._

### 5) Install nfs-common

This will let you connect to the Synology NAS via NFS.

`sudo apt-get update`
`sudo apt-get install nfs-common`

### 6) Allow new server to connect to NAS

Log into your Synology NAS UI. Go to Control Panel > Shared Folder and select the Media volume and click Edit. Go to the NFS Permissions tab and enter the local IP of the new server.

### 7) Copy over `~/docker/`

**TKTK:** Fix this with Step 9.

_You only need to do this step if you've backed up an old server to your NAS, and want to move everything to a new server._

`rsync -aPuHxv dannberg@192.168.1.70:/home/dannberg/docker /home/dannberg`

| Option | Function |
|---|---|
| `-a` | archive mode |
| `-P` | preserve permissions |
| -u | skip files that are newer on the receiver |
| -H | output numbers in a human-readable format |
| -x | don't cross filesystem boundaries |
| -v | verbose |

**Note:** when using rsync on a directory, the actual directory (not just the contents) will be moved to the destination location.

### 8) Copy over env variables

Shh, these are your passwords. Whenever you see a ${UPPERCASE_WORD} in the config file, that's a variable set in the `/etc/environment` file.

`sudo nano /etc/environment`

### 9) Create the traefik docker network

`docker network create traefik_proxy`

### Get things ready for Docker

First we're creating the docker directory and giving it the correct permissions.

`mkdir ~/docker`
`sudo apt-get install acl`
`sudo setfacl -Rdm g:docker:rwx ~/docker`
`sudo chmod -R 775 ~/docker`

Next we need to get things ready for Diskover. Elasticsearch doesn't set directory permissions correctly, so we'll need to do that first.

`mkdir ~/docker/elasticsearch`
`sudo chmod -R 777 ~/docker/elasticsearch`

Then we need to increase memory map areas. The first command changes this in a live environment. The second changes the machine's settings.

`sudo sysctl -w vm.max_map_count=262144`
`grep vm.max_map_count /etc/sysctl.conf`

### 10) Run Docker Compose file

`docker-compose -f ~/docker/docker-compose.yml up -d`

### 11) Log into services to re-set up any passwords

### 12) Update home router to forward ports 80 (http) and 443 (https) to the new server's IP

## Reference

* [SmartHomeBeginner - Basic](https://www.smarthomebeginner.com/docker-home-media-server-2018-basic/) - Tutorial I largely used
* [His docker-compose file](https://github.com/htpcBeginner/AtoMiC-ToolKit-Docker/blob/master/docker-compose-basic.yml)
* [CloudBox Wiki](https://github.com/Cloudbox/Cloudbox/wiki) - Ansible playbook that builds all this out automatically. Great reference.
* [Diskover setup tutorial](https://engineerworkshop.com/2020/02/04/how-to-monitor-disk-usage-and-growth-with-diskover/)

# To Do
- [ ] Set up rsync on new server
