# Self-hosting this fork with custom extensions

This guide implements a practical deployment path for this repository using the existing Docker Compose files.

## 1) Pick your deployment mode

Base files:

- `docker-compose.yml`
- `docker-compose-local.yml`

Modes:

- **Simple (SQLite)**: run with base + local files only.
- **Production (PostgreSQL)**: add `docker-compose-db.yml`.

## 2) Prepare host and config

1. Install Docker Engine and Docker Compose v2.
2. Create a deployment directory on your host.
3. Copy these files from this repository:
	- `Docker/freshrss/example.env` -> `.env`
	- `Docker/freshrss/docker-compose.yml`
	- `Docker/freshrss/docker-compose-local.yml`
	- Optional: `Docker/freshrss/docker-compose-db.yml`
	- Optional: `Docker/freshrss/docker-compose-proxy.yml`
4. Edit `.env` and set real values:
	- `BASE_URL`
	- `SERVER_DNS`
	- `ADMIN_EMAIL`
	- `ADMIN_PASSWORD`
	- `ADMIN_API_PASSWORD`
	- `TZ`
	- If using PostgreSQL: `DB_HOST`, `DB_BASE`, `DB_USER`, `DB_PASSWORD`

## 3) Start FreshRSS

From your deployment directory:

### SQLite

```sh
docker compose -f docker-compose.yml -f docker-compose-local.yml pull
docker compose -f docker-compose.yml -f docker-compose-local.yml up -d --remove-orphans
```

### PostgreSQL

```sh
docker compose -f docker-compose.yml -f docker-compose-db.yml -f docker-compose-local.yml pull
docker compose -f docker-compose.yml -f docker-compose-db.yml -f docker-compose-local.yml up -d --remove-orphans
```

Then open `BASE_URL` in your browser and complete installation.

Persistent storage is already configured:

- `/var/www/FreshRSS/data`
- `/var/www/FreshRSS/extensions`

## 4) Custom extensions workflow

1. Put each extension in its own folder under the mounted `extensions` directory.
2. Ensure each extension includes expected files such as:
	- `extension.php`
	- `metadata.json`
3. Keep `extensions/README.md` and `extensions/.gitignore` untouched.
4. Enable and configure from **Configuration -> Extensions** in the UI.
5. Team workflow recommendation:
	- Keep extension source in git.
	- Sync/release extension folders into the mounted extensions volume.

## 5) Run official image vs custom fork

- If you only need third-party/custom extensions, keep the official image from `docker-compose.yml`.
- If you need to run customised core code from this repository:
	- Use `docker-compose-development.yml` (bind mounts this repo into `/var/www/FreshRSS`), or
	- Enable the local `build:` section in `docker-compose.yml`.

## 6) Production hardening

1. Add HTTPS reverse proxy with `docker-compose-proxy.yml`.
2. Set `SERVER_DNS` and `ADMIN_EMAIL` in `.env`.
3. Set `TRUSTED_PROXY` to your real proxy network/IP ranges.
4. Expose only required ports publicly.
5. Keep FreshRSS data paths non-public.

## 7) Operations

- **Feed refresh**: set `CRON_MIN` (or use host cron strategy).
- **Backups** (regularly):
	- `data/`
	- `extensions/`
	- external DB backups when using PostgreSQL/MySQL/MariaDB
- **Updates**:
	- pull new image(s)
	- restart stack
	- verify extension compatibility before full rollout

## 8) Go-live validation checklist

- [ ] Admin login works.
- [ ] Feed import and refresh work.
- [ ] API/mobile access works (if used).
- [ ] Custom extensions are visible and can be enabled.
- [ ] Container restart keeps data and extensions.
- [ ] Backup and restore procedure is tested.
