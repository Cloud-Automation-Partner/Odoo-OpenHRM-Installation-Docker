# Odoo Docker OpenHRM Integration

Welcome to the Odoo Docker OpenHRM Integration repository. This comprehensive guide is designed to facilitate the setup of Odoo with Docker, incorporating the powerful features of OpenHRM. Follow the structured steps below for a seamless integration and deployment process.

## Cloning the Repository

First, clone this repository to your local environment to access the necessary Docker configurations and scripts.

**Command:**
```bash
git clone https://github.com/Cloud-Automation-Partner/Odoo-OpenHRM-Installation-Docker.git
```
## Building the Odoo Docker Image

Using the Dockerfile provided within this repository, build the custom Odoo Docker image that is tailored for this setup and as of I am using the Odoo 16 to setup so switch the directory first.

**Instructions:**
Navigate to the folder containing the Dockerfile and run:

```bash
cd Odoo-docker/16.0
docker build -t Odoo-OpenHRM:latest .
docker images #Verify the image creation
```
## Integrating OpenHRM  

For effective integration of OpenHRM, manually copy the OpenHRM source code into the designated Docker volume as specified in your Docker Compose configuration.

Run below commands:

```bash
mkdir addons
cp -r OpenHRMS ./addons
mkdir config
cp Odoo-docker/16.0/odoo.conf ./config
```

## Configuring Docker Compose with Postgres

Customize your Docker Compose setup by editing the `docker-compose.yml` file. This file orchestrates the configuration of your Odoo and PostgreSQL services.

- Ensure the PostgreSQL service is correctly configured with the necessary environment variables.
- Adjust the Odoo service settings for optimal performance and compatibility with OpenHRM.

```yml
version: '3.1'
services:
  web:
    image: odoo:latest
    depends_on:
      - db
    ports:
      - "8069:8069"
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./config:/etc/odoo
      - ./addons:/mnt/extra-addons
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=myodoo
  db:
    image: postgres:15
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_PASSWORD=myodoo
      - POSTGRES_USER=odoo
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - odoo-db-data:/var/lib/postgresql/data/pgdata
volumes:
  odoo-web-data:
  odoo-db-data:
```

## Spinning Up the Containers

Launch the Docker containers using Docker Compose. This command will initiate and run both the PostgreSQL database and the Odoo application.

**Command:**
```bash
docker-compose up -d
```
## Additional Notes

This setup allows for customization and extension:

- Modify the environment variables and other configurations in the `docker-compose.yml` file according to your project's specific requirements.
- Explore additional Docker and Odoo configurations to further tailor this setup.


## Accessing Odoo

Once the Docker containers are operational, access the Odoo application using the following URL:

- **URL:** [http://localhost:8069](http://localhost:8069)

## Adding Nginx Web server

Now that your Odoo server is accessible on the specified port, let's enhance security and flexibility by setting up Nginx as a reverse proxy.

### Install Nginx

```bash
sudo apt update
sudo apt install nginx
systemctl status nginx
```
### Configure Nginx

Create an Nginx configuration file for Odoo. Replace your_domain with your actual domain.
```bash
vim /etc/nginx/nginx.conf
```
Now copy the below nginx configs over there

```nginx
server {
    listen 80;
    server_name your_domain;
    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name your_domain;

    ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem; #Copy your ssl path running below certbot commands
    ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem; #Copy your ssl path running below certbot commands


    location / {
        location / {
        proxy_pass http://127.0.0.1:8069;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    }
    }   }

```
After saving the above configs, Test the Nginx configuration:
```bash
nginx -t
```
Restart Nginx:
```bash
sudo systemctl restart nginx
```
## Adding SSL Certificates with Certbot
Enhance security further by securing your Odoo instance with SSL certificates using Certbot.

### Install Certbot
```bash
sudo apt update
sudo apt install certbot
```
### Obtain SSL Certificates
Run Certbot and follow the interactive prompts. Replace your_domain with your actual domain.
```bash
sudo certbot certonly --standalone -d your_domain.com
```
This will create SSL certs for your domain and return you the location where certs are stored copy that path in your nignx config

Restart Nginx:
```bash
sudo systemctl restart nginx
```
With Nginx configured as a reverse proxy and SSL certificates added, your Odoo instance is now secure and accessible over HTTPS.

We wish you a successful implementation and a productive experience with your Odoo and OpenHRM integrated environment.

Happy coding! 🚀🎉

