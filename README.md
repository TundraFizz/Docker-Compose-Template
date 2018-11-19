
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

1. Build the Docker image
```
docker build -t "example_app" "example_app"
```

2. Create a Docker service
```
docker service create --network tundra_default --name "$SERVICE_NAME" "$SERVICE_NAME"
curl -Lso "nginx_conf.d/$SERVICE_NAME.conf" "https://tundrafizz.page.link/conf-basic"
sed -i "s/SERVICE_NAME/$SERVICE_NAME/g" "nginx_conf.d/$SERVICE_NAME.conf"
sed -i "s/DOMAIN_URL/$DOMAIN_URL/g" "nginx_conf.d/$SERVICE_NAME.conf"
```

3. Restart NGINX
```
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
docker build -t SERVICE_NAME SERVICE_NAME
```

3. Restart the Docker container
```
docker container restart $(docker container ls | grep SERVICE_NAME | grep -Eo '^[^ ]+')
```

########################################################################################################################

# OLD STUFF BELOW

# Adding a new service

1. Add the code to it's own directory `SERVICE_NAME` and then run the command below
```
bash <(curl -Ls https://tundrafizz.page.link/setup-ssl)
```
This will do the following:
 * Prompt the user for SERVICE_NAME and DOMAIN_URL
 * Build the Docker image
 * Create a Docker service
 * Download a basic NGINX config file, and modify the file with the user's inputs
 * Restart NGINX
 * Run Let's Encrypt
 * Download an NGINX config file for SSL, and modify the file with the user's inputs
 * Restart NGINX

# Updating an existing service

1. Replace the code with its updated version in its directory `SERVICE_NAME`

2. Build the new Docker image
```
docker build -t SERVICE_NAME SERVICE_NAME
```

3. Restart the Docker container
```
docker container restart $(docker container ls | grep SERVICE_NAME | grep -Eo '^[^ ]+')
```
