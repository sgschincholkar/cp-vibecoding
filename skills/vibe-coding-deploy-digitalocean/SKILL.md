---
name: vibe-coding-deploy-digitalocean
description: |
  Deploy projects to DigitalOcean — App Platform (PaaS), Droplets (VPS), or managed databases. Local-machine focused.
  Use when: (1) vibe-coding-ship detects DigitalOcean as the deployment target,
  (2) User says "deploy to DigitalOcean", "host on DO", "create a Droplet", "use App Platform",
  (3) User says "set up DigitalOcean", "configure DO deployment", "create DO app".
  Covers: App Platform (zero-config PaaS), Droplet (VPS with full control), and managed Postgres/MySQL.
  Returns deployment URL and status to vibe-coding-ship on completion.
---

# Vibe Coding — Deploy to DigitalOcean

Two paths: App Platform (managed) or Droplet (VPS).

## Entry Router

```
Path not specified?
→ Ask: "App Platform (managed, like Heroku) or Droplet (full VPS control)?"

App Platform → READ: ## Path A: App Platform
Droplet → READ: ## Path B: Droplet
Managed database only → READ: ## Managed Database
```

**Jump directly to the matched path. Do not read all three paths.**

---

## Path A: App Platform

```bash
# Step 1: Install + auth
brew install doctl  # or scoop install doctl (Windows) / snap install doctl (Linux)
doctl auth init     # prompts for API token (get from cloud.digitalocean.com/account/api/tokens)
```

**Step 2: Create `.do/app.yaml` spec:**

Next.js:
```yaml
name: [your-app-name]
region: nyc1
services:
  - name: web
    github:
      repo: [user]/[repo]
      branch: main
      deploy_on_push: true
    build_command: npm run build
    run_command: npm start
    environment_slug: node-js
    instance_size_slug: basic-xxs   # $5/mo cheapest
    http_port: 3000
    envs:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        value: "${db.DATABASE_URL}"
        type: SECRET
databases:
  - name: db
    engine: PG
    version: "15"
    size: db-s-1vcpu-1gb
```

Flask/Django (replace services block):
```yaml
    build_command: pip install -r requirements.txt
    run_command: gunicorn app:app --bind 0.0.0.0:8080      # Flask
    # run_command: gunicorn [project].wsgi --bind 0.0.0.0:8080  # Django
    environment_slug: python
    http_port: 8080
```

**Step 3: Deploy:**
```bash
doctl apps create --spec .do/app.yaml
APP_ID=$(doctl apps list --format ID --no-header | head -1)
doctl apps create-deployment $APP_ID   # redeploy after git push
doctl apps logs $APP_ID --type=build   # view build logs
doctl apps logs $APP_ID --type=run     # view runtime logs
```

**Update env vars:** edit `.do/app.yaml` envs section → `doctl apps update $APP_ID --spec .do/app.yaml`

---

## Path B: Droplet (VPS)

```bash
# Create
doctl compute droplet create [name] \
  --image ubuntu-22-04-x64 \
  --size s-1vcpu-1gb \        # $6/mo cheapest
  --region nyc1 \
  --ssh-keys [fingerprint]    # from: doctl compute ssh-key list

# SSH in
ssh root@[droplet-ip]

# System setup
apt update && apt upgrade -y
# Node.js: curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && apt install -y nodejs
# Python:  apt install -y python3 python3-pip python3-venv
apt install -y nginx
npm install -g pm2  # Node.js process manager
```

**Deploy app:**
```bash
git clone [repo-url] /var/www/[app-name]
cd /var/www/[app-name] && npm ci && npm run build
pm2 start npm --name "[app-name]" -- start
pm2 save && pm2 startup
```

**Nginx config** (`/etc/nginx/sites-available/[app-name]`):
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
```bash
ln -s /etc/nginx/sites-available/[app-name] /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
certbot --nginx -d yourdomain.com  # free SSL
```

**Deploy updates script:**
```bash
#!/bin/bash  # /var/www/[app-name]/deploy.sh
cd /var/www/[app-name] && git pull origin main && npm ci && npm run build && pm2 restart [app-name]
```
Run from local: `ssh root@[ip] "bash /var/www/[app-name]/deploy.sh"`

---

## Managed Database

```bash
doctl databases create [db-name] --engine pg --version 15 --size db-s-1vcpu-1gb --region nyc1
doctl databases connection [db-id] --format URI  # get connection string
doctl databases firewalls append [db-id] --rule "droplet:[droplet-id]"  # allow Droplet
```

---

## Common Errors

**App Platform build fails:** check `doctl apps logs $APP_ID --type=build`. Add `"engines": { "node": "20.x" }` to package.json for version mismatch.

**Droplet 502 Bad Gateway:** check `pm2 status` → is app running? Check `pm2 logs`. Verify nginx port matches app port.

**Cannot connect to managed DB:** add Droplet to firewall (`doctl databases firewalls append`). Use private network hostname.

---

## Output to vibe-coding-ship

```
DIGITALOCEAN DEPLOYMENT COMPLETE
Path: [App Platform | Droplet]
URL: https://[app].ondigitalocean.app (App Platform) | https://[ip] (Droplet)
```

Write to progress.txt:
```
DEPLOYMENT:
  platform: digitalocean  path: [app_platform|droplet]
  url: [url]  status: [success|failed]
```
