# Deploy n8n on GCP Free Tier (with Neon Postgres)

## What you're building and why

By the end of this guide, you'll have your own **n8n automation server** running at a public URL like `https://yourname.aidilab.dev`. This gives you a platform to:

- **Build automation workflows** — connect APIs, move data between services, and automate repetitive tasks using n8n's visual workflow editor
- **Receive webhooks from third-party services** — Stripe, GitHub, Slack, Google, and hundreds of others can send real-time events to your n8n instance, triggering workflows automatically
- **Integrate with AI tools via MCP** — n8n has built-in support for the Model Context Protocol (MCP), so AI agents like Claude Desktop or Cursor can discover and call your workflows as tools, and your n8n workflows can call external MCP servers
- **Have a portfolio-ready project** — your instance runs on infrastructure you set up yourself, at a professional URL, demonstrating real DevOps and automation skills

### Why so many components?

This setup has several moving parts. Here's what each one does and why it's needed:

| Component | What it is | Why you need it |
|-----------|-----------|-----------------|
| **GCP (Google Cloud)** | Cloud provider that hosts your virtual machine | Gives you a server running 24/7 on Google's infrastructure, with a free-tier eligible VM |
| **Static IP** | A permanent public IP address for your VM | Without it, your IP changes every time the VM restarts, breaking your domain |
| **DNS (Cloudflare)** | Maps `yourname.aidilab.dev` to your VM's IP | So browsers (and webhook senders) can find your server by name instead of IP address |
| **Traefik** | A reverse proxy that handles HTTPS | Automatically gets and renews your SSL certificate from Let's Encrypt. Third-party services require HTTPS for webhooks |
| **n8n** | The automation platform itself | Where you build and run workflows — this is the whole point |
| **Neon Postgres** | A cloud-hosted database | Stores your workflows, credentials, and execution history. Hosted externally because the VM only has 1 GB of RAM |

### How they connect

```
Browser / Webhook sender
        ↓
   yourname.aidilab.dev  (DNS resolves to your static IP)
        ↓
   Traefik on port 443  (handles HTTPS, forwards to n8n)
        ↓
   n8n on port 5678  (runs workflows, processes webhooks)
        ↓
   Neon Postgres  (stores everything)
```

---

## Technical details

**Stack:** GCP `e2-micro` VM → Traefik (HTTPS) → n8n `2.10.4` → Neon Postgres

**Why Neon instead of local Postgres?** The `e2-micro` has 1 GB RAM. Running n8n + Traefik + Postgres on that will cause out-of-memory kills. Neon moves the database off the VM entirely. It's one connection string.

**Cost:** The VM and 30 GB disk are free-tier eligible. The static IPv4 address is **not free** — expect ~$3–4/month. Neon's free tier covers the database.

