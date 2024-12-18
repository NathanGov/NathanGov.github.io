---
layout: page
---
# Docker Installation
Directions from: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

# Setup Docker apt Repo
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

# Install and Verify Docker Installation
`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`
- Install latest Docker version

`sudo docker run hello-world`
- Download test Docker image and run in container
- Successful installation will print a confirmation message

# Install Pi-hole (Network blocking tool)
`cd ~`
`mkdir Pihole`
`cd Pihole`
- Create Pi-hole directory to house docker-compose.yml file

Navigate to [https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker](https://github.com/pi-hole/docker-pi-hole/#running-pi-hole-docker)
- Paste the given docker-compose.yml code block into a new folder in the Pi-hole directory called `docker-compose.yml` (can use a different filename but must specify with `-f`)

## Enable Port 53 Listening For Pi-hole
`sudo sed -r -i.orig 's/#?DNSStubListener=yes/DNSStubListener=no/g' /etc/systemd/resolved.conf`
- Enable and set DNSStub listener to no so that Pihole can listen on port 53

`sudo sh -c 'rm /etc/resolv.conf && ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf'`
- Remove /etc/resolv.conf and create a symbolic link from /etc/resolv.conf to /runsystemd/resolv.conf, allowing DNS resolution to occur dynamically

`sudo systemctl restart systemd-resolved`
- Restart systemd-resolved for changes to take effect

`docker compose up -d`
- Start the Pi-hole service, without service logs appearing in terminal
- Docker compose command must be run from the directory that houses the docker-compose.yml file

\* At this point, Pihole is running, and can be accessed from the internet at the IP address of the VM. Congrats!
