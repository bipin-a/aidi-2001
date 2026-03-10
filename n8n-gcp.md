The first thing to fix is the goal: there are really **two different setups**. If you want **n8n reachable by third-party webhooks**, the current n8n docs point you toward a **domain + HTTPS + reverse proxy** setup, not raw HTTP on `:5678`. If you want **the UI reachable only from your IP**, you can do that, but then external webhook services like GitHub/Slack/Stripe won’t be able to call into n8n. ([n8n Docs][1])

Also, the “Always Free” part still covers **1 non-preemptible `e2-micro` in `us-east1`/`us-central1`/`us-west1`**, plus **30 GB-months of standard persistent disk** and **1 GB/month outbound transfer from North America**. What no longer fits the old “definitely $0” claim is **external IPv4**: Google’s current pricing says **static and ephemeral external IP addresses are charged** under external IP pricing. ([Google Cloud Documentation][2])

## Which version you should use

I’d use this split:

1. **Recommended for real use:** public **HTTPS** with a domain and reverse proxy. This matches current n8n docs and supports public webhooks. ([n8n Docs][1])
2. **Minimal admin-only version:** direct access on `http://VM_IP:5678`, firewall-limited to your IP. This is okay for private use/testing, but not for public webhook triggers. ([n8n Docs][3])

---

# Option A — Recommended: n8n on GCP VM with HTTPS, domain, and public webhooks

This is the version closest to current official n8n guidance. n8n’s Docker Compose docs use a **domain/subdomain**, **Traefik**, **HTTPS**, bind n8n only to `127.0.0.1:5678`, and expose only `80/443`. The docs also show `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true`, `N8N_RUNNERS_ENABLED=true`, `NODE_ENV=production`, `WEBHOOK_URL`, and timezone settings. ([n8n Docs][1])

## Step 1: Create the GCP project and budget alert

Create a project in Google Cloud and attach billing. I would also create a budget alert immediately, because this setup is **not guaranteed** to stay at exactly $0 once you involve external IPv4 or go beyond the Always Free allowances. ([Google Cloud Documentation][2])

## Step 2: Reserve a static external IPv4 address

For a domain-backed n8n instance, a static external IP is the safer choice. Google says an **ephemeral** external IP is released when a VM is stopped and a **new** one can be assigned on restart; a static IP avoids that. Google’s current pricing also says external IPv4 addresses are billed separately, so this is a stability decision, not a “free while attached” trick. ([Google Cloud Documentation][4])

Use the GCP console path:

* **VPC network → IP addresses**
* **Reserve external static IP address**
* reserve a **regional IPv4** in the same region as the VM. ([Google Cloud Documentation][5])

## Step 3: Create the VM

Use:

* **Region:** `us-east1`, `us-central1`, or `us-west1`
* **Machine type:** `e2-micro`
* **Boot disk:** **Ubuntu 24.04 LTS**
* **Boot disk type:** **Standard persistent disk**
* **Size:** **30 GB**
* **External IP:** attach the static IP you reserved
* **Network tag:** `n8n-public` ([Google Cloud Documentation][2])

I would **leave “Allow HTTP traffic” and “Allow HTTPS traffic” unchecked** and create the firewall rules yourself. Those checkboxes create ingress rules for `tcp:80` and `tcp:443`; that’s fine, but explicit rules with your own target tag are cleaner and easier to audit. ([Google Cloud Documentation][6])

## Step 4: Create explicit firewall rules for 80 and 443

Google firewall rules apply at the network level, and if you don’t specify a target, the rule defaults to **all instances in the network**. That is why the rule should target your `n8n-public` tag. ([Google Cloud Documentation][7])

From **Cloud Shell** or another machine with the Google Cloud CLI configured, run:

