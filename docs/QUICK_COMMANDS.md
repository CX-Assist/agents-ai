# Quick Commands — Agents on EC2

Crucial commands you'll run regularly. For the full reference, see [RUNBOOK in the deployment thread](#).

---

## Connect to the EC2 server

From your local Windows PowerShell:

```powershell
ssh -i "C:\Users\nites\Downloads\agents.pem" ubuntu@18.168.195.191
```

After connecting, your prompt becomes:
```
ubuntu@ip-172-31-38-146:~$
```

All commands below run **inside this SSH session**.

---

## Generate / rotate the admin invite URL

Run this whenever you need a fresh bootstrap admin URL (e.g., share with Paul, or replace a leaked one):

```bash
docker exec -it agents-server-1 pnpm agentsai auth bootstrap-ceo
```

The output URL is single-use. **Share via Slack DM / 1Password — never plain chat or email.**

---

## Common server operations

All commands run from `/home/ubuntu/agents`:

```bash
cd /home/ubuntu/agents
```

### View live logs

```bash
docker compose logs -f server
```

`Ctrl+C` to exit logs (server keeps running).

### Restart the Agents server

```bash
docker compose restart server
```

### Check container status

```bash
docker compose ps
```

Both `agents-server-1` and `agents-db-1` should show `Up (healthy)`.

### Stop / start everything

```bash
docker compose down       # stop
docker compose up -d      # start (background)
```

### Health check (internal)

```bash
curl http://localhost:3100/api/health
```

Should return: `{"status":"ok",...}`

### Health check (public)

From anywhere:

```bash
curl -I https://agents.cx-assist.io
```

Should return: `HTTP/2 200`

---

## After code changes (deploy update)

```bash
cd /home/ubuntu/agents
git pull origin main      # or your active branch
docker compose down
docker compose build --no-cache
docker compose up -d
docker compose logs -f server   # watch startup
```

---

## Reverse proxy (Caddy)

```bash
sudo systemctl status caddy --no-pager     # status
sudo systemctl restart caddy               # restart
sudo journalctl -u caddy -n 50 --no-pager  # logs
```

Caddyfile lives at `/etc/caddy/Caddyfile`.

---

## Database backup

```bash
cd /home/ubuntu/agents
docker compose exec -T db pg_dump -U agents agents > backup-$(date +%Y%m%d).sql
```

To restore:

```bash
docker compose exec -T db psql -U agents agents < backup-YYYYMMDD.sql
```

---

## Reminders

- Container is bound to `127.0.0.1:3100` only — **never expose port 3100 publicly.**
- All public traffic goes through Cloudflare → Caddy → localhost:3100.
- `.env` is gitignored; never commit it.
- Caddy auto-renews TLS certs — no manual cert work.
- If you see the server log warning `Agent JWT  found ... but not loaded`, it's for AI agent processes only and doesn't affect SSO / login.

---

## Troubleshooting checklist

| Symptom | First thing to check |
|---|---|
| `https://agents.cx-assist.io` returns 502 | `docker compose ps` — is server up? |
| `https://agents.cx-assist.io` returns 521 | `sudo systemctl status caddy` — is Caddy up? |
| `https://agents.cx-assist.io` returns 525/526 | Cloudflare SSL mode (Kumar / Cloudflare dashboard) |
| SSO from cx-assist shows login form on Agents | `JWT_SECRET` mismatch between cx-assist `.env` and Agents `.env` |
| Server logs show repeated `ERROR` or `FATAL` | `docker compose logs --tail 100 server` |

---

## Server details

- **Public URL:** https://agents.cx-assist.io
- **EC2 IP:** 18.168.195.191
- **SSH user:** ubuntu
- **Working directory:** /home/ubuntu/agents
- **Repo:** https://github.com/CX-Assist/agents-ai
