# SysReptor Installation Guide
This is an inofficial guide how to install [SysReptor](https://docs.sysreptor.com/) without building it yourself.

Make sure you have Docker and the [Compose (v2) plugin](https://docs.docker.com/compose/migrate/) installed (no podman or compose v1).

## Create volumes
```bash
docker volume create sysreptor-db-data
docker volume create sysreptor-app-data
```

## Create app.env with secrets
```bash
printf "SECRET_KEY=\"$(openssl rand -base64 64 | tr -d '\n=')\"\n" >> app.env
KEY_ID=$(uuidgen) && printf "ENCRYPTION_KEYS=[{\"id\": \"${KEY_ID}\", \"key\": \"$(openssl rand -base64 32)\", \"cipher\": \"AES-GCM\", \"revoked\": false}]\nDEFAULT_ENCRYPTION_KEY_ID=\"${KEY_ID}\"\n" >> app.env
```

If you use Professional, also add your license key:
```bash
echo "LICENSE=<key>" >> app.env
```

## Create docker compose files

Download the docker-compose.yml from [SysReptor project from GitHub](https://github.com/Syslifters/sysreptor):
```bash
curl -O https://raw.githubusercontent.com/Syslifters/sysreptor/main/deploy/docker-compose.yml
```

Create a docker-compose.override.yml to prevent building the image:

```yaml
version: '3.9'
name: sysreptor

services:
  app:
    image: cocoa1344/sysreptor
```

If you are professional user, use this file contents instead:
```yaml
version: '3.9'
name: sysreptor

services:
  app:
    image: cocoa1344/sysreptor
  languagetool:
    image: cocoa1344/sysreptor-languagetool
```

## Start Docker containers

Your directory stucture is now:
```
sysreptor/
├── app.env
├── docker-compose.yml
└── docker-compose.override.yml
```

Start your Docker containers:

```bash
docker compose up -d
```

The images are now pulled from [Docker Hub](https://hub.docker.com/r/cocoa1344/sysreptor) and started.

## Create initial user
Run command to create initial user. Username is “reptor” and the password is entered interactively.

```bash
username=reptor
docker compose exec app python3 manage.py createsuperuser --username "$username"
```

## Add demo data

Add demo projects, designs, finding templates:

```bash
# Projects
url="https://docs.sysreptor.com/assets/demo-projects.tar.gz"
curl -s "$url" | docker compose exec --no-TTY app python3 manage.py importdemodata --type=project --add-member="$username"

# Designs
url="https://docs.sysreptor.com/assets/demo-designs.tar.gz"
curl -s "$url" | docker compose exec --no-TTY app python3 manage.py importdemodata --type=design

# Finding templates
url="https://docs.sysreptor.com/assets/demo-templates.tar.gz"
curl -s "$url" | docker compose exec --no-TTY app python3 manage.py importdemodata --type=template
```
