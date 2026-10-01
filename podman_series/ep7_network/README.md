# Episode 7 — Podman Networking

Learn how Podman containers communicate with each other, how custom networks provide DNS-based service discovery, how network isolation works, and how port publishing controls external access.

This episode builds the networking foundation needed for multi-container and microservice applications.

## 🎯 What You'll Learn

* How container network isolation works
* The difference between default and custom Podman networks
* How to create and inspect a Podman network
* How containers communicate by name
* How Podman's internal DNS works with `aardvark-dns`
* How to use network aliases
* How to connect a container to multiple networks
* How network isolation can separate application tiers
* The difference between internal communication and port publishing
* How to bind published ports to specific interfaces
* Rootless networking considerations
* How to build a multi-container application network

## 🧠 Concepts Covered

### Container Networking

Each container has its own network namespace. Containers are isolated from each other unless they share a network.

### Custom Networks

Custom networks provide:

* Private networking
* Container-to-container communication
* Automatic DNS
* Network isolation
* Service discovery through container names

### DNS

Podman's `aardvark-dns` provides DNS resolution on custom networks.

For example:

```text
http://backend:8080
```

instead of using a container IP:

```text
http://10.89.0.3:8080
```

### Network Aliases

A container can expose a stable network name using:

```bash
--network-alias api
```

Other containers can then communicate with:

```text
http://api
```

### Port Publishing

The `-p` option exposes a container port outside its network:

```bash
-p 8080:80
```

Internal container-to-container communication does not require port publishing.

## 🧪 Labs

### Lab 1 — From Broken to Working

Demonstrates the complete networking problem and solution.

You'll:

1. Run two containers without a custom network
2. Attempt container-to-container communication
3. Observe the DNS/networking failure
4. Create a custom network
5. Run both containers on the network
6. Inspect their IP addresses
7. Communicate using container names
8. Connect a container to multiple networks

📁 [`lab-01-network-basics`](./lab1)

---

### Lab 2 — Two Services Communicating by Name

Build a simple two-service setup.

You'll:

1. Create a custom network
2. Run a `times-app`
3. Run a `cities-app`
4. Communicate using container names
5. Compare IP-based communication with DNS-based communication
6. Configure a service URL using an environment variable
7. Use a network alias

📁 [`lab-02-service-communication`](./lab2)

---

### Lab 3 — Port Publishing and Host Access

Learn the difference between internal container communication and external access.

You'll:

1. Publish a container port
2. Inspect published ports
3. Bind a port to all interfaces
4. Bind a port to localhost only
5. List all published ports

📁 [`lab-03-port-publishing`](./lab3)

---

