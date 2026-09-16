# Podman Secrets - Hands-On Labs

This document contains all commands used in Episode 6 of the GoDevOps Podman series.

---

# LAB 1 - EXPOSING THE PROBLEM LIVE

## STEP 1 - Recreate the Episode 5 Container

```bash
podman run -d \
  --name demo-postgres \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  docker.io/library/postgres:15
```

## STEP 2 - Leak 1: `podman inspect`

```bash
podman inspect demo-postgres | grep -A 20 '"Env"'
```

## STEP 3 - Leak 2: Shell History

```bash
cat ~/.bash_history | grep POSTGRES
```

---

# LAB 2 - MIGRATING THE POSTGRES CONTAINER TO SECRETS

## STEP 1 - Remove the Old Container

```bash
podman rm -f demo-postgres
```

## STEP 2 - Create the Secret

```bash
printf "secret" | podman secret create postgres_password -
```

## STEP 3 - Verify the Secret Exists

```bash
podman secret ls
```

## STEP 4 - Run PostgreSQL with the Secret

```bash
podman run -d \
  --name demo-postgres \
  -e POSTGRES_PASSWORD_FILE=/run/secrets/postgres_password \
  --secret postgres_password \
  -v postgres-data:/var/lib/postgresql/data \
  -p 5432:5432 \
  docker.io/library/postgres:15
```

## STEP 5 - Verify with `podman inspect`

```bash
podman inspect demo-postgres | grep -A 20 '"Env"'
```

---

# LAB 3 - BUILD SECRET IN A CONTAINERFILE

## STEP 1 - Create the Wrong Containerfile

Create the working directory:

```bash
mkdir build-secret-demo && cd build-secret-demo
```

### Containerfile - First Version

```dockerfile
FROM docker.io/library/alpine

ARG API_TOKEN

RUN echo "Using token: $API_TOKEN" 
```

### Build the Image

```bash
podman build \
  --build-arg API_TOKEN=supersecrettoken123 \
  -t secretapp-wrong:latest .
```

## STEP 2 - Inspect the Layer History

```bash
podman image history secretapp-wrong:latest
```

## STEP 3 - Create the Correct Containerfile

### Containerfile - Second Version

```dockerfile
FROM docker.io/library/alpine

RUN --mount=type=secret,id=build_token \
    TOKEN=$(cat /run/secrets/build_token) 
```

## STEP 4 - Create the Secret and Build

```bash
printf "supersecrettoken123" | podman secret create build_token -
```

```bash
podman build \
  --secret id=build_token \
  -t secretapp-correct:latest .
```

## STEP 5 - Inspect the Clean History

```bash
podman image history secretapp-correct:latest
```

---

# LAB 4 - WORDPRESS + MYSQL FULL STACK WITH SECRETS

## STEP 1 - Clean Up from Previous Labs

```bash
cd ~

podman rm -f demo-postgres
podman secret rm build_token
```

## STEP 2 - Create Volumes for Persistence

```bash
podman volume create mysql-data
podman volume create wordpress-data
```

## STEP 3 - Create the Secrets

```bash
printf "mysql_root_secret" | podman secret create mysql_root_password -

printf "wp_db_secret" | podman secret create mysql_wp_password -
```

## STEP 4 - Verify Secrets

```bash
podman secret ls
```

## STEP 5 - Create a Network

```bash
podman network create wp-network
```

## STEP 6 - Run MySQL with Secrets

```bash
podman run -d \
  --name mysql \
  --network wp-network \
  -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql_root_password \
  -e MYSQL_PASSWORD_FILE=/run/secrets/mysql_wp_password \
  -e MYSQL_DATABASE=wordpress \
  -e MYSQL_USER=wp \
  --secret mysql_root_password \
  --secret mysql_wp_password \
  -v mysql-data:/var/lib/mysql \
  docker.io/library/mysql:8
```

## STEP 7 - Run WordPress with Secrets

```bash
podman run -d \
  --name wordpress \
  --network wp-network \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_NAME=wordpress \
  -e WORDPRESS_DB_PASSWORD_FILE=/run/secrets/mysql_wp_password \
  --secret mysql_wp_password \
  -v wordpress-data:/var/www/html \
  -p 8080:80 \
  docker.io/library/wordpress
```

## STEP 8 - Verify the Stack is Running

```bash
podman ps
```

```bash
curl -s -o /dev/null -w "%{http_code}" localhost:8080
```

## STEP 9 - Verify Secrets are Hidden in `podman inspect`

```bash
podman inspect mysql | grep -A 20 '"Env"'
```

```bash
podman inspect wordpress | grep -A 20 '"Env"'
```

---

# CLEANUP

```bash
podman rm -f mysql wordpress
```

```bash
podman volume rm mysql-data wordpress-data
```

```bash
podman network rm wp-network
```

```bash
podman secret rm mysql_root_password
```
