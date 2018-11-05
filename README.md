
# Getting started

1. Run all of these commands
```
# Disable SELinux permanently
sudo setenforce 0
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

# Install nano and wget
sudo yum -y install nano wget

# Download the Community Edition of Docker
sudo yum -y install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum -y install docker-ce

# Enable docker to automatically start on boot
sudo systemctl enable docker
sudo systemctl start docker

# Add Google's DNS to the Docker daemon; this allows Docker containers to connect to the internet
echo '{"dns": ["8.8.8.8", "8.8.4.4"]}' | sudo tee --append /etc/docker/daemon.json > /dev/null

# Add the user to the docker group and then reboot to apply the changes
sudo usermod -aG docker $USER
sudo reboot
```

2. After rebooting, run these to initialize Docker
```
docker swarm init
docker stack deploy -c docker-compose.yml tundra
```

# Adding a new service

1. Add the code to it's own directory `SERVICE_NAME`

2. Build the Docker image
```
docker build -t SERVICE_NAME SERVICE_NAME
```

3. Add this to services in `docker-compose.yml`

```
  SERVICE_NAME:
    image: SERVICE_NAME
```

4. Create a basic .conf in `nginx_conf.d`, replacing `SERVICE_NAME` and `DOMAIN_URL`
```
curl -Lso nginx_conf.d/SERVICE_NAME.conf https://tundrafizz.page.link/conf-basic
sed -i 's/SERVICE_NAME/111111111111/g' nginx_conf.d/SERVICE_NAME.conf
sed -i 's/DOMAIN_URL/2222222222/g' nginx_conf.d/SERVICE_NAME.conf
```

5. Update the Docker stack
```
docker stack deploy -c docker-compose.yml tundra
```

6. Restart NGINX
```
docker container restart $(docker container ls | grep nginx | grep -Eo '^[^ ]+')
```

7. Once the service is accessible on DOMAIN_URL, create an SSL certificate with Let's Encrypt

```
docker run -it --rm --name certbot                      \
-v tundra_ssl:/etc/letsencrypt                          \
-v tundra_ssl_challenge:/ssl_challenge                  \
certbot/certbot certonly                                \
--register-unsafely-without-email --webroot --agree-tos \
-w /ssl_challenge -d DOMAIN_URL
```

8. Update the .conf file in `nginx_conf.d` for SSL, replacing `SERVICE_NAME` and `DOMAIN_URL`
```
curl -Lso nginx_conf.d/SERVICE_NAME.conf https://tundrafizz.page.link/conf-ssl
sed -i 's/SERVICE_NAME/111111111111/g' nginx_conf.d/SERVICE_NAME.conf
sed -i 's/DOMAIN_URL/2222222222/g' nginx_conf.d/SERVICE_NAME.conf
```

9. Restart NGINX
```
docker container restart $(docker container ls | grep nginx | grep -Eo '^[^ ]+')
```

# How to update a service

1. Replace the code with its updated version in its directory `SERVICE_NAME`

2. Build the new Docker image
```
docker build -t SERVICE_NAME SERVICE_NAME
```

3. Restart the Docker container
```
docker container restart $(docker container ls | grep SERVICE_NAME | grep -Eo '^[^ ]+')
```
