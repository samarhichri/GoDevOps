# Lab 2 — Service-to-Service Communication

This lab demonstrates how two services can communicate through a Podman custom network using container names and DNS.

## 🎯 Objective

You will learn how to:

* Create a service network
* Run multiple services on the same network
* Communicate using container names
* Compare IP-based and name-based communication
* Use environment variables for service configuration
* Use network aliases

## Architecture

```text
┌──────────────────── cities-network ────────────────────┐
│                                                        │
│   ┌─────────────┐             ┌─────────────┐          │
│   │  times-app  │◄────────────│ cities-app  │          │
│   │   :80       │             │   :80       │          │
│   └─────────────┘             └─────────────┘          │
│                                                        │
│        DNS: times-app → container IP                   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

## 1. Create the Network

```bash
podman network create cities-network
```

## 2. Run the Times Service

```bash
podman run -d \
  --name times-app \
  --network cities-network \
  -p 8080:80 \
  docker.io/library/nginx
```

## 3. Inspect the IP

```bash
podman inspect times-app \
  -f '{{(index .NetworkSettings.Networks "cities-network").IPAddress}}'
```

The IP is useful for debugging, but application configuration should use the service name instead.

## 4. Test Communication by IP

```bash
IP_TIMES=$(podman inspect times-app \
  -f '{{(index .NetworkSettings.Networks "cities-network").IPAddress}}')
```

Then:

```bash
podman run --rm \
  --network cities-network \
  docker.io/library/nginx \
  curl -s http://$IP_TIMES | head -3
```

## 5. Test Communication by Name

```bash
podman run --rm \
  --network cities-network \
  docker.io/library/nginx \
  curl -s http://times-app | head -3
```

This is the preferred approach.

Instead of:

```text
http://10.89.x.x
```

the application uses:

```text
http://times-app
```

## 6. Configure the Service URL

Run the cities service:

```bash
podman run -d \
  --name cities-app \
  --network cities-network \
  -p 8090:80 \
  -e TIMES_APP_URL=http://times-app:80 \
  docker.io/library/nginx
```

The service URL is now externalized through an environment variable.

## 7. Verify Communication

```bash
podman exec cities-app \
  curl -s http://times-app | head -3
```

## 8. Test a Network Alias

Create a backend with the alias `api`:

```bash
podman run -d \
  --name my-backend-service-v2 \
  --network cities-network \
  --network-alias api \
  docker.io/library/nginx
```

Test:

```bash
podman run --rm \
  --network cities-network \
  docker.io/library/nginx \
  curl -s http://api | head -3
```

The actual container name does not matter to the client.

The service is available through:

```text
api
```

## 9. Cleanup

```bash
podman rm -f times-app cities-app my-backend-service-v2

podman network rm cities-network
```

## ✅ Key Takeaways

* Use custom networks for service-to-service communication.
* Prefer service names over container IP addresses.
* Container IPs can change.
* DNS provides service discovery.
* Network aliases provide stable service names.
* Environment variables keep service configuration outside the application image.
