# Docker & WireGuard Setup on Ubuntu

## Create the DigitalOcean Droplet and Log In

1. Log into DigitalOcean in your browser.
2. Create a new Droplet:
    1. Image: Ubuntu 24.04 LTS
3. Log into console via DigitalOcean website as root


## Install Docker and Docker Compose

1. Update packages:  
   apt update
2. Install Docker prerequisites:  
   apt install ca-certificates curl -y
3. Create a keyring directory:  
   install -m 0755 -d /etc/apt/keyrings
4. Download Docker’s GPG key:  
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

### Add Docker Repo

5. Add the Docker repo

   tee /etc/apt/sources.list.d/docker.sources << 'EOF'  
   Types: deb  
   URIs: https://download.docker.com/linux/ubuntu  
   Suites: /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}" 
   Components: stable  
   Signed-By: /etc/apt/keyrings/docker.asc  
   EOF


### Install and start Docker

6. Install Docker Engine, CLI, containerd, Buildx, and Compose plugin:  
    apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
7. Start Docker:  
    systemctl start docker



## WireGuard Setup in Docker

### Create WireGuard directory and compose file

1. Create a directory and cd into it:  
   mkdir -p /opt/wireguard && cd /opt/wireguard
2. Create docker-compose.yml:  

   version: "3.8"  
     
   services:  
     wireguard:  
       image: lscr.io/linuxserver/wireguard:latest  
       container_name: wireguard  
       cap_add:  
         - NET_ADMIN  
         - SYS_MODULE  
       environment:  
         - PUID=0  
         - PGID=0  
         - TZ=Etc/UTC  
         - SERVERURL=165.227.60.177  
         - SERVERPORT=51820  
         - PEERS=phone,pc  
         - PEERDNS=1.1.1.1  
         - INTERNAL_SUBNET=10.13.13.0  
         - ALLOWEDIPS=0.0.0.0/0  
         - LOG_CONFS=true  
       volumes:  
         - ./config:/config  
         - /lib/modules:/lib/modules  
       ports:  
         - 51820:51820/udp  
       sysctls:  
         - net.ipv4.conf.all.src_valid_mark=1  
       restart: unless-stopped  
   EOF


### Start WireGuard

3. Start the WireGuard container stack:  
   cd /opt/wireguard && docker compose up -d


## Configure Phone Tunnel (peer: phone)

1. On your phone, install the WireGuard app from the App Store / Play Store.
2. On the droplet, display the QR code for the phone peer:  
    1. docker exec -it wireguard /app/show-peer phone
3. In the WireGuard app on your phone:
    1. Tap Add tunnel
    2. Choose Create from QR code
    3. Point the camera at the QR code in the terminal
    4. Name the tunnel
4. Toggle the tunnel on to connect.


## Configure Windows PC Tunnel (peer: pc)

1. On your Windows PC, download and install WireGuard from:  
   https://www.wireguard.com/install/
2. Open the WireGuard app on Windows.
3. On the droplet, display the pc peer config:  
   1. cat config/peer_pc/peer_pc.conf
4. Select and copy all of the config text from [Interface] through [Peer].
5. In WireGuard on Windows:
    1. Click Add Tunnel → Add empty tunnel
    2. Delete the placeholder content
    3. Paste the config text you copied from peer_pc.conf
    4. Name the tunnel
    5. Click Save
6. Click Activate to bring the tunnel up.