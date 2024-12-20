---
layout: page
---
# Wireguard Installation
# Create a Digital Ocean Account and Provision a Droplet
Create an account or log into Digital Ocean

Once inside, create a new project
- I titled mine `Wireguard Project`

Inside this new project, create a Droplet with the following specifications
- Nearest region to you
- Ubuntu 22.04 LTS
- Size: Basic
- Disk type: SSD, $6/mo option

Everything else is optional and doesn't need to be configured for this lab

Finally, confirm the creation of the Droplet. Once it is activated, log into the console as root

# Install Docker on the Droplet
## Setup Docker apt Repo
`sudo apt update`

`sudo apt install ca-certificates curl`
- Installs ca-certificates and curl packages

`sudo install -m 0755 -d /etc/apt/keyrings`
- More efficient way to create the /etc/apt/keyrings directory with 0755 permissions

`sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`
- Outputs GPG key to a new file called docker.asc in the keyrings folder

`sudo chmod a+r /etc/apt/keyrings/docker.asc`
- Enable read permissions for all

````bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
````
- Add repository to apt sources

`sudo apt-get update`
- Refresh the apt sources to include Docker repositories

## Install and Verify Docker Installation
`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-compose``
- Install latest Docker version

`sudo docker run hello-world`
- Download test Docker image and run in container
- Successful installation will print a confirmation message

# Install Wireguard
```
cd ~
mkdir Wireguard
cd Wireguard
```

- Create Wireguard directory to house docker-compose.yml file

`mkdir config`
- Create config directory for later use

`touch docker-compose.yml`
- Creates file to specify settings for Wireguard server
- Use a text editor such as `nano` or `gedit` and paste the following .yml configuration file below, which modifications as necessary

```
version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles      //Must modify
      - SERVERURL=134.209.69.145    //Must modify
      - SERVERPORT=51820
      - PEERS=PC,phone              //Must modify
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
	  - SYS_MODULE
	  - net.ipv4.conf.all.src_valid_mark=1
```
Modifications
- Choose an applicable Timezone (`TZ`) to run the Wireguard server
- The `SERVERURL` can be found by pasting the copied IP address of your DigitalOcean droplet
- Change the entities in `PEERS` as necessary to generate user config files for all declared users

# Wireguard Service Start
`docker-compose up -d`
- Start the Wireguard service
