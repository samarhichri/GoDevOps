# Lab 3 — Port Publishing and Host Access

## Objective

Understand the difference between:

- Container-to-container communication
- Container-to-host communication
- Public port publishing
- Localhost-only binding

## 1. Publish a Port

podman run -d --name web-public -p 8080:80 docker.io/library/nginx

Test:

curl -s localhost:8080 | head -3

Check:

podman port web-public

## 2. Bind to Localhost Only

podman run -d --name web-local -p 127.0.0.1:8090:80 docker.io/library/nginx

Check:

podman port web-local

## 3. List All Published Ports

podman port --all

## 4. Cleanup

podman rm -f web-public web-local

## Key Takeaways

- `-p HOST:CONTAINER` publishes a container port.
- `0.0.0.0` exposes the port on all interfaces.
- `127.0.0.1` restricts access to localhost.
- Containers on the same custom network do not need `-p` to communicate.
- Only publish ports that need to be accessed from outside the container network.
