
# Getting started

Run the first command below, and then the second command after the system reboots
```
bash <(curl -Ls https://tundrafizz.page.link/setup-1)
bash <(curl -Ls https://tundrafizz.page.link/setup-2)
```

Modify phpmyadmin-basic.conf and restart NGINX so that it gets the new config domain
```
DOMAIN_URL="db.example.com"
sed -i "s/DOMAIN_URL/$DOMAIN_URL/g" "nginx_conf.d/phpmyadmin-basic.conf"
docker container restart "$(docker container ls | grep nginx | grep -Eo '^[^ ]+')"
```

Create an SSL certificate for phpMyAdmin with Let's Encrypt
```
docker run -it --rm --name certbot                      \
-v tundra_ssl:/etc/letsencrypt                          \
-v tundra_ssl_challenge:/ssl_challenge                  \
certbot/certbot certonly                                \
--register-unsafely-without-email --webroot --agree-tos \
-w /ssl_challenge -d "$DOMAIN_URL"
```

Update the NGINX config file for phpMyAdmin and restart NGINX
```
rm nginx_conf.d/phpmyadmin-basic.conf
mv nginx_conf.d/phpmyadmin-ssl.conf.new nginx_conf.d/phpmyadmin.conf
sed -i "s/DOMAIN_URL/$DOMAIN_URL/g" "nginx_conf.d/phpmyadmin.conf"
docker container restart "$(docker container ls | grep nginx | grep -Eo '^[^ ]+')"
```

# Adding a new service

1. Set a service name and domain url
```
SERVICE_NAME="example_app"
DOMAIN_URL="example.com"
```

2. Build the Docker image
```
docker build -t "$SERVICE_NAME" "$SERVICE_NAME" --build-arg MODE=dev
docker build -t "$SERVICE_NAME" "$SERVICE_NAME" --build-arg MODE=prod
```

3. Create a Docker service and the NGINX config file, and then restart NGINX
```
docker service create --network tundra_default --name "$SERVICE_NAME" "$SERVICE_NAME"
curl -Lso "nginx_conf.d/$SERVICE_NAME.conf" "https://tundrafizz.page.link/conf-basic"
sed -i "s/SERVICE_NAME/$SERVICE_NAME/g" "nginx_conf.d/$SERVICE_NAME.conf"
sed -i "s/DOMAIN_URL/$DOMAIN_URL/g" "nginx_conf.d/$SERVICE_NAME.conf"
docker container restart "$(docker container ls | grep nginx | grep -Eo '^[^ ]+')"
```

4. Once the service is accessible on DOMAIN_URL, create an SSL certificate with Let's Encrypt, modify the NGINX config files, and restart NGINX
```
docker run -it --rm --name certbot                      \
-v tundra_ssl:/etc/letsencrypt                          \
-v tundra_ssl_challenge:/ssl_challenge                  \
certbot/certbot certonly                                \
--register-unsafely-without-email --webroot --agree-tos \
-w /ssl_challenge -d "$DOMAIN_URL"

curl -Lso "nginx_conf.d/$SERVICE_NAME.conf" "https://tundrafizz.page.link/conf-ssl"
sed -i "s/SERVICE_NAME/$SERVICE_NAME/g" "nginx_conf.d/$SERVICE_NAME.conf"
sed -i "s/DOMAIN_URL/$DOMAIN_URL/g" "nginx_conf.d/$SERVICE_NAME.conf"
docker container restart "$(docker container ls | grep nginx | grep -Eo '^[^ ]+')"
```

# Updating an existing service

1. Replace the code with its updated version in its directory `SERVICE_NAME`

2. Build the new Docker image
```
docker build -t "$SERVICE_NAME" "$SERVICE_NAME" --build-arg MODE=dev
docker build -t "$SERVICE_NAME" "$SERVICE_NAME" --build-arg MODE=prod
```

3. Restart the Docker container
```
docker container stop $(docker container ls | grep "$SERVICE_NAME" | grep -Eo '^[^ ]+')
```

# Other Notes

#### Images
| Command                                   | Description                                 |
| ----------------------------------------- | ------------------------------------------- |
| `docker images`                           | List all available images                   |
| `docker build -t image-name .`            | Create image from source code               |
| `docker save image-name > image-name.tar` | Save an image to a .tar                     |
| `docker load -i image-name.tar`           | Load an image from a .tar                   |
| `docker run -p 80:9001 image-name`        | Run image in terminal (useful for testing)  |
| `docker tag image-name repo:tag`          | Set an image's repository and tag           |
| `docker rmi IMAGE_ID`                     | Delete a particular image                   |
| `docker rmi -f $(docker images -q)`       | Delete all images                           |
| `docker login`                            | You need to login before pushing            |
| `docker push tundrafizz/image-name`       | Push an image to Docker's online repository |

#### Containers
| Command                         | Description                           |
| ------------------------------- | ------------------------------------- |
| `docker container ls`           | List currently running containers     |
| `docker container ls -aq`       | List all containers, only showing IDs |
| `docker restart CONTAINER_ID`   | Restart a container                   |
| `docker stop $(docker ps -aq)`  | Stop all containers                   |
| `docker rm -f $(docker ps -aq)` | Delete all containers                 |

#### Swarms
| Command                           | Description                                |
| --------------------------------- | ------------------------------------------ |
| `docker swarm init`               | Initialize a Docker swarm                  |
| `docker swarm join TOKEN`         | Join a Docker swarm                        |
| `docker swarm join-token worker`  | Display the token for joining as a worker  |
| `docker swarm join-token manager` | Display the token for joining as a manager |

#### Services
| Command                                             | Description                               |
| --------------------------------------------------- | ----------------------------------------- |
| `docker stack deploy -c docker-compose.yml sample`  | Create services from a stack/compose file |
| `docker node ls`                                    | List all workers/managers in the swarm    |
| `docker node ps $(docker node ls -q)`               | List all tasks across all nodes           |
| `docker service ls`                                 | List all services in the swarm            |
| `docker service ls -q`                              | List all services in the swarm (only IDs) |
| `docker service ps NAME`                            | Check detailed status of a service        |
| `docker service rm ID`                              | Remove a particular service               |
| `docker service rm $(docker service ls -q)`         | Remove all services                       |

#### Misc
| Command                                                          | Description                                |
| ---------------------------------------------------------------- | ------------------------------------------ |
| `docker service logs SERVICE_NAME -f`                            | Displays logs of a given service           |
| `find /var/lib/docker/containers/ -type f -name "*.log" -delete` | Deletes all log files                      |
| `docker exec -it CONTAINER_ID bash`                              | SSH into a container                       |
| `docker cp file.ext CONTAINER_ID:/path/to/file.ext`              | Copy a file from the host into a container |
| `docker cp CONTAINER_ID:/path/to/file.ext .`                     | Copy a file from a container into the host |
| `docker run --name=sample -d tundrafizz/sample-app`              | Create a container from image manually     |

#### Miscellaneous

SSH into a container with the name "nginx"
`docker exec -it $(docker container ls | grep nginx | grep -Eo '^[^ ]+') bash`

Perform the command "nginx -s reload" in the container with the name "nginx"
`docker exec -it $(docker container ls | grep nginx | grep -Eo '^[^ ]+') nginx -s reload`

`sudo yum -y install epel-release`
`sudo curl -sL https://rpm.nodesource.com/setup_8.x | sudo bash -`
`sudo yum -y install nodejs`
`sudo npm i -g nodemon`

`docker run -it -d -p 8080:8080 -e HOST=54.202.110.238 -v /var/run/docker.sock:/var/run/docker.sock docker.io/dockersamples/visualizer:latest`

