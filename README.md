# docker-plex
A docker container to run the latest version of Plex Media Server

## Installation

To download the docker container and execute it, simply run:

`sudo docker run -h your_host_name --name plex -d -v /route/to/your/media:/media -v /route/to/your/config:/config -t -i -p 32400:32400 odarriba/plex`

If you don't want to save your configuration outside the docker container, you can omit the `-v /route/to/your/config:/config` variable.

Now you should have a docker instance running `Plex Media Server`.

## First configuration

Due to some network security aspects of Docker, the container is going to be running in a separate subnet, **so you can't really access your Plex server for configuration** (to do it, the server should be in the same network).

**Solution:**, you must edit a file called `Preferences.xml` inside your configuration folder. On it you have to add an attribute to the tag `<Preference>`, following this structure: `allowedNetworks="192.168.0.0/255.255.255.0"` (with your own's network configuration).

After making the change you can restart the container and you should be able to access to your Plex Media Server using your web browser (`http://your_server_ip:32400/web`).

## Auto-discovering

Below this text you can see two different approaches to this problem. The secure one doesn't actually enable the auto discovery, but it's a good approach if you want to have a secure environment. On the other hand, you can use an insecure method that will work very well but it's not recommended because of it's insecurity.

### Secure method

**NOTE:** due to some problems inside Plex, it appears not to know it's local IP address inside Docker subnet, avoiding the local discovery to work. If you really want to use this feature, look below for the insecure method.

Avahi daemon is commonly used to help your computers to find the services provided by a server.

Avahi isn't built into this Docker image because, due to Docker's networking limitations, Avahi can't spread it's messages to announce the services out of the Docker virtual network.

**If you want to enable this feature, you can install Avahi daemon in your host** following this steps (Ubuntu version):

* Install `avahi-daemon`: run `sudo apt-get install avahi-daemon avahi-utils`
* Copy the file from `avahi/nsswitch.conf` to `/etc/nsswitch.conf`
* Copy the service description file from `avahi/plex.service` to `/etc/avahi/services/plex.service`
* Restart Avahi's daemon: `sudo /etc/init.d/avahi-daemon restart`

**But why you need to install this on your host and not in the container?** Because if you don't do it this way, the discovery message won't be able to reach your computers.

**What will I get with this approach?:** The service will be announced on the network, but you will have to login with your account to detect your server. Also, all the streaming you receive is going to be reduced as if you are in an external network.

### Insecure method

The method described in this section is **insecure and non-recommended**. But at the moment of writing this, Plex appears to be incompatible with a double NAT configuration like the one Docker has.

But, if you are brave like a viking, you can use it at your own risk.

The idea is to attach the Docker container to the network stack of your host. To do it, you can simply change the `-h your_host_name` to `--net=host` and everything should work.

**Disadvantages (for disclaimers):** connecting your container to your host network stack means that the container can listen to everything you receive. It will use as many ports as it needs but it's completely insecure and it isn't the correct approach for an isolated service.

## Auto start the service

This repository contains a script to run the container at boot time **in Ubuntu-based distros**, but **it requires that the container have been run manually at least one time**.

To install the script, just execute `sudo cp config/plexmediaserver.conf /etc/init/plexmediaserver.conf`.

* To start the service: `sudo service plexmediaserver start`
* To stop the service: `sudo service plexmediaserver stop`

**Note:** when you stop the service, the container keeps running. Yo must execute `sudo docker stop plexmediaserver`in order to stop the server.

## Contributors

* Óscar de Arriba (odarriba@gmail.com)