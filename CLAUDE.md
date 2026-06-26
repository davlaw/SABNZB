# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## System Overview

This is a Debian Linux server (Trixie) running several self-hosted services. The working directory is the root filesystem `/`.

**Key services:**
- **SABnzbd** — Automated Usenet binary downloader (`/opt/sabnzbd/`, Python 3)
- **OpenTelemetry Collector** — Host metrics and log forwarding to OpenObserve at `192.168.5.43:5080`
- **par2cmdline / par2cmdline-turbo** — PAR2 repair tool source code in `/home/david/`
- **Webmin** — Web-based server administration

## Service Management

```bash
# Check service status
systemctl status otel-collector
systemctl status sabnzbd@david

# View service logs
journalctl -u otel-collector -f
journalctl -u sabnzbd@david -f

# Restart services
systemctl restart otel-collector
systemctl restart sabnzbd@david
```

## SABnzbd

- **Location:** `/opt/sabnzbd/`
- **Service file:** `/etc/systemd/system/sabnzbd@.service`
- **User config:** `~/.sabnzbd/`
- **Run manually:** `python3 -OO /opt/sabnzbd/SABnzbd.py`
- **Run in background:** `python3 -OO /opt/sabnzbd/SABnzbd.py -d -f ~/.sabnzbd/sabnzbd.ini`
- **Python venv:** `/opt/sabnzbd-venv/` (required — system Python conflicts with Debian-managed packages)
- **Install/update dependencies:** `/opt/sabnzbd-venv/bin/pip install -r /opt/sabnzbd/requirements.txt -U`
- **Update SABnzbd:** `sabnzbd-update` (pulls, updates deps, restarts service)

## OpenTelemetry Collector

- **Config:** `/etc/otel-config.yaml`
- **Binary:** `/usr/local/bin/otelcol-contrib`
- **Service:** `otel-collector.service` (runs as `openobserve-agent` user)
- **Collects:** host metrics (CPU, disk, filesystem, memory, network, paging, processes) and logs from `/var/log/**log`
- **Ships to:** OpenObserve instance at `http://192.168.5.43:5080/api/default/`
- **Re-install/reconfigure:** run `/home/david/install.sh <URL> <Auth_Key>` as root

## Building par2cmdline

Both `/home/david/par2cmdline/` and `/home/david/par2cmdline-turbo/` use autotools:

```bash
cd /home/david/par2cmdline   # or par2cmdline-turbo
./automake.sh
./configure
make
make check
make install
```

`par2cmdline-turbo` is a performance-focused fork that replaces GF16/MD5/CRC32 computation with ParPar's backend and uses C++11 threads instead of OpenMP.

## Admin Scripts (`/home/david/`)

| Script | Purpose |
|--------|---------|
| `install.sh <URL> <Auth_Key>` | Install otelcol-contrib and configure for OpenObserve |
| `setup-repos.sh` | Add Webmin apt/yum repository (Debian/RHEL) |
| `upgrade.sh` | Upgrade Debian bookworm → trixie (destructive, reboots) |