```bash
gcloud compute firewall-rules create n8n-public-http \
  --network=default \
  --action=ALLOW \
  --direction=INGRESS \
  --source-ranges=0.0.0.0/0 \
  --target-tags=n8n-public \
  --rules=tcp:80

gcloud compute firewall-rules create n8n-public-https \
  --network=default \
  --action=ALLOW \
  --direction=INGRESS \
  --source-ranges=0.0.0.0/0 \
  --target-tags=n8n-public \
  --rules=tcp:443
```

These flags line up with Google’s current firewall docs for network, target tags, source ranges, and protocol/port rules. ([Google Cloud Documentation][8])

## Step 5: Point a DNS record at the static IP

n8n’s current Compose guide expects a subdomain like `n8n.example.com` pointing at your server IP. Create an **A record** for your chosen subdomain to the static IPv4 you reserved. ([n8n Docs][1])

## Step 6: SSH into the VM and install Docker the current way

Docker’s current Ubuntu docs recommend the **apt repository** method, not the convenience script, and they recommend the **Compose plugin** (`docker compose`), not the old `docker-compose` package. They also note the convenience script is only recommended for testing/development. ([Docker Documentation][9])

Run:

```bash
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1) || true

sudo apt update
sudo apt install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

docker --version
docker compose version
sudo systemctl status docker
```

That matches Docker’s current Ubuntu install flow and verification commands. ([Docker Documentation][9])

## Step 7: Optional but practical on a tiny VM — add swap

This is not an n8n doc requirement, but on an `e2-micro` it is a reasonable operational safeguard to reduce OOM crashes. Treat it as a practical tweak, not “extra RAM.” The free-tier machine and disk limits are still the same. ([Google Cloud Documentation][2])

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h
```

## Step 8: Allow your user to run Docker without sudo

Current n8n docs show:

```bash
sudo usermod -aG docker ${USER}
exec sg docker newgrp
groups
```

That is the current n8n-documented way to refresh group membership in the current session. ([n8n Docs][10])

## Step 9: Create the n8n project files

n8n’s Compose docs use a project directory, an `.env` file, a `local-files` directory, and a `compose.yaml`. The current official example uses `docker.n8n.io/n8nio/n8n`, binds n8n only to loopback on `5678`, and puts Traefik in front for TLS. ([n8n Docs][1])

Create the directories:

```bash
mkdir -p ~/n8n-compose/local-files
cd ~/n8n-compose
```

Create `.env`:

```bash
cat > .env <<'EOF'
DOMAIN_NAME=example.com
SUBDOMAIN=n8n
GENERIC_TIMEZONE=America/Toronto
SSL_EMAIL=you@example.com
EOF
```

Now create `compose.yaml`:

```yaml
services:
  traefik:
    image: traefik
    restart: always
    command:
      - "--providers.docker=true"
      - "--providers.docker.exposedbydefault=false"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.mytlschallenge.acme.tlschallenge=true"
      - "--certificatesresolvers.mytlschallenge.acme.email=${SSL_EMAIL}"
      - "--certificatesresolvers.mytlschallenge.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - traefik_data:/letsencrypt
      - /var/run/docker.sock:/var/run/docker.sock:ro

  n8n:
    image: docker.n8n.io/n8nio/n8n:2.10.4
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    labels:
      - traefik.enable=true
      - traefik.http.routers.n8n.rule=Host(`${SUBDOMAIN}.${DOMAIN_NAME}`)
      - traefik.http.routers.n8n.tls=true
      - traefik.http.routers.n8n.entrypoints=web,websecure
      - traefik.http.routers.n8n.tls.certresolver=mytlschallenge
    environment:
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_HOST=${SUBDOMAIN}.${DOMAIN_NAME}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - N8N_PROXY_HOPS=1
      - N8N_RUNNERS_ENABLED=true
      - NODE_ENV=production
      - WEBHOOK_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}/
      - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
      - TZ=${GENERIC_TIMEZONE}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./local-files:/files

volumes:
  n8n_data:
  traefik_data:
