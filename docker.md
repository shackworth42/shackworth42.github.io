# Docker and Gotify Setup

## Install Docker and Docker Compose
1. Update your system to ensure compliance with docker packages "sudo pacman -Syu"
2. Install docker "sudo pacman -S docker"
3. Install docker compose "sudo pacman -S docker-compose"
4. Load required modules "sudo modprobe overlay" && "sudo modprobe br_netfilter"
5. Start Docker "sudo systemctl enable --now docker.service"

## Gotify Setup
1. Create and cd into a docker directory "mkdir ~/gotify && cd ~/gotify"
2. Create docker-compose.yml file "vim docker-compose.yml"
3. Painfully write a docker-compose.yml file and have it fail 6 times because you indent like a 7 year old:

version: '3'

services:
  gotify:
    image: gotify/server:latest
    container_name: gotify
    ports:
      - "8080:80"
    environment:
      - GOTIFY_DEFAULTUSER_PASS=admin
    volumes:
      - ./gotify_data:/app/data
    restart: unless-stopped

4. Start Gotify "sudo docker-compose up -d"

## Accessing Docker Container
1. Find your container's IP address "ip addr show"
2. Paste address in a browser window and login with default login and password