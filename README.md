
<p align="center">
  <a href="https://netmaker.io">
  <img src="https://raw.githubusercontent.com/gravitl/netmaker-docs/master/images/netmaker-github/netmaker-teal.png" width="50%"><break/>
  </a>
</p>

<p align="center">
<a href="https://runacap.com/ross-index/annual-2022/" target="_blank" rel="noopener">
    <img src="https://runacap.com/wp-content/uploads/2023/02/Annual_ROSS_badge_white_2022.svg" alt="ROSS Index - Fastest Growing Open-Source Startups | Runa Capital" width="17%" />
</a>  
<a href="https://www.ycombinator.com/companies/netmaker/" target="_blank" rel="noopener">
    <img src="https://raw.githubusercontent.com/gravitl/netmaker-docs/master/images/netmaker-github/y-combinator.png" alt="Y-Combinator" width="16%" />
</a>  
</p>

<p align="center">
  <a href="https://github.com/gravitl/netmaker/releases">
    <img src="https://img.shields.io/badge/Version-0.90.0-informational?style=flat-square" />
  </a>
  <a href="https://hub.docker.com/r/gravitl/netmaker/tags">
    <img src="https://img.shields.io/docker/pulls/gravitl/netmaker?label=downloads" />
  </a>  
  <a href="https://goreportcard.com/report/github.com/gravitl/netmaker">
    <img src="https://goreportcard.com/badge/github.com/gravitl/netmaker" />
  </a>
  <a href="https://twitter.com/intent/follow?screen_name=netmaker_io">
    <img src="https://img.shields.io/twitter/follow/netmaker_io?label=follow&style=social" />
  </a>
  <a href="https://www.youtube.com/channel/UCach3lJY_xBV7rGrbUSvkZQ">
    <img src="https://img.shields.io/youtube/channel/views/UCach3lJY_xBV7rGrbUSvkZQ?style=social" />
  </a>
  <a href="https://reddit.com/r/netmaker">
    <img src="https://img.shields.io/reddit/subreddit-subscribers/netmaker?label=%2Fr%2Fnetmaker&style=social" />
  </a>  
  <a href="https://discord.gg/zRb9Vfhk8A">
    <img src="https://img.shields.io/discord/825071750290210916?color=%09%237289da&label=chat" />
  </a> 
</p>


#be sure to create jail.local in fail2ban, install xcaddy, and use transform encoder, check log file paths
xcaddy build --with github.com/caddyserver/transform-encoder
chmod +x ./caddy
mv ./caddy /usr/bin/caddy  # or package it in a Dockerfile
Custom Caddy docker
FROM caddy:builder AS builder
RUN xcaddy build --with github.com/caddyserver/transform-encoder

FROM alpine:3.18
RUN apk add --no-cache ca-certificates
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
RUN chmod +x /usr/bin/caddy
ENTRYPOINT ["/usr/bin/caddy"]
docker build -f Dockerfile.custom-caddy -t caddy:custom .

