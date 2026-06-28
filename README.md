<div align="center">
  <h1>Tunaar</h1>
  <p><strong>IPTV for Plex that just works.</strong><br>
  A single-container HDHomeRun-emulating bridge that turns any IPTV M3U playlist
  into Live TV for Plex, Emby &amp; Jellyfin — with XMLTV guide data and reliable,
  ffmpeg-remuxed streaming.</p>
  <p><strong>Version 0.8.2</strong> · Image: <code>ghcr.io/munerisuk/tunaar:latest</code></p>
  <p>📦 <a href="#install">Install</a> · 🔑 <a href="#licensing">Licensing</a> · 🛟 <a href="#support">Support</a></p>
</div>

---

## What is Tunaar?

Tunaar emulates a Silicondust **HDHomeRun** network tuner so **Plex** (and Emby /
Jellyfin) can consume IPTV M3U playlists — and a real HDHomeRun — as Live TV,
with a unified XMLTV guide and ffmpeg-backed remuxing for reliable playback.

It's a **neutral tuner bridge**: it provides no channels itself. You supply the
playlists and/or a physical HDHomeRun.

## Key features

- 📺 **HDHomeRun emulation** — Plex, Emby & Jellyfin auto-discover it as a network tuner (UDP 65001).
- 🔗 **Merge IPTV + a real HDHomeRun** — many M3U playlists and an OTA tuner behind one device, collision-free numbering.
- 🗓️ **Unified XMLTV guide** — auto-detected from playlists and merged, with name-matching and manual EPG mapping.
- 🩹 **Plex duplicate-description fix** — optional guide tweak so Plex keeps a separate description per airing (clean titles, date-based episode numbers).
- 🎞️ **Reliable streaming** — ffmpeg remux with real tuner slots (concurrency cap + release), plus direct/redirect modes.
- ⚡ **One-click country playlists** — free [iptv-org](https://github.com/iptv-org/iptv) lists for UK, US, Canada, France, Germany, Spain and more.
- 🎛️ **Web dashboard** — manage sources, guide, groups and tuners from a branded UI; no config files needed.
- 🔄 **Auto-update** — optional Watchtower companion, scoped to Tunaar only.

## Requirements

- **Docker** on an always-on host — a NAS (QNAP / Synology / Unraid), mini-PC, or home server.
- **Plex Media Server** (or Emby / Jellyfin) on the same LAN.
- An **IPTV playlist URL** and/or a **real HDHomeRun** — or start empty and add sources from the dashboard.
- A few hundred MB of disk for the image; a small persistent volume for config.

> Tunaar runs best with **host networking** so Plex's HDHomeRun auto-discovery works. On hosts without it, use bridge mode and add the tuner by IP.

## Install

Tunaar is distributed as a Docker image — there's nothing to clone or build.

### Option A — `docker run` (recommended)

```bash
docker run -d --name tunaar --restart unless-stopped \
  --network host \
  -v tunaar-config:/config \
  ghcr.io/munerisuk/tunaar:latest
```

On a host without host networking, swap `--network host` for `-p 5004:5004`.
Then open `http://<host-ip>:5004` and run the setup wizard.

### Option B — Docker Compose

```yaml
services:
  tunaar:
    image: ghcr.io/munerisuk/tunaar:latest
    container_name: tunaar
    restart: unless-stopped
    network_mode: host
    environment:
      - TUNAAR_PLAYLIST=https://your/playlist.m3u   # optional
    volumes:
      - tunaar-config:/config
volumes:
  tunaar-config:
```

```bash
docker compose up -d
```

> **Persist your config:** always mount `/config` to a named volume or bind path, or your sources, guide and groups reset when the container is recreated.

## Updating

**Hands-off — Watchtower (recommended).** Scope it to only the `tunaar` container:

```bash
docker run -d --name watchtower-tunaar --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower --cleanup --interval 86400 tunaar
```

**One-click from the dashboard.** Add `-e TUNAAR_SELF_UPDATE=1` and
`-v /var/run/docker.sock:/var/run/docker.sock` to your run/compose, then use
**Console → Update**. (Both grant Docker root-equivalent access — opt in
deliberately.) Otherwise just pull the new image and recreate the container.

## Add the tuner in Plex

1. **Settings → Live TV & DVR → Set up Plex DVR.**
2. Tunaar answers HDHomeRun discovery, so it should appear automatically. If not, choose **"Enter its network address manually"** and enter `<host-ip>:5004`.
3. For the guide, choose **"Have an XMLTV file?"** and point Plex at `http://<host-ip>:5004/epg.xml`.

Emby / Jellyfin: add an **M3U Tuner** at `…/lineup.json` and an **XMLTV** guide at `…/epg.xml`.

## Licensing

Tunaar is **proprietary software** — © 2026 Muneris Management Ltd, all rights
reserved. There's a **30-day free trial** (full features, no key), after which
**channels keep streaming** and a licence unlocks the premium extras and removes
the reminder.

- **£16.99 / year** or a one-off **£49.99 lifetime**
- Buy at **<https://muneris.lemonsqueezy.com/>**

## Support

Questions or something looks off? Email **<info@muneris.co.uk>**.

---

<sub>Tunaar is a neutral tuner bridge — it provides no channels itself. Provider
presets point at third-party community lists, which are the responsibility of
their respective operators.</sub>
