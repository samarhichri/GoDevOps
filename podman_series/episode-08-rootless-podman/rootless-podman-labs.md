# Episode 8 : Rootless Podman Labs

## Lab 1 : Who Is Running Inside Your Containers?

```bash
id
whoami

podman run --rm docker.io/library/alpine id
podman run --rm docker.io/library/alpine cat /proc/self/uid_map

podman run -d --name running-test docker.io/library/alpine sleep 3600
podman top running-test huser user

sudo podman run --rm docker.io/library/alpine cat /proc/self/uid_map

podman rm -f running-test
```

## Lab 2 : Proving the User Mapping

```bash
cat > /tmp/Containerfile.maptest << 'EOF'
FROM docker.io/library/alpine
RUN adduser -D -u 1500 appuser
USER appuser
CMD ["sleep", "3600"]
EOF

podman build -f /tmp/Containerfile.maptest -t maptest:latest

podman run -d --name as-root docker.io/library/alpine sleep 3600
podman run -d --name as-user maptest:latest

podman top as-root huser user
podman top as-user huser user

podman exec as-root cat /proc/self/uid_map
podman exec as-user cat /proc/self/uid_map

podman rm -f as-root as-user
podman rmi maptest:latest
rm /tmp/Containerfile.maptest
```

## Lab 3 : Building the Same App Two Ways

```bash
mkdir security-demo && cd security-demo
```

Create `server.py`:

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import os

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        uid = os.getuid()
        try:
            import pwd
            username = pwd.getpwuid(uid).pw_name
        except Exception:
            username = "unknown"
        self.send_response(200)
        self.end_headers()
        response = f"Process UID: {uid} | Username: {username}\n"
        self.wfile.write(response.encode())

    def log_message(self, format, *args):
        pass

if __name__ == '__main__':
    print(f"Starting server as UID {os.getuid()}")
    HTTPServer(('', 8080), Handler).serve_forever()
```

Build the insecure image:

```bash
cat > Containerfile.insecure << 'EOF'
FROM docker.io/library/python:3.11-slim
WORKDIR /app
COPY server.py .
CMD ["python3", "server.py"]
EOF

podman build -f Containerfile.insecure -t demo:insecure .
```

Build the secure image:

```bash
cat > Containerfile.secure << 'EOF'
FROM docker.io/library/python:3.11-slim

RUN adduser \
    --disabled-password \
    --no-create-home \
    --system \
    --uid 1001 \
    appuser

WORKDIR /app
COPY server.py .
RUN chown appuser:nogroup /app/server.py

USER appuser

CMD ["python3", "server.py"]
EOF

podman build -f Containerfile.secure -t demo:secure .
```

Run and compare:

```bash
podman run -d --name insecure -p 8080:8080 demo:insecure
podman run -d --name secure -p 8081:8080 demo:secure

sleep 1

curl -s localhost:8080
curl -s localhost:8081

podman top insecure huser user
podman top secure huser user
```

Cleanup:

```bash
podman rm -f insecure secure
podman rmi demo:insecure demo:secure
cd .. && rm -rf security-demo
```

## Lab 4 — Breaking and Fixing the UID Mapping

> **Warning:** This lab modifies `/etc/subuid` and `/etc/subgid`. Run it only on a test Linux system.

Create the test image:

```bash
cat > /tmp/Containerfile.apptest << 'EOF'
FROM docker.io/library/alpine
RUN adduser -D -u 1001 appuser
USER appuser
CMD ["id"]
EOF

podman build -f /tmp/Containerfile.apptest -t apptest:latest
podman run --rm apptest:latest
```

Back up and remove the current user's mappings:

```bash
sudo cp /etc/subuid /etc/subuid.bak
sudo cp /etc/subgid /etc/subgid.bak

sudo sed -i "/^$(whoami)/d" /etc/subuid
sudo sed -i "/^$(whoami)/d" /etc/subgid

cat /etc/subuid
podman run --rm apptest:latest
```

Restore the mapping:

```bash
sudo touch /etc/subuid /etc/subgid

sudo usermod \
  --add-subuids 100000-165535 \
  --add-subgids 100000-165535 \
  "$(whoami)"

cat /etc/subuid /etc/subgid
```

Reload Podman's configuration and test:

```bash
podman run --rm apptest:latest

podman system migrate
podman run --rm apptest:latest
```

Restore the original configuration:

```bash
sudo cp /etc/subuid.bak /etc/subuid
sudo cp /etc/subgid.bak /etc/subgid

podman system migrate

sudo rm -f /etc/subuid.bak /etc/subgid.bak
podman rmi apptest:latest
rm /tmp/Containerfile.apptest
```
