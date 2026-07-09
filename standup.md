# Installing Docker and Running Jenkins in Docker on Ubuntu

## 1. Update the system

``` bash
sudo apt update
sudo apt upgrade -y
```

Updates package lists and upgrades installed packages.

## 2. Install Docker

### Install prerequisites

``` bash
sudo apt install -y ca-certificates curl gnupg
```

### Add Docker's GPG key

``` bash
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

### Add the Docker repository

``` bash
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker

``` bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 3. Start Docker

``` bash
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

Enable Docker at boot, start it now, and verify it is running.

## 4. Run Docker without sudo

``` bash
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
```

Adds your user to the Docker group and verifies the installation.

# Running Jenkins

## Create a persistent volume

``` bash
docker volume create jenkins_home
docker volume ls
```

Stores Jenkins configuration and jobs.

## Start Jenkins

``` bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk21
```

## Verify

``` bash
docker ps
```

## Access Jenkins

Open:

-   http://localhost:8080
-   http://`<server-ip>`{=html}:8080

## Initial admin password

``` bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Paste the password into the Jenkins setup page.

Choose **Install Suggested Plugins** and create the first administrator
account.

# Useful Commands

``` bash
docker ps
docker ps -a
docker stop jenkins
docker start jenkins
docker restart jenkins
docker logs -f jenkins
docker exec -it jenkins bash
docker rm -f jenkins
```

# Optional: Allow Jenkins to Build Docker Images

``` bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/bin/docker:/usr/bin/docker \
  jenkins/jenkins:lts-jdk21
```
