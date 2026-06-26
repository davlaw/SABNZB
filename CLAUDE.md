# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**GitHub:** github.com/davlaw/SABNZB

## System Overview

This is a Debian Linux server (Trixie) running several self-hosted services. The working directory is the root filesystem `/`.

**Key services:**
- **SABnzbd** — Automated Usenet binary downloader (`/opt/sabnzbd/`, Python 3)
- **par2cmdline / par2cmdline-turbo** — PAR2 repair tool source code in `/home/david/`
- **Webmin** — Web-based server administration

## Service Management

```bash
# Check service status
systemctl status sabnzbd@david

# View service logs
journalctl -u sabnzbd@david -f

# Restart service
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

## Kernel Tweaks

Tuned for SABnzbd in `/etc/sysctl.d/99-sabnzbd.conf` (applied at boot, live immediately via `sysctl -p`):

| Parameter | Value | Reason |
|-----------|-------|--------|
| `vm.swappiness` | 10 | Avoid swapping on 1.9GB RAM system |
| `net.core.rmem_max/wmem_max` | 16MB | Larger socket buffers for download throughput |
| `net.ipv4.tcp_slow_start_after_idle` | 0 | Don't restart TCP slow-start between NZB downloads |

## Admin Scripts (`/home/david/`)

| Script | Purpose |
|--------|---------|
| `setup-repos.sh` | Add Webmin apt/yum repository (Debian/RHEL) |
| `upgrade.sh` | Upgrade Debian bookworm → trixie (destructive, reboots) |

