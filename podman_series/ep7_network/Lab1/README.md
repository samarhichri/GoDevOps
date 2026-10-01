# Lab 1 — Podman Network Basics

This lab demonstrates how Podman containers communicate before and after joining a custom network.

## 🎯 Objective

By the end of this lab, you will understand:

* Why containers cannot automatically resolve each other by name
* How to create a custom Podman network
* How containers receive private IP addresses
* How Podman's internal DNS resolves container names
* How to inspect network configuration
* How to connect a container to multiple networks

---

## 1. Run Two Containers Without a Custom Network

Start two Alpine containers:

```bash
podman run -d --name container-a docker.io/library/alpine sleep 3600

podman run -d --name container-b docker.io/library/alpine sleep 3600
```

Verify:

```bash
podman ps
```

---

## 2. Try Container-to-Container Communication

Try to reach `container-b` from `container-a`:

```bash
podman exec container-a wget -qO- http://container-b 2>&1
```

Expected result:

```text
wget: bad address 'container-b'
```

The container name cannot be resolved.

---

## 3. Create a Network

Create a custom network:

```bash
podman network create demo-network
```

Inspect it:

```bash
podman network inspect demo-network
```

Look for:

```text
dns_enabled: true
```

---

## 4. Recreate the Containers

Remove the existing containers:

```bash
podman rm -f container-a container-b
```

Run them again on the custom network:

```bash
podman run -d \
  --name container-a \
  --network demo-network \
  docker.io/library/nginx
```

```bash
podman run -d \
  --name container-b \
  --network demo-network \
  docker.io/library/nginx
```

---

## 5. Inspect the IP Address

```bash
podman inspect container-a \
  -f '{{(index .NetworkSettings.Networks "demo-network").IPAddress}}'
```

The container receives a private IP address from the network.

---

## 6. Communicate Using the Container Name

From `container-a`, reach `container-b`:

```bash
podman exec container-a \
  curl -s http://container-b | head -3
```

The request succeeds.

The important point is that we did **not** use the IP address.

Podman's internal DNS resolved:

```text
container-b
      ↓
container-b's IP address
```

---

## 7. Connect a Container to Multiple Networks

Create a second network:

```bash
podman network create second-network
```

Run a container on both networks:

```bash
podman run -d \
  --name bridge-container \
  --network demo-network,second-network \
  docker.io/library/nginx
```

The container now belongs to both networks.

Inspect:

```bash
podman inspect bridge-container
```

You should see both networks in its network configuration.

---

## 8. Cleanup

Remove the containers:

```bash
podman rm -f container-a container-b bridge-container
```

Remove the networks:

```bash
podman network rm demo-network second-network
```

Verify:

```bash
podman network ls
```

## ✅ Key Takeaways

* Containers are isolated by default.
* Custom networks enable container-to-container communication.
* Containers on a custom network can communicate using their names.
* Podman's `aardvark-dns` provides DNS resolution.
* Container IP addresses should not be hardcoded into application configuration.
* A container can belong to multiple networks.
* Network topology can be used to isolate application components.
