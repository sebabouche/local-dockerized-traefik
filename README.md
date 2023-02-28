# Local dockerized traefik to serve multiple docker projects

> This dockerized traffic setup will enable you to develop multiple projects with their own set of urls (no more localhost:3000, 8000, 8080…)

## 1. Choose a local top level domain for your local projects, for example `.test`, `.local` or `.localhost`

### For MacOS

If you don't have many urls per projects and don't switch from one project to another very often, you can add all project urls to `/etc/hosts`:

```
127.0.0.1   admin.myapp1.localhost
127.0.0.1   www.myapp1.localhost

127.0.0.1   grafana.myapp2.localhost
127.0.0.1   www.myapp2.localhost
```

**If you want a wildcard setup** where you can set as many urls you want only once for your projects (inside their `docker-compose.yml`, see below) you should consider installing **dnsmasq**.

Install instructions are for `.localhost`:

```
brew install dnsmasq
mkdir -pv $(brew --prefix)/etc/
echo 'address=/.localhost/127.0.0.1' >> $(brew --prefix)/etc/dnsmasq.conf
echo 'port=53' >> $(brew --prefix)/etc/dnsmasq.conf
sudo mkdir -v /etc/resolver
sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/localhost'
sudo brew services start dnsmasq
```

You can test with `nslookup example.localhost 127.0.0.1` or a simple `ping -c 1 example.localhost`.

### TODO: Linux & Windows

## 2. If needed, adapt the `.env` file

The present setup is configured to:

- set `.localhost` as the local top level domain,
- serve the Traefik Dashboard at `http://traefik.localhost`.

## 3. Create `traefik-network` and start the proxy

```
docker network create traefik-public --driver=overlay --attachable=true
docker-compose up -d
```

## 4. Add traefik labels and the `traefik-public` network to your projects

This setup creates a docker network called `traefik-public` that you will want to add for the frontend parts of your projects. Don't forget to explicitely add the `default` network so that these services can still communicate with the ones that don't need to be exposed on the `traefik-public` network (like databases…)

```
version: "3.8"

services:
  my-admin-app:
    ...
    networks:
      - default
      - traefik-public
    labels:
      - traefik.enable=true
      - traefik.http.routers.admin-http.rule=Host(`my-admin.myapp.localhost`)

  my-frontend-app:
    networks:
      - default
      - traefik-public
    labels:
      - traefik.enable=true
      - traefik.http.routers.fontend-http.rule=Host(`myapp.localhost`)

networks:
  traefik-public:
    external: true

```