```

Why this version is better than the original:

* it uses the **official registry path** and current env vars from n8n docs, ([n8n Docs][11])
* it uses **HTTPS** the way n8n currently recommends, ([n8n Docs][12])
* it binds n8n only to **127.0.0.1:5678** and exposes **80/443** through Traefik, just like the official Compose example, ([n8n Docs][1])
* and I pinned `n8n` to **2.10.4**, which is the current stable release as of March 10, 2026, instead of using a floating tag. n8n says new minor versions ship most weeks, so pinning reduces surprise breakage. ([n8n Docs][13])

## Step 10: Start n8n

Start it with the current Compose command:

```bash
cd ~/n8n-compose
docker compose up -d
docker compose ps
docker compose logs -f
```

Current Docker and n8n docs use `docker compose`, not `docker-compose`. ([Docker Documentation][14])

## Step 11: Verify the instance

n8n’s monitoring docs expose `/healthz`, where `200` means the instance is reachable, and `/healthz/readiness`, where `200` means the DB is connected and migrated. ([n8n Docs][15])

Check:

```bash
curl -I https://n8n.example.com/healthz
curl -I https://n8n.example.com/healthz/readiness
```

Then open `https://n8n.example.com` in a browser and create the owner account. n8n’s Compose docs say the final result should be reachable via **secure HTTPS, not plain HTTP**. ([n8n Docs][1])

## Step 12: Backups

n8n’s Compose docs say the `n8n_data` volume stores the SQLite database file and encryption key. If you disable snapshots entirely, recovery will be much harder. At minimum, back up the disk or the volume regularly. ([n8n Docs][1])

---

# Option B — Minimal private/admin-only setup on port 5678

This version is only for **private access** from your own IP. It is **not** appropriate if you want third-party webhook services to reach n8n, because n8n says webhook-based triggers need the instance reachable from the web. ([n8n Docs][16])

## VM choices

Use the same free-tier-friendly Compute Engine choices:

* `e2-micro`
* region `us-east1` / `us-central1` / `us-west1`
* Ubuntu 24.04 LTS x86/64, amd64 noble
* 30 GB standard persistent disk. ([Google Cloud Documentation][2])

Assign a network tag like `n8n-admin`. You can use an ephemeral external IP here if you accept that it may change after stop/start. If you want the URL to stay stable, use a static IP and accept current external-IP pricing. ([Google Cloud Documentation][4])

## Firewall rule

Create a rule that targets only that VM tag and only your IP on port 5678. Google’s firewall docs say if you omit the target, the rule can apply to all instances in the network, so keep the target tag. ([Google Cloud Documentation][7])

```bash
gcloud compute firewall-rules create n8n-admin-5678 \
  --network=default \
  --action=ALLOW \
  --direction=INGRESS \
  --source-ranges=YOUR_PUBLIC_IP/32 \
  --target-tags=n8n-admin \
  --rules=tcp:5678
```

## Docker install

Use the same current Docker install flow from Option A. The Docker convenience script and legacy `docker-compose` package are still the wrong choices for a current long-running setup. ([Docker Documentation][9])

## Minimal compose file

For this admin-only version, you do not need Traefik or a domain. n8n’s deployment variables default to `N8N_PORT=5678`, `N8N_PROTOCOL=http`, and `N8N_HOST=localhost`, so for remote IP-based access you should set host/protocol explicitly. `WEBHOOK_URL` is documented for reverse-proxy cases, so I would omit it here. ([n8n Docs][3])

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:2.10.4
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
      - N8N_HOST=YOUR_VM_IP
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - GENERIC_TIMEZONE=America/Toronto
      - TZ=America/Toronto
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

Start it with:

```bash
docker compose up -d
docker compose ps
docker compose logs -f
```

Then browse to `http://YOUR_VM_IP:5678`. This can work for private admin use, but it is not the path n8n recommends for exposed production access, and secure cookies default to HTTPS-only in the security env var docs. ([n8n Docs][17])