api.nm.ingsoc.xyz {
  tls /certs/fullchain.pem /certs/dashboard_nm_ingsoc_xyz.key

  log {
    format transform "{common_log}"
    output file /root/data/logs/access.log {
      roll_size 5MiB
      roll_keep 10
      roll_keep_for 336h
    }
  }

  handle /api/* {
    reverse_proxy netmaker:8081 {
      header_up X-Real-IP {remote_host}
    }
  }

  handle {
    respond "404 Not Found" 404
  }
}

[Definition]
failregex = ^<HOST> - - \[.*\] "POST /api/users/adm/authenticate HTTP/2\.0" 401 \d+$
ignoreregex =

[netmaker-api]
enabled = true
port = http,https
filter = netmaker-api
logpath = /root/data/logs/access.log
maxretry = 3
findtime = 600
bantime = 3600
backend = auto

sudo systemctl restart fail2ban
sudo fail2ban-client status netmaker-api

for i in {1..6}; do
  curl -k -X POST https://api.nm.ingsoc.xyz/api/users/adm/authenticate \
       -H "Content-Type: application/json" \
       -d '{"username":"fake@user.com","password":"wrong"}'
  sleep 1
done

sudo fail2ban-client status netmaker-api


=========================================================================

sudo fail2ban-client status netmaker-api

fail2ban-regex /var/log/caddy/access.log /etc/fail2ban/filter.d/netmaker-api.conf

docker restart caddy
docker logs -f caddy

docker exec -it caddy tail -n 5 /root/data/logs/access.log

docker exec -it caddy caddy adapt --config /etc/caddy/Caddyfile --pretty


cat /etc/fail2ban/filter.d/netmaker-api.conf
cat /etc/fail2ban/jail.d/netmaker-api.local
cat /root/Caddyfile

/root/data/logs/access.log 
cat /var/log/netmaker-fail2ban.log
cat /usr/local/bin/netmaker-logfilter.sh
cat /var/log/netmaker-filter-debug.log

hexdump -C /etc/fail2ban/filter.d/netmaker-api.conf
printf '[Definition]\nfailregex = ^<HOST> - \\[[^]]+\\] "POST /api/users/adm/authenticate" 401$\n' > /etc/fail2ban/filter.d/netmaker-api.conf
echo -e '[Definition]\nfailregex = ^<HOST> - \\[[^\\]]+\\] "POST /api/users/adm/authenticate" 401$' > /etc/fail2ban/filter.d/netmaker-api.conf
cat -A /etc/fail2ban/filter.d/netmaker-api.conf
od -c /etc/fail2ban/filter.d/netmaker-api.conf | grep 'POST'
docker exec -it caddy wget -qO- http://netmaker:8081/api/auth/login


docker logs -f caddy

docker inspect caddy | grep -A 10 Mounts

tail -n 20 /var/lib/docker/volumes/root_caddy_data/_data/caddy/logs/api.log

docker exec -it caddy tail -n 5 /root/data/logs/access.log


jq -r 'select(.status == 401) | "\(.request.remote_ip) - - [\(.ts | strftime("%d/%b/%Y:%H:%M:%S %z"))] \"POST \(.request.uri)\" 401"' /root/data/logs/access.log > /var/log/netmaker/converted-access.log


for i in {1..6}; do   curl -k -X POST https://api.nm.ingsoc.xyz/api/auth/login     -H "Content-Type: application/json"     -d '{"username":"fakeuser","password":"wrong"}'; done


curl -k -X POST https://api.nm.ingsoc.xyz/api/users/adm/authenticate   -H "Content-Type: application/json"   -d '{"username":"no@no.com","password":"wrong"}'


root@ubnmCA020a01:~# sudo jq -r '
>   select(.status == 401) |
>   "\(.request.remote_ip) - - [\(.ts | floor | strftime(\"%d/%b/%Y:%H:%M:%S\"))] \"POST \(.request.uri)\" 401"
> ' /root/data/logs/access.log
jq: error: syntax error, unexpected INVALID_CHARACTER (Unix shell quoting issues?) at <top-level>, line 3:
  "\(.request.remote_ip) - - [\(.ts | floor | strftime(\"%d/%b/%Y:%H:%M:%S\"))] \"POST \(.request.uri)\" 401"                                                       
jq: 1 compile error



Caddyfile

# API
api.nm.ingsoc.xyz {
  tls /certs/fullchain.pem /certs/dashboard_nm_ingsoc_xyz.key

  log {
    output file /data/logs/access.log {
      roll_size 5MiB
      roll_keep 10
      roll_keep_for 336h
    }
    format json
  }

  handle /api/* {
    reverse_proxy netmaker:8081 {
      header_up X-Real-IP {remote_host}
    }
  }

  handle {
    respond "404 Not Found" 404
  }
}



keygen process
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
scp ~/.ssh/id_rsa youruser@windows_machine:/path/to/save/
ssh-keygen -p -m PEM -f ~/.ssh/id_rsa
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
openssl req -new -newkey rsa:2048 -nodes -keyout nm.ingsoc.xyz.key -out nm.ingsoc.xyz.csr
sudo mkdir -p /etc/ssl/nm.ingsoc.xyz
sudo cp nm.ingsoc.xyz.key wildcard_nm_ingsoc_xyz.crt ca_bundle.crt /etc/ssl/nm.ingsoc.xyz/
cat wildcard_nm_ingsoc_xyz.crt ca_bundle.crt > fullchain.pem

server {
    listen 443 ssl;
    server_name dashboard.nm.ingsoc.xyz api.nm.ingsoc.xyz broker.nm.ingsoc.xyz;

    ssl_certificate /etc/ssl/nm.ingsoc.xyz/fullchain.pem;
    ssl_certificate_key /etc/ssl/nm.ingsoc.xyz/nm.ingsoc.xyz.key;

    # Your proxy / app settings here
}

sudo systemctl reload nginx

openssl s_client -connect dashboard.nm.ingsoc.xyz:443

dig CNAME _a1b2c3d4e5.dashboard.nm.ingsoc.xyz +short

sudo mkdir -p /opt/netmaker/certs
sudo cp your_domain.crt /opt/netmaker/certs/fullchain.pem
sudo cp ca_bundle.crt /opt/netmaker/certs/ca.pem
sudo cp private.key /opt/netmaker/certs/privkey.pem

cat your_domain.crt ca_bundle.crt > /opt/netmaker/certs/fullchain.pem

dashboard.nm.ingsoc.xyz, api.nm.ingsoc.xyz, broker.nm.ingsoc.xyz {
    tls /certs/fullchain.pem /certs/privkey.pem
    reverse_proxy localhost:8080  # Or whatever Netmaker uses
}


services:
  caddy:
    image: caddy:latest
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - /opt/netmaker/certs:/certs:ro
    ports:
      - "80:80"
      - "443:443"
    restart: unless-stopped

openssl s_client -connect dashboard.nm.ingsoc.xyz:443

cd /opt/netmaker/certs

# Create fullchain.pem for Caddy
cat dashboard_nm_ingsoc_xyz.crt dashboard_nm_ingsoc_xyz.ca-bundle > fullchain.pem
mv dashboard_nm_ingsoc_xyz.crt privkey.pem  # If you generated privkey earlier


4. Run Your Modified Installer
bash
Copy
Edit
sudo ./nm-quick.sh


local COMPOSE_URL="https://raw.githubusercontent.com/youruser/yourrepo/main/docker-compose.yml"
local CADDY_URL="https://raw.githubusercontent.com/youruser/yourrepo/main/Caddyfile"

=========================================================================





# WireGuard<sup>®</sup> automation from homelab to enterprise

| Create                                    | Manage                                  | Automate                                |
|-------------------------------------------|-----------------------------------------|-----------------------------------------|
| :heavy_check_mark: WireGuard Networks     | :heavy_check_mark: Admin UI             | :heavy_check_mark: Linux                |
| :heavy_check_mark: Remote Access Gateways | :heavy_check_mark: OAuth                | :heavy_check_mark: Docker              |
| :heavy_check_mark: Mesh VPNs              | :heavy_check_mark: Private DNS          | :heavy_check_mark: Mac                  |
| :heavy_check_mark: Site-to-Site           | :heavy_check_mark: Access Control Lists | :heavy_check_mark: Windows              |

# Try Netmaker SaaS  

If you're looking for a managed service, you can get started with just a few clicks, visit [netmaker.io](https://account.netmaker.io) to create your netmaker server.  

# Self-Hosted Open Source Quick Start  

These are the instructions for deploying a Netmaker server on your cloud VM as quickly as possible. For more detailed instructions, visit the [Install Docs](https://docs.netmaker.io/docs/server-installation/quick-install#quick-install-script).  

1. Get a cloud VM with Ubuntu 24.04 and a static public IP.
2. Allow inbound traffic on port 443,51821 TCP and UDP to the VM firewall in cloud security settings, and for simplicity, allow outbound on All TCP and All UDP.
3. (recommended) Prepare DNS - Set a wildcard subdomain in your DNS settings for Netmaker, e.g. *.netmaker.example.com, which points to your VM's public IP.
4. Run the script to setup open source version of Netmaker: 

`sudo wget -qO /root/nm-quick.sh https://raw.githubusercontent.com/gravitl/netmaker/master/scripts/nm-quick.sh && sudo chmod +x /root/nm-quick.sh && sudo /root/nm-quick.sh`

**<pre>To Install Self-Hosted PRO Version - https://docs.netmaker.io/docs/server-installation/netmaker-professional-setup</pre>** 



<p float="left" align="middle">
<img src="https://raw.githubusercontent.com/gravitl/netmaker-docs/master/images/netmaker-github/readme.gif" />
</p>

After installing Netmaker, check out the [Walkthrough](https://itnext.io/getting-started-with-netmaker-a-wireguard-virtual-networking-platform-3d563fbd87f0) and [Getting Started](https://docs.netmaker.io/docs/getting-started) guides to learn more about configuring networks. Or, check out some of our other [Tutorials](https://www.netmaker.io/blog) for different use cases, including Kubernetes.

# Get Support

- [Discord](https://discord.gg/zRb9Vfhk8A)

- [Reddit](https://reddit.com/r/netmaker)

- [Learning Resources](https://netmaker.io/blog)

# Why Netmaker + WireGuard?

- Netmaker automates virtual networks between data centres, clouds, and edge devices, so you don't have to.

- Kernel WireGuard offers maximum speed, performance, and security. 

- Netmaker is built to scale from small businesses to enterprises. 

- Netmaker with WireGuard can be highly customized for peer-to-peer, site-to-site, Kubernetes, and more.

# Community Projects

- [Netmaker + Traefik Proxy](https://github.com/bsherman/netmaker-traefik)

- [OpenWRT Netclient Packager](https://github.com/sbilly/netmaker-openwrt)

- [Golang GUI](https://github.com/mattkasun/netmaker-gui)

- [CoreDNS Plugin](https://github.com/gravitl/netmaker-coredns-plugin)

- [Multi-Cluster K8S Plugin](https://github.com/gravitl/netmak8s)

- [Terraform Provider](https://github.com/madacluster/netmaker-terraform-provider)

- [VyOS Integration](https://github.com/kylechase/vyos-netmaker)

- [Netmaker K3S](https://github.com/geragcp/netmaker-k3s)

- [Run Netmaker + Netclient with Podman](https://github.com/agorgl/nm-setup)

## Disclaimer
 [WireGuard](https://wireguard.com/) is a registered trademark of Jason A. Donenfeld.

## License

Netmaker's source code and all artifacts in this repository are freely available.
All content that resides under the "pro/" directory of this repository, if that
directory exists, is licensed under the license defined in "pro/LICENSE".
All third party components incorporated into the Netmaker Software are licensed
under the original license provided by the owner of the applicable component.
Content outside of the above mentioned directories or restrictions above is
available under the "Apache Version 2.0" license as defined below.
All details for the licenses used can be found here: [LICENSE.md](./LICENSE.md).
