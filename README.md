## 🚀 Evolution API + Redis + PostgreSQL + Nginx + Cloudflare Setup

This guide explains how to set up **Evolution API** with **Redis**, **PostgreSQL**, and **Nginx** using Docker Compose, and how to attach a **domain via Cloudflare** for secure public access.

---

## 1️⃣ Install Docker on Linux

Run:

```bash
# Update system
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update



```
Install the Docker packages.
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Enable & start
sudo systemctl enable docker
sudo systemctl start docker

# Verify
docker --version
```

### Install Docker Compose plugin:
```bash
sudo apt install -y docker-compose-plugin
docker compose version
```

---

##  2️⃣ Environment File

Create a `.env` file in the project root:

```env
# Database settings
POSTGRES_DB=database
POSTGRES_USER=user
POSTGRES_PASSWORD=*Beejay123qweasd

# API Key
AUTHENTICATION_API_KEY='KEY'

# Redis cache
CACHE_REDIS_ENABLED=true
CACHE_REDIS_URI=redis://redis:6379/6
CACHE_REDIS_PREFIX_KEY=evolution
CACHE_REDIS_SAVE_INSTANCES=false
CACHE_LOCAL_ENABLED=false

# Database provider
ABASE_ENABLED=true
DATABASE_PROVIDER=postgresql
DATABASE_CONNECTION_URI=postgresql://user:*Beejay123qweasd@db:5432/database?schema=public
DATABASE_CONNECTION_CLIENT_NAME=evolution_exchange

# Data persistence
DATABASE_SAVE_DATA_INSTANCE=true
DATABASE_SAVE_DATA_NEW_MESSAGE=true
DATABASE_SAVE_MESSAGE_UPDATE=true
DATABASE_SAVE_DATA_CONTACTS=true
DATABASE_SAVE_DATA_CHATS=true
DATABASE_SAVE_DATA_LABELS=true
DATABASE_SAVE_DATA_HISTORIC=true
```

---


## 3️⃣  Docker Compose File

Create **`docker-compose.yml`** in the project root:

```yaml

services:

   db:
      image: postgres:15
      container_name: postgres_db
      restart: always
      environment:
         POSTGRES_DB: ${POSTGRES_DB}
         POSTGRES_USER: ${POSTGRES_USER}
         POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      ports:
         - "5432:5432"   # optional: expose only if you need host access
      volumes:
         - postgres_data:/var/lib/postgresql/data
      networks:
         - shared_network

   redis:
      image: redis:latest
      container_name: redis
      command: >
         redis-server --port 6379 --appendonly yes
      volumes:
         - redis_data:/data
      networks:
         - shared_network

   api:
      container_name: evolution_api
      image: atendai/evolution-api:v1.8.7
      restart: always
      env_file:
         - .env
      volumes:
         - evolution_instances:/evolution/instances
         - ./ui-dist:/usr/share/nginx/html:ro
      depends_on:
         - db
         - redis
      networks:
         - shared_network
   nginx:
    container_name: nginx
    build:
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "80:80"
    env_file:
      - .env
    depends_on:
      - api
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - static_data:/static/
    networks:
      -  shared_network
networks:
   shared_network:
volumes:
   postgres_data:
   redis_data:
   evolution_instances:
   static_data:
```

## 4️⃣ Nginx (HTTP-only) Config
Server name you can change with your domain or ip address
```bash

    server {
        listen 80;
        server_name  ${SERVER_NAME}  ;


        client_max_body_size 4G;
        location /static/ {
                alias /static/;
            }

        location / {
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header Host $http_host;
            proxy_redirect off;
            if (!-f $request_filename) {
                proxy_pass http://api:8080;
                break;
            }
        }
    }



```




### ️5️⃣ Run Everything

Start the services:

```bash
docker compose up -d
```

Check logs:

```bash
docker compose logs -f
```



---

## ✅ Summary

- Installed Docker & Docker Compose
- Created `docker-compose.yml` with PostgreSQL, Redis, Evolution API, Nginx
- Configured `.env` file



