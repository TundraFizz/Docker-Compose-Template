
# Getting started

Run the first command below, and then the second command after the system reboots
```
bash <(curl -Ls https://tundrafizz.page.link/setup-1)
bash <(curl -Ls https://tundrafizz.page.link/setup-2)
```
Note: The name of the Docker stack that will be created is `tundra` and phpMyAdmin will used the domain name `db.tundrafizz.com`

# Adding a new service

1. Add the code to it's own directory `SERVICE_NAME`

2. Build the Docker image
```
docker build -t SERVICE_NAME SERVICE_NAME
```

3. Create a Docker service
```
docker service create --network tundra_default --name sample_app sample_app
```

4. Run this command
```
bash <(curl -Ls https://tundrafizz.page.link/setup-ssl)
```
This will do the following:
 * Prompt the user for SERVICE_NAME and DOMAIN_URL
 * Download a basic NGINX config file, and modify the file with the user's inputs
 * Restart NGINX
 * Run Let's Encrypt
 * Download an NGINX config file for SSL, and modify the file with the user's inputs
 * Restart NGINX

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
