---
title: "Host Your Own Gitlab Server "
date: 2024-03-19
draft: true
---
# Host Your Own Git Server with Docker

Have you ever wanted to host your own git server with a handy user interface? You can now do it in minutes using Docker on any server you want. Follow this guide to get started.

## Installation Guide

### Step 1: Install Docker

First, you need to install Docker on your server. Open a terminal and execute the following commands:

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker # Check if Docker is running successfully
sudo usermod -aG docker ${USER} # Add your user to the Docker group
```

### Step 2: Run the GitLab CE

Now, it's time to set up GitLab CE using Docker. GitLab CE (Community Edition) is a powerful and flexible tool for managing your Git repositories with a user-friendly interface. Perform the following steps in your terminal:

```bash
mkdir Gitlab
cd Gitlab
mkdir gitlab-config gitlab-logs gitlab-data # Create directories for GitLab configuration, logs, and data

# Run GitLab CE in a Docker container
docker run --detach \
--publish 1443:443 --publish 8080:80 --publish 1001:22 \
--name gitlab \
--restart always \
--volume gitlab-config:/etc/gitlab \
--volume gitlab-logs:/var/log/gitlab \
--volume gitlab-data:/var/opt/gitlab \
--shm-size 2gb \
gitlab/gitlab-ce:latest
```

### Step 3: Retrieve the Default Root User Password

After GitLab has started, you can retrieve the default password for the `root` user by executing the following command:

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```
> **Note:** The container may require up to 15 minutes to start up. Please allow some time for the initialization process to complete before attempting to access GitLab.

Congratulations! You now have your own Git server running on Docker. Navigate to your server's IP address or domain on the specified ports to start using GitLab.

## Automated Script

For convenience, here's a script that automates the above steps:

```bash
#!/bin/bash

# Update and install Docker
sudo apt update
sudo apt install docker.io -y

# Enable and start Docker service
sudo systemctl enable docker
sudo systemctl start docker

# Check Docker status (optional)
echo "Checking Docker status..."
sudo systemctl status docker --no-pager

# Add the current user to the Docker group
sudo usermod -aG docker ${USER}
echo "Added ${USER} to the Docker group."

# Create directories for GitLab
mkdir -p Gitlab/gitlab-config Gitlab/gitlab-logs Gitlab/gitlab-data

# Run GitLab CE in a Docker container
sudo docker run --detach \
  --publish 1443:443 --publish 8080:80 --publish 1001:22 \
  --name gitlab \
  --restart always \
  --volume $(pwd)/Gitlab/gitlab-config:/etc/gitlab \
  --volume $(pwd)/Gitlab/gitlab-logs:/var/log/gitlab \
  --volume $(pwd)/Gitlab/gitlab-data:/var/opt/gitlab \
  --shm-size 2gb \
  gitlab/gitlab-ce:latest

echo "GitLab is starting up. It might take a few minutes to be ready."

# Retrieve the default root password
echo "Retrieving the default root password..."
sudo docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```
