# 🏠 Homelab Stack

A self-hosted homelab setup using Docker Compose with Traefik as a reverse proxy, featuring a dashboard, monitoring, home automation, and a browser-based code editor.

## 📦 Services

| Service | Description | URL |
|---|---|---|
| **Traefik** | Reverse proxy & SSL termination | — |
| **Glance** | Personal dashboard (weather, DNS stats, bookmarks, monitoring) | `https://dash.YOUR_DOMAIN_NAME` |
| **Code Server** | VS Code in the browser | `https://code.YOUR_DOMAIN_NAME` |
| **Home Assistant** | Home automation platform | `https://ha.YOUR_DOMAIN_NAME` |
| **Prometheus** | Metrics collection | internal |
| **Grafana** | Metrics visualization & dashboards | `https://monitor.YOUR_DOMAIN_NAME` |

## 📁 Project Structure

```
.
├── compose.yaml              # Main Docker Compose file
├── .env                      # Environment variables (see .env.example)
├── traefik/
│   ├── dynamic.yml           # Traefik dynamic configuration (TLS, routers)
│   └── certs/                # TLS certificates (gitignored)
├── glance/
│   ├── config/
│   │   ├── glance.yml        # Main Glance config
│   │   └── home.yml          # Dashboard page layout
│   └── assets/
│       └── user.css          # Custom CSS overrides
├── codeserver/
│   ├── config/               # Code Server user data (gitignored)
│   └── .env                  # Code Server environment variables
├── homeassistant/            # Home Assistant config directory
└── monitoring/
    ├── prometheus.yml        # Prometheus scrape config
    ├── grafana.env           # Grafana environment variables (gitignored)
    └── grafana-data/         # Grafana persistent data (gitignored)
```

## 🚀 Getting Started

### Prerequisites

- Docker & Docker Compose installed (see [Installing Docker](#installing-docker) below)
- A domain name with DNS records pointing to your server
- TLS certificates for your domain (place in `traefik/certs/`)
- A [Pi-hole](https://pi-hole.net/) instance running and accessible
- An [OpenWeatherMap](https://openweathermap.org/api) API key

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
```

### 2. Installing Docker

An installation script is included to quickly set up Docker on Ubuntu. It adds the official Docker repository, installs Docker, and configures it to run without `sudo`.

```bash
chmod +x install_docker.sh
./install_docker.sh
```

> **Note:** After running the script, your user will be added to the `docker` group. Log out and back in (or start a new session) for this to take effect.

The script performs the following steps:
1. Adds the official Docker GPG key
2. Adds the Docker apt repository
3. Installs `docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin`, and `docker-compose-plugin`
4. Adds the current user to the `docker` group (so `sudo` is no longer required)

### 3. Configure environment variables

```bash
cp .env.example .env
```

Edit `.env` and fill in your values:

```env
PIHOLE_PASSWORD=your_pihole_password
OPENWEATHERMAP_API_KEY=your_openweathermap_api_key
```

Also create the Code Server env file:

```bash
cp codeserver/.env.example codeserver/.env
# Edit codeserver/.env with your preferred settings
```

### 4. Set your domain name

Replace all occurrences of `YOUR_DOMAIN_NAME` in the following files with your actual domain:

- `compose.yaml` — Traefik router rules
- `glance/config/home.yml` — Service monitor URLs & Pi-hole URL

### 5. Add TLS certificates

Place your TLS certificate and key in `traefik/certs/`:

```
traefik/certs/cert.crt
traefik/certs/cert.key
```

Make sure `traefik/dynamic.yml` references these files correctly.

### 6. Start the stack

```bash
docker compose up -d
```

## ⚙️ Configuration

### Glance Dashboard

The dashboard is configured via `glance/config/home.yml`. It includes:

- **Search bar** — DuckDuckGo search, opens in a new tab
- **Weather widget** — Powered by OpenWeatherMap (location set to Eindhoven by default, change as needed)
- **Service monitor** — Checks uptime of Pi-hole, Code Server, Home Assistant, and Grafana
- **Bookmarks** — Quick links to GitHub, YouTube, and Gmail
- **DNS stats** — Pi-hole v6 statistics

To change the weather location, edit the `location` field in `glance/config/home.yml`.

For custom styling, add CSS to `glance/assets/user.css` (clear browser cache after changes with `Ctrl+F5`).

### Monitoring (Prometheus + Grafana)

Configure Prometheus scrape targets in `monitoring/prometheus.yml`. Grafana is pre-configured to connect to Prometheus. Set your Grafana admin credentials in `monitoring/grafana.env`:

```env
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=yourpassword
```

## 🔒 Security Notes

- Keep `.env`, `codeserver/.env`, and `monitoring/grafana.env` out of version control — they are already listed in `.gitignore`
- TLS certificates are gitignored by default
- Code Server config (which may contain sensitive user data) is gitignored

## 🛑 Stopping the Stack

```bash
docker compose down
```

To also remove volumes:

```bash
docker compose down -v
```
