# Nextcloud in docker

**Documentation in progress**

Docker compose recipe to run your nextcloud exposed to web behind traefik proxy. This is my current configuration but i removed all sensitive informations.

## Prerequisites

- Yow need have server with installed **Docker** and **docker-compose**
- Public static IP address. Fastest way is run this in cloud. I'm using [Hetzner](https://hetzner.cloud/?ref=5vMMq9U8b2cM)
- You will need to have your own domain
- For now traefik trying to get SSL certificates from **Cloudflare**. Otherwise you will need to edit traefik's configuration file `traefik.yml`.
- You need to **create fronted network manually**

## Different deployment

- `docker-compose.yml` which contains only containers needed to run Nextcolud. You will need to use your own traefik.
- `docker-compose-traefik.yml` contains all you will need to deploy and expose your Nextcloud instance to web. (Contains traefik)

## Deployment

- Edit Informations in `nextcloud_admin_password.txt`, `nextcloud_admin_user.txt`, `postgres_db.txt`, postgres_password.txt`, 'postgres_user.txt`. Most imortant are files that contains passwords
- Edit postgress password in docker-compose yaml files. Default postgress password is `PgR00tPa$$word`. This is root user

### Create frontend network

``` bash
docker network create frontend
```

### Create databse user and database

User and database must match with ones in the `postgres_db.txt`, postgres_password.txt`, 'postgres_user.txt`

``` bash
# Start postgres container
docker-compose up -f docker-compose-traefik.yml -d nextcloud-postgres
# Login as admin
docker exec -it nextcloud_nextcloud-postgres_1 psql --username postgres # default password is PgR00tPa$$word
# Create user and database
CREATE DATABASE nextcloud;
CREATE USER nextcloud WITH PASSWORD '<your password>';
GRANT ALL PRIVILEGES ON DATABASE nextcloud TO nextcloud;
```

### Deploy instance

``` bash
docker-compose up -f docker-compose-traefik.yml -d
```

## Backing up

### Backup database

```bash
docker exec -t nextcloud_nextcloud-postgres_1 pg_dump -c -U postgres > nextcloud_`date +%d-%m-%Y"_"%H_%M_%S`.sql nextcloud
```

### backup data and keep permision

```bash
tar -cpf data.tar data
```

allow acces to user to copy it with chown

### and restore it

```bash
sudo tar -xpf data.tar --same-owner
```

run as root.


## Not so easy to deploy

This is not just download and deploy, you will need to do some steps

- database and user is not created automatically
- need some configuration before start -> letsencrypt, cloudflare api databse passwords...
- missing configuration to run in local network (without exposing ports to internet)
- you will need to create external network `frontend` manually if you want to use our traefik