---

# What I changed from your original plan

These are the exact corrections I made:

* **Ubuntu 24.04 LTS** instead of 22.04 as the current Google example image. 22.04 is still supported by Docker, but 24.04 is the current example in Google’s VM docs. ([Google Cloud Documentation][18])
* **Docker apt repo + Compose plugin** instead of `get.docker.com` + `apt install docker-compose`. ([Docker Documentation][9])
* **`docker compose`** instead of `docker-compose`. ([Docker Documentation][14])
* **Explicit targeted firewall rules** instead of a generic `gcloud compute firewall-rules create` with no target. ([Google Cloud Documentation][7])
* **No claim that static IP is free while attached**; current Google pricing says external IPv4 is billed. ([Google Cloud][19])
* **Split “private admin-only” from “public webhook” mode**, because those are different networking goals. ([n8n Docs][16])
* **Current n8n env vars and registry path** from the official docs. ([n8n Docs][11])
* **Pinned n8n version** to the current stable `2.10.4` to reduce upgrade surprises. ([n8n Docs][13])

If you want, I can turn this into a polished copy-paste guide with either:
**“private admin-only”** or **“public HTTPS + webhooks”** as the single final version.

[1]: https://docs.n8n.io/hosting/installation/server-setups/docker-compose/ "Docker Compose | n8n Docs  "
[2]: https://docs.cloud.google.com/free/docs/free-cloud-features "Free Google Cloud features and trial offer  |  Google Cloud Free Program  |  Google Cloud Documentation"
[3]: https://docs.n8n.io/hosting/configuration/environment-variables/deployment/?utm_source=chatgpt.com "Deployment environment variables"
[4]: https://docs.cloud.google.com/compute/docs/instances/suspend-stop-reset-instances-overview "Suspend, stop, or reset Compute Engine instances  |  Google Cloud Documentation"
[5]: https://docs.cloud.google.com/vpc/docs/reserve-static-external-ip-address?utm_source=chatgpt.com "Reserve a static external IP address | Virtual Private Cloud"
[6]: https://docs.cloud.google.com/compute/docs/tutorials/basic-webserver-apache?utm_source=chatgpt.com "Running a basic Apache web server | Compute Engine"
[7]: https://docs.cloud.google.com/vpc/docs/add-remove-network-tags "Add network tags  |  Virtual Private Cloud  |  Google Cloud Documentation"
[8]: https://docs.cloud.google.com/firewall/docs/using-firewalls "Use VPC firewall rules  |  Cloud Next Generation Firewall  |  Google Cloud Documentation"
[9]: https://docs.docker.com/engine/install/ubuntu/ "Ubuntu | Docker Docs"
[10]: https://docs.n8n.io/hosting/installation/server-setups/docker-compose/?utm_source=chatgpt.com "Docker Compose | n8n Docs"
[11]: https://docs.n8n.io/hosting/installation/docker/ "Docker | n8n Docs  "
[12]: https://docs.n8n.io/hosting/securing/set-up-ssl/ "Set up SSL | n8n Docs "
[13]: https://docs.n8n.io/release-notes/?utm_source=chatgpt.com "Release notes | n8n Docs"
[14]: https://docs.docker.com/compose/install/linux/ "Plugin | Docker Docs"
[15]: https://docs.n8n.io/hosting/logging-monitoring/monitoring/ "Monitoring | n8n Docs  "
[16]: https://docs.n8n.io/hosting/installation/docker/?utm_source=chatgpt.com "Docker | n8n Docs"
[17]: https://docs.n8n.io/hosting/configuration/environment-variables/security/ "Security environment variables | n8n Docs "
[18]: https://docs.cloud.google.com/compute/docs/create-linux-vm-instance?utm_source=chatgpt.com "Create a Linux VM instance in Compute Engine"
[19]: https://cloud.google.com/vpc/pricing "Virtual Private Cloud pricing | Google Cloud"