**Self-hosted vs n8n Cloud:** n8n offers a paid cloud service at [n8n.io](https://n8n.io) where they host everything for you. What we're doing here is **self-hosting** — running the same software yourself on your own server, for free. These are completely separate. If you already have (or had) an n8n Cloud account, it has no connection to your self-hosted instance. Different login, different database, different workflows. When you open your self-hosted URL you'll create a brand new account from scratch. No conflicts.

---

## Before you start

You will need:

- A **Google account** (for GCP and Cloud Shell)
- A **credit card** to enable GCP billing (you won't be charged beyond ~$3–4/month for the static IP if you follow these instructions)
- A **Neon account** (free — sign up at [neon.tech](https://neon.tech))
- Your assigned **aidilab.dev subdomain** — this is your name (email me your external IP address along with your name once you've completed Step 3 below)
- A **safe place to save passwords** (a password manager or secure note — you'll need to save an encryption key)
- About **30–45 minutes**

---

## Step 1 — Create your GCP project (browser)

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Click the project dropdown at the top → **New Project**
3. Name it something like `n8n-prod` and click **Create**
4. **Write down the Project ID** shown below the name field — you'll need it. It's often the same as the name but may have a number suffix like `n8n-prod-443012`
5. After creation, select that project from the dropdown
6. Go to **Billing** (left sidebar) → link a billing account to this project
7. Optional but smart: go to **Billing → Budgets & Alerts** → create a budget for $5/month so you get email warnings

> **Common error:** `PERMISSION_DENIED` when enabling APIs means your `PROJECT_ID` is wrong. GCP almost always adds a number suffix to your project name (e.g. `n8n-prod` becomes `n8n-prod-123456`). Run `gcloud projects list` in Cloud Shell to see your actual project IDs, and use the exact value from the `PROJECT_ID` column.

---

## Step 2 — Create your Neon database (browser)

1. Go to [neon.tech](https://neon.tech) and sign up / log in
2. Click **New Project**
3. Pick a name (e.g. `n8n-db`)
4. For **Region**, pick the one closest to your GCP VM. If your VM will be in `us-east1`, pick **AWS us-east-1** (Neon runs on AWS, but cross-cloud latency within the same geographic area is fine — we're talking 5–10ms)
5. After creation, click the **Connect** button in the dashboard
6. You'll see a "Connect to your database" dialog with a connection string. The **Connection pooling** toggle (green switch) is ON by default — **click it to turn it off**. n8n runs migrations at startup and needs a direct connection for that. You'll notice `-pooler` disappears from the hostname when you toggle it off
7. Click **Show password** so the asterisks become the real password
8. Copy the full connection string. It should look like:
   ```
   postgresql://neondb_owner:YOURPASSWORD@ep-something-123456.us-east-1.aws.neon.tech/neondb?sslmode=require
   ```
   Make sure there's no `-pooler` in the hostname
9. Paste it somewhere safe (a text file, a note). You'll need it in Step 5

---

## Step 3 — Set up the VM (Cloud Shell)

Open **Cloud Shell** from the GCP console (the `>_` icon at the top right).

**First, find your actual project ID.** Paste this:

```bash
gcloud projects list
```

Look for the row with your n8n project. The value you need is in the `PROJECT_ID` column — it usually has a number suffix (e.g. `n8n-prod-489817`, not just `n8n-prod`).

**Now edit the values below** using that project ID, then paste the whole block.

> **Important:** Step 3 is split into several code blocks with explanations between them. Paste them one at a time **in the same Cloud Shell session**. The variables you set in the first block (like `$REGION` and `$STATIC_IP`) carry forward to later blocks — but only if you stay in the same session. If Cloud Shell disconnects, see the Troubleshooting section at the bottom.

```bash
# ===== EDIT THESE — every single one =====
PROJECT_ID="your-project-id-here"  # paste YOUR project ID from the output above
REGION="us-east1"                  # us-east1, us-central1, or us-west1 for free tier
ZONE="us-east1-b"                  # a zone inside your region
VM_NAME="n8n-server"               # name for the VM, can be anything
DOMAIN="yourname.aidilab.dev"  # use YOUR name, e.g. "aasi.aidilab.dev" or "shah.aidilab.dev"
# ==========================================

gcloud config set project "$PROJECT_ID"
gcloud services enable compute.googleapis.com

# --- Check: did it work? ---
echo "Active project: $(gcloud config get-value project)"
# You should see your project ID printed back. If you see an error, your PROJECT_ID is wrong.
```

> **What's a static IP?** Normally, when you stop and restart a VM, GCP gives it a new IP address. That would break your DNS record (which points your domain at a specific IP). A "static IP" is a reserved address that stays the same no matter what. It costs ~$3–4/month even when the VM is running.

```bash
# Reserve a static IP (see explanation above)
gcloud compute addresses create n8n-ip --region="$REGION" 2>/dev/null || true
STATIC_IP=$(gcloud compute addresses describe n8n-ip --region="$REGION" --format='get(address)')

# --- Check: what IP did you get? ---
echo "Static IP reserved: $STATIC_IP"
# You should see an IP like 34.75.123.45. Write it down — you'll need it for DNS in Step 4.
```

> **What are firewall rules?** By default, GCP blocks ALL incoming traffic to your VM. Nothing can reach it from the internet. But n8n is a web app — browsers need to connect to it. So we open exactly two ports:
>
> - **Port 80 (HTTP)** — only used to redirect visitors to HTTPS
> - **Port 443 (HTTPS)** — the encrypted connection where n8n is served
>
> `source-ranges=0.0.0.0/0` means "allow from any IP on the internet." This sounds scary, but it's how every public website works — Google.com, GitHub, Wikipedia all allow `0.0.0.0/0` on ports 80 and 443.
>
> `target-tags=n8n-web` means these rules only apply to VMs in **your** project that have the tag `n8n-web`. A random person on the internet cannot add this tag — only people with editor/owner access to your GCP project can tag VMs.
>
> **What's NOT exposed:** SSH (port 22) is handled separately by GCP and requires your Google credentials. n8n's internal port (5678) is bound to localhost only, so it can't be reached from the internet — only through Traefik. The real security layer is your n8n login password (more on this in the Security section at the bottom).

```bash
# Open ports 80 and 443 for VMs tagged "n8n-web" (see explanation above)
gcloud compute firewall-rules create allow-http  --allow=tcp:80  --target-tags=n8n-web --source-ranges=0.0.0.0/0 2>/dev/null || true
gcloud compute firewall-rules create allow-https --allow=tcp:443 --target-tags=n8n-web --source-ranges=0.0.0.0/0 2>/dev/null || true

# --- Check: are the firewall rules there? ---
gcloud compute firewall-rules list --filter="name:(allow-http allow-https)" --format="table(name, allowed, targetTags)"
# You should see two rules: allow-http (tcp:80) and allow-https (tcp:443), both targeting n8n-web.
```

> **What does the VM creation command do?** It creates a virtual machine (a computer in the cloud) with:
>
> - `e2-micro` — the smallest (free-tier) machine: 1 GB RAM, shared CPU
> - `ubuntu-2404-lts` — Ubuntu Linux 24.04, a stable operating system
> - `30GB pd-standard` — 30 GB hard drive (max free-tier size)
> - `tags=n8n-web` — links this VM to the firewall rules we just created
> - `address=...` — attaches the static IP so the VM is reachable at that address

```bash
# Create the VM (see explanation above)
gcloud compute instances create "$VM_NAME" \
  --zone="$ZONE" \
  --machine-type=e2-micro \
  --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=30GB \
  --boot-disk-type=pd-standard \
  --tags=n8n-web \
  --address="$STATIC_IP"

# --- Check: is the VM running? ---
gcloud compute instances list --filter="name=$VM_NAME" --format="table(name, zone, machineType, status, networkInterfaces[0].accessConfigs[0].natIP)"
# You should see your VM with status RUNNING and the static IP in the last column.

echo ""
echo "=========================================="
echo "  SAVE THESE VALUES"
echo "=========================================="
echo "  Project ID:  $PROJECT_ID"
echo "  VM name:     $VM_NAME"
echo "  Zone:        $ZONE"
echo "  Static IP:   $STATIC_IP"
echo "  Domain:      $DOMAIN"
echo "=========================================="
echo ""
echo "NEXT STEP:"
echo "  Send your instructor this static IP: $STATIC_IP"
echo "  Your subdomain is your name (already set in DOMAIN above)."
echo "  Your instructor will create the DNS record for you."
echo "  Your n8n URL will be: https://$DOMAIN"
```

If this fails with a permission error, double-check that `PROJECT_ID` matches exactly what's shown in the GCP console.

---

## Step 4 — Get your domain set up

> **What is DNS?** DNS (Domain Name System) translates human-readable names like `aasi.aidilab.dev` into IP addresses like `34.26.163.45`. Without DNS, browsers have no way to find your VM — they only understand IP addresses. A DNS "A record" is an entry that says "this domain name points to this IP address."

**What's happening here:** Your instructor owns the domain `aidilab.dev` and will create a subdomain for you using your **name**. For example, if your name is Aasi, your n8n URL will be `https://aasi.aidilab.dev`.

**What you need to do:**

1. Send your instructor your **static IP** from Step 3 (the number printed at the end, like `34.26.163.45`) and your **name** this will be in your server name.
2. Wait for your instructor to confirm your DNS record is created

**Verify it's working:**

```bash
# Run this in Cloud Shell — replace with YOUR name
nslookup yourname.aidilab.dev
# Look for "Address:" in the output — it should show your static IP from Step 3.
# If you see "NXDOMAIN" or "can't find", wait a few minutes and try again.
# If it still doesn't work after 15 minutes, ask your instructor to double-check.
```

When you see your static IP in the answer, move on.

> **STOP — do not continue to Step 5 until `nslookup` returns your static IP.** If you skip ahead, Traefik won't be able to get your HTTPS certificate and you'll spend time debugging something that isn't actually broken — it's just not ready yet.

---

## Step 5 — Install Docker and deploy n8n (VM)

SSH into the VM from Cloud Shell. Use the VM name and zone you set in Step 3:

```bash
gcloud compute ssh n8n-server --zone=us-east1-b
```

(If you changed `VM_NAME` or `ZONE` in Step 3, adjust this command to match. If it asks about SSH keys, just hit Enter to accept defaults.)

Once you're in, verify you're on the VM:

```bash
# --- Check: are you on the VM? ---
hostname
# You should see "n8n-server" (or whatever you named your VM), NOT "cloudshell".
# If you still see "cloudshell", the SSH didn't work — try the command again.

whoami
# You should see your Google username. This is the user you'll run everything as.
```

Now paste this block inside the VM. It installs Docker, creates a swap file, and verifies everything works.

> **What is Docker?** Docker lets you run applications in "containers" — isolated packages that include the app and everything it needs. Instead of manually installing n8n, Traefik, Node.js, etc., you just tell Docker "run these containers" and it handles the rest. This is how n8n officially recommends deploying.

> **What is swap?** The `e2-micro` VM only has 1 GB of RAM. When programs need more memory than that, Linux can use disk space as overflow memory — that's swap. It's slower than real RAM but prevents the system from crashing. We add 2 GB of swap as a safety net.

```bash
set -euo pipefail

# --- Install Docker ---
# These commands add Docker's official package repository to Ubuntu,
# then install Docker and the Compose plugin (which lets us run multi-container apps).
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker

# --- Add swap (e2-micro only has 1 GB RAM) ---
# Creates a 2 GB file on disk that Linux can use as overflow memory.
sudo fallocate -l 2G /swapfile && sudo chmod 600 /swapfile
sudo mkswap /swapfile && sudo swapon /swapfile
grep -q '/swapfile' /etc/fstab || echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# --- Check: is everything installed? ---
echo ""
echo "Docker version:"
docker --version
# You should see something like "Docker version 27.x.x"

echo ""
echo "Docker Compose version:"
sudo docker compose version
# You should see something like "Docker Compose version v2.x.x"

echo ""
echo "Memory and swap:"
free -h
# Look at the Swap row — it should show 2.0G, not 0.
# The Mem row will show about 1 GB total — that's normal for e2-micro.
```

Now create the n8n project directory and config. This block generates two files:

- **`.env`** — stores all your settings (domain, passwords, database connection, encryption key)
- **`compose.yaml`** — tells Docker which containers to run and how they connect

> **Reminder:** This is where the architecture from the top of the guide comes together. The `compose.yaml` defines Traefik and n8n as Docker containers, and the `.env` feeds in your domain, database, and encryption settings. You never configure Traefik yourself — it's all handled by these two files.

**Edit the values below first**, then paste the whole block:

```bash
set -euo pipefail
mkdir -p ~/n8n-compose/local-files && cd ~/n8n-compose

# ===== EDIT THESE =====
N8N_DOMAIN="yourname.aidilab.dev"    # same as DOMAIN from Step 3, e.g. "aasi.aidilab.dev"
SSL_EMAIL="you@example.com"              # any email you check — Let's Encrypt sends a warning if your HTTPS cert is about to expire (rare, since Traefik auto-renews). Not linked to Neon, Cloudflare, or GCP.
TZ="America/Toronto"                     # your timezone
export NEON_URL="paste-your-entire-neon-connection-string-here"
                                         # Replace everything between the quotes with the FULL string
                                         # you copied from Neon in Step 2. Don't edit parts of it —
                                         # just paste the whole thing. It already has your password in it.
# =======================

# --- Parse the Neon URL into individual database settings ---
# (n8n needs host, port, database, user, and password as separate values)
eval "$(python3 -c "
import os, urllib.parse as u
p = u.urlparse(os.environ['NEON_URL'])
print(f'DB_HOST={u.unquote(p.hostname or \"\")}')
print(f'DB_PORT={p.port or 5432}')
print(f'DB_NAME={u.unquote((p.path or \"\").lstrip(\"/\"))}')
print(f'DB_USER={u.unquote(p.username or \"\")}')
print(f'DB_PASS={u.unquote(p.password or \"\")}')
")"
unset NEON_URL

# --- Check: did the Neon URL parse correctly? ---
echo "Database host: $DB_HOST"
echo "Database name: $DB_NAME"
echo "Database user: $DB_USER"
# You should see values like:
#   Database host: ep-spring-sound-abc123.us-east-1.aws.neon.tech
#   Database name: neondb
#   Database user: neondb_owner
# If any of these are blank, your Neon connection string is wrong.

ENCRYPTION_KEY=$(openssl rand -hex 32)

# --- Write .env ---
cat > .env <<EOF
N8N_DOMAIN=${N8N_DOMAIN}
SSL_EMAIL=${SSL_EMAIL}
TZ=${TZ}
N8N_VERSION=2.10.4
N8N_ENCRYPTION_KEY=${ENCRYPTION_KEY}
DB_HOST=${DB_HOST}
DB_PORT=${DB_PORT}
DB_NAME=${DB_NAME}
DB_USER=${DB_USER}
DB_PASS=${DB_PASS}
EOF
chmod 600 .env

# --- Write compose.yaml ---
cat > compose.yaml <<'COMPOSE'
services:
  traefik:
    image: traefik:latest
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
    image: docker.n8n.io/n8nio/n8n:${N8N_VERSION}
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    labels:
      - traefik.enable=true
      - traefik.http.routers.n8n.rule=Host(`${N8N_DOMAIN}`)
      - traefik.http.routers.n8n.tls=true
      - traefik.http.routers.n8n.entrypoints=web,websecure
      - traefik.http.routers.n8n.tls.certresolver=mytlschallenge
      - traefik.http.middlewares.n8n.headers.SSLRedirect=true
      - traefik.http.middlewares.n8n.headers.STSSeconds=315360000
      - traefik.http.middlewares.n8n.headers.browserXSSFilter=true
      - traefik.http.middlewares.n8n.headers.contentTypeNosniff=true
      - traefik.http.middlewares.n8n.headers.forceSTSHeader=true
      - traefik.http.middlewares.n8n.headers.STSIncludeSubdomains=true
      - traefik.http.middlewares.n8n.headers.STSPreload=true
      - traefik.http.routers.n8n.middlewares=n8n@docker
    environment:
      - N8N_HOST=${N8N_DOMAIN}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - N8N_PROXY_HOPS=1
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - N8N_RUNNERS_ENABLED=true
      - NODE_ENV=production
      - WEBHOOK_URL=https://${N8N_DOMAIN}/
      - GENERIC_TIMEZONE=${TZ}
      - TZ=${TZ}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=${DB_HOST}
      - DB_POSTGRESDB_PORT=${DB_PORT}
      - DB_POSTGRESDB_DATABASE=${DB_NAME}
      - DB_POSTGRESDB_USER=${DB_USER}
      - DB_POSTGRESDB_PASSWORD=${DB_PASS}
      - DB_POSTGRESDB_SSL_ENABLED=true
      - DB_POSTGRESDB_SSL_REJECT_UNAUTHORIZED=false
      - EXECUTIONS_DATA_PRUNE=true
      - EXECUTIONS_DATA_MAX_AGE=168
      - EXECUTIONS_DATA_PRUNE_MAX_COUNT=5000
    volumes:
      - n8n_data:/home/node/.n8n
      - ./local-files:/files

volumes:
  n8n_data:
  traefik_data:
COMPOSE

echo ""
echo "Config written. Now start it with:"
echo "  cd ~/n8n-compose && sudo docker compose up -d"
```

After the script finishes, verify what it created:

```bash
# --- Check: what did the script write? ---
echo "=== Your .env file (passwords hidden) ==="
grep -v 'PASS\|ENCRYPTION' ~/n8n-compose/.env
# You should see your domain, email, timezone, n8n version, and DB host.
# Passwords and the encryption key are hidden here for safety, but they're in the file.

echo ""
echo "=== Files in ~/n8n-compose ==="
ls -la ~/n8n-compose/
# You should see: .env, compose.yaml, and a local-files/ directory.
```

---

## Step 6 — Start n8n (VM)

```bash
cd ~/n8n-compose
sudo docker compose pull
sudo docker compose up -d
```

Wait 30–60 seconds for n8n to run its database migrations, then check:

```bash
# --- Check: are both containers running? ---
sudo docker compose ps
# You should see TWO services, both with status "Up":
#   n8n-compose-traefik-1   Up   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
#   n8n-compose-n8n-1       Up   127.0.0.1:5678->5678/tcp
#
# If either shows "Restarting" instead of "Up", check the logs below.

# --- Check: any errors? ---
sudo docker compose logs --tail=30
# Scroll through the output. You're looking for:
#   - Traefik: no errors, maybe a line about getting a certificate
#   - n8n: "n8n ready on 0.0.0.0, port 5678" means it started successfully
#   - If you see "ECONNREFUSED" or "password authentication failed", your Neon connection string is wrong

# --- Check: can n8n reach its own HTTPS endpoint? ---
curl -s -o /dev/null -w "HTTP status: %{http_code}\n" https://$(grep N8N_DOMAIN ~/n8n-compose/.env | cut -d= -f2)/healthz
# You should see "HTTP status: 200". 
# If you see 000 or an error, DNS may not have propagated yet, or the certificate is still being issued.
```

---

## Step 7 — Verify and create your account (browser)

Open your n8n URL in your browser — it's whatever you entered as your domain in Step 5, e.g.:

```
https://aasi.aidilab.dev
```

You should see the n8n setup screen. Create your owner account there.

**This is a one-time setup.** The first person to open this page becomes the **owner** — after that, the setup page disappears permanently. No one else can create an account by visiting your URL. The login page will still be visible to anyone, but they can't do anything without your credentials. If you ever want to add another user, you do it manually from inside n8n's settings.

**Use a strong password.** Your n8n instance is on the public internet — anyone who knows the URL can see the login page. The firewall rules allow all traffic on ports 80/443 (that's normal for a web app), so your n8n password is the main thing keeping people out. Don't use `password123`.

If you get a certificate error, wait a couple of minutes — Traefik is still getting the Let's Encrypt certificate. If it persists, check that your DNS record is correct and that you didn't leave Cloudflare's proxy enabled.

---

## Step 8 — Save your encryption key

Run this on the VM:

```bash
grep N8N_ENCRYPTION_KEY ~/n8n-compose/.env
# You should see a long hex string like: N8N_ENCRYPTION_KEY=a3f7b9c1d4e6...
# This is a 64-character random key that was auto-generated during setup.
```

**Copy that key and save it somewhere safe** (password manager, secure note).

> **Why this matters:** Every API key, token, and password you save inside n8n (your OpenAI key, Google credentials, Slack tokens, etc.) gets encrypted with this key before it's stored in the Neon database. If your VM dies and you set up a fresh one pointing at the same database, n8n can't decrypt any of those saved credentials without the original encryption key. You'd have to re-enter every single API key and token from scratch. It takes 10 seconds to save it now. It saves hours of pain later.

You may also want to save a copy of the **entire `.env` file**, not just the key. It contains your domain, database settings, and encryption key — everything needed to recover this deployment. You can view it with `cat ~/n8n-compose/.env` and copy the contents to a secure note.

---

## Updating n8n later

Edit `~/n8n-compose/.env` and change the version number, then:

```bash
cd ~/n8n-compose
sudo docker compose pull
sudo docker compose up -d
```

n8n ships new versions most weeks. Check [docs.n8n.io/release-notes](https://docs.n8n.io/release-notes/) before updating.

---

## Managing costs

**Short answer: just leave the VM running.** Here's why:

| Resource | Running | Stopped |
|----------|---------|---------|
| VM (e2-micro) | Free | Free |
| 30 GB disk | Free | Free (disk still exists) |
| Static IP | ~$3–4/month | **More expensive** — GCP charges extra for a reserved IP not attached to a running VM |

Stopping the VM doesn't save money — it actually costs more because of the static IP billing. The only real cost is the static IP, and that's charged whether the VM is on or off.

**If you want to go to true zero cost** (e.g. over a long break where you won't use n8n at all):

1. Delete the static IP:
   ```bash
   gcloud compute addresses delete n8n-ip --region=us-east1
   ```
2. Stop the VM:
   ```bash
   gcloud compute instances stop n8n-server --zone=us-east1-b
   ```

**To bring it back later**, you'd need to: reserve a new static IP (you'll get a different one), start the VM, attach the new IP, update the DNS record in Cloudflare to the new IP, wait for DNS to propagate, and let Traefik get a new certificate. That's a lot of steps to save $3–4/month.

**Recommendation:** For a semester-long class, leave everything running and accept the static IP cost. Set up that $5/month budget alert from Step 1 and don't worry about it.

---

## Troubleshooting

**"PERMISSION_DENIED" in Cloud Shell** — Your `PROJECT_ID` doesn't match your actual GCP project, or billing isn't linked. Check the project dropdown in the console.

**Cloud Shell disconnected / "You do not currently have an active account"** — Cloud Shell sessions expire after being idle. Just re-authenticate and reconnect:
```bash
gcloud auth login
gcloud config set project your-project-id-here
gcloud compute ssh your-vm-name --zone=your-zone
```
Replace `your-project-id-here`, `your-vm-name`, and `your-zone` with the values from your Step 3 "SAVE THESE VALUES" output. Your VM and everything on it is fine — nothing is lost. But if you were in the middle of Step 3 (setting up the VM from Cloud Shell), you'll need to re-paste the variable block at the top since those variables only live in your shell session.

**n8n keeps restarting** — Run `sudo docker compose logs n8n --tail=100`. Usually it's a bad database connection string. Double-check the Neon URL and make sure you used the Direct connection, not pooled.

**Certificate errors in browser** — DNS isn't pointing at the VM yet, or port 443 is blocked. Run `nslookup yourname.aidilab.dev` to verify the IP, and check that the firewall rules exist with `gcloud compute firewall-rules list`.

**Out of memory** — Make sure the swap file is active (`free -h` should show 2 GB swap). If n8n is still getting killed, you can reduce the execution pruning limits in `.env`.

---

## Security overview

Your n8n instance is on the public internet. Here's what's protecting it and what to be aware of.

### What's protected

- **Password in transit** — HTTPS encrypts everything between your browser and the server. When you type your password on the login page, no one can intercept it on the network. That's the entire point of the Traefik/Let's Encrypt setup.
- **SSH access to the VM** — Requires your Google account credentials, not just a password. Random people on the internet cannot SSH into your machine.
- **Database connection** — Neon requires SSL and is password-protected. The credentials are stored in your `.env` file on the VM, which is only readable by your user.
- **Saved credentials inside n8n** — API keys, tokens, and passwords you save in n8n workflows are encrypted at rest using your `N8N_ENCRYPTION_KEY`. Without that key, they can't be decrypted.
- **No public signup** — After the owner creates their account in Step 7, the setup page disappears. No one else can register.

### What's NOT protected (and what to know)

- **Brute force login attempts** — This setup does not have rate limiting on the login page. If someone tries thousands of passwords, nothing will block them automatically. Your defense here is a **strong password** — use 20+ characters, random, from a password manager. This is the weakest spot in the setup.
- **No two-factor authentication (2FA)** — n8n's self-hosted community edition does not support 2FA. Your password is the only barrier.
- **No IP allowlisting** — Anyone on the internet can reach your login page. A production setup might restrict access to specific IP ranges, but that's not practical for a class where students are logging in from different locations.
- **No fail2ban** — There's no automatic blocking of IPs that fail login repeatedly. This would be an improvement for a production deployment but is beyond the scope of this setup.

### Useful commands for monitoring

```bash
# Run these from the VM (SSH in first, then cd to the project directory)
cd ~/n8n-compose

# See who's hitting your server (every web request goes through Traefik)
sudo docker compose logs traefik --tail=50

# See n8n activity, including failed login attempts
sudo docker compose logs n8n --tail=50

# Watch logs in real time (Ctrl+C to stop)
sudo docker compose logs -f
```

### Changing your password

You can change your password anytime: open n8n → click your avatar (bottom left) → **Settings** → **Personal** → **Password**.

### Bottom line

For a class project, a strong password is sufficient. The main things to remember: don't reuse a password from another site, don't share your login, and save your `N8N_ENCRYPTION_KEY` somewhere safe.
