<div align="center">
  <img src="tunaar/static/logo.svg" width="96" alt="Tunaar logo">
  <h1>Tunaar</h1>
  <p><strong>IPTV for Plex that just works.</strong><br>
  A single-container HDHomeRun-emulating bridge that turns any IPTV M3U playlist
  into Live TV for Plex, Emby & Jellyfin — with XMLTV guide data and reliable,
  ffmpeg-remuxed streaming.</p>
  <p>
    📦 <a href="docs/install.html">Docker Install Guide</a> &nbsp;·&nbsp;
    🐳 <a href="https://github.com/MunerisUK/Tunaar/pkgs/container/tunaar">Container Package</a> &nbsp;·&nbsp;
    📖 <a href="docs/user-guide.html">User Guide</a>
    &nbsp;<em>(branded HTML — open in a browser)</em>
  </p>
</div>

---

## Acceptable use — Tunaar supplies no content

**Tunaar ships empty.** It contains no channels, no streams, no playlists, no guide
data and no subscriptions, and it provides access to none. Tunaar is a neutral tuner
bridge: **you supply your own legitimate sources** — your own tuner hardware, or a
service you lawfully subscribe to — and you are solely responsible for their legality.
Using Tunaar with pirated or infringing sources, to circumvent access controls, or for
commercial retransmission is prohibited.

Full policy: **[Acceptable Use Policy](docs/acceptable-use.html)**. Tunaar also requires
every user to agree to it in-app before setup can be completed.

---

## Why Tunaar?

Tools like xTeVe are powerful but notoriously clunky — fiddly UI, tedious EPG
mapping, buffering that drops streams, and config that corrupts. Tunaar focuses
on being **robust and effortless**:

- 🎯 **One Docker container.** `docker compose up` and you're done — ffmpeg
  included.
- 📡 **Real tuner slots.** Concurrent streams are capped at `tuner_count` and
  released the moment a client disconnects — no phantom "all tuners busy".
- 🎬 **Reliable streaming.** Each channel is remuxed through ffmpeg
  (`-c copy -f mpegts`, auto-reconnect) into a clean MPEG-TS that players accept
  without the hit-and-miss buffering of bare redirects. HLS sources just work.
- 🗓️ **EPG that just works.** Tunaar auto-detects the guide URL embedded in your
  playlist (`url-tvg`) and merges in any extra XMLTV URLs you add — no manual
  channel mapping. Filtered to your lineup and served at `/epg.xml`.
- ➕ **Multiple IPTV sources.** Add as many M3U playlists as you like (from the
  dashboard or config); channels are merged with collision-free numbering.
- 🗂️ **Group control.** Pick which groups appear in Plex from a checklist;
  per-source group overrides supported.
- 🖥️ **Editable dashboard.** Add sources, set EPG URLs, and choose groups right
  in the web UI — changes persist to config, no SSH required.
- 🔎 **Auto-discovery.** Answers HDHomeRun discovery (UDP 65001) so Plex finds
  the tuner automatically — no manual IP entry.
- 🛠️ **Admin console.** A `/console` page with a live activity log (streamed via
  SSE), active-stream details, and controls to refresh, restart, and test a
  channel's upstream — debugging without digging through container logs.
- 🛡️ **Config that can't corrupt.** Written atomically and validated on load.

## Quick start (Docker)

A pre-built multi-arch image (amd64 + arm64) is published to GHCR on every push
to `main`: **`ghcr.io/munerisuk/tunaar:latest`**.

**No config file needed** — everything can be set with `TUNAAR_*` environment
variables. The only one you must set is your playlist:

```bash
docker run -d --name tunaar --restart unless-stopped \
  --network host \
  -e TUNAAR_PLAYLIST="https://iptv-org.github.io/iptv/index.m3u" \
  -v tunaar-config:/config \
  ghcr.io/munerisuk/tunaar:latest
```

Open `http://<host>:5004` for the dashboard. That's it.

> Host networking keeps the advertised stream URLs correct on your LAN. On
> Docker Desktop (Mac/Windows), drop `--network host` and add `-p 5004:5004`.

### QNAP (Container Station)

QNAP NAS are Linux, so the pre-built image and host networking both work. The
one-liner above works over SSH (Container Station ships Docker). Or use the UI:

**Container Station → Containers → Create**, search the registry for
`ghcr.io/munerisuk/tunaar`, choose **Host** networking, add an environment
variable `TUNAAR_PLAYLIST=<your playlist URL>`, and create.

> The GHCR package must be **public** (GitHub → Packages → tunaar → Package
> settings → Change visibility) for the NAS to pull it without credentials.

Then browse to `http://<nas-ip>:5004`.

### Automatic updates

Two ways to keep Tunaar current:

**Hands-off (recommended) — Watchtower.** Runs a tiny companion container that
checks for a new image and updates Tunaar for you. Scope it to *only* the
`tunaar` container so nothing else on your host is touched:

```bash
docker run -d --name watchtower-tunaar --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  containrrr/watchtower --cleanup --interval 86400 tunaar
```

Lower `--interval` (seconds) to check more often; stop it with
`docker rm -f watchtower-tunaar`.

**One-click from the dashboard.** The **Console → Update** button lets Tunaar
pull a new image and recreate itself. Enable it by adding the Docker socket and
flag to your `docker run` / Compose:

```bash
-e TUNAAR_SELF_UPDATE=1 \
-v /var/run/docker.sock:/var/run/docker.sock
```

Both grant the container/companion Docker access (root-equivalent), so opt in
deliberately. If you'd rather not, pull the new image and recreate the container
over SSH.

### Using a compose file instead

Create a `docker-compose.yml` next to wherever you keep your stacks:

```yaml
services:
  tunaar:
    image: ghcr.io/munerisuk/tunaar:latest
    container_name: tunaar
    restart: unless-stopped
    network_mode: host
    environment:
      - TUNAAR_PLAYLIST=https://your/playlist.m3u
    volumes:
      - tunaar-config:/config
volumes:
  tunaar-config:
```

then `docker compose up -d`.

### Unraid

**Docker tab → Add Container** and fill in:

| Field | Value |
|-------|-------|
| Name | `Tunaar` |
| Repository | `ghcr.io/munerisuk/tunaar:latest` |
| Network Type | **Host** *(so Plex auto-discovers the tuner via UDP 65001)* |
| Path | Container `/config` → Host `/mnt/user/appdata/tunaar` |
| WebUI | `http://[IP]:5004/` |

Apply, then open `http://<unraid-ip>:5004` and run the setup wizard. Config lives in
`appdata`, so it survives updates. A ready-made template is in
[`unraid/tunaar.xml`](unraid/tunaar.xml) — drop it in
`/boot/config/plugins/dockerMan/templates-user/` to get it under *Add Container →
Template*.

**Updates:** Unraid handles them natively — the Docker tab shows *update available*;
or install **CA Auto Update Applications** for hands-off updates (no Watchtower needed).
If you must use bridge networking instead of host, add a port map `5004:5004` and add
the tuner in Plex manually by IP.

## Add the tuner in Plex

1. **Settings → Live TV & DVR → Set up Plex DVR.**
2. Tunaar answers HDHomeRun discovery (UDP 65001), so it should appear
   automatically. If it doesn't (e.g. Plex is on another subnet), click
   **"Enter its network address manually"** and enter `192.168.1.50:5004`.
3. Map channels. For the guide, choose **"Have an XMLTV file?"** and point Plex at
   `http://192.168.1.50:5004/epg.xml` (or use Plex's own guide and let it match).

> Auto-discovery needs the container on **host networking** (the default) so the
> broadcast reaches it. Disable it with `TUNAAR_DISCOVERY=false` if needed.

Emby/Jellyfin: add an **M3U Tuner** at `…/lineup.json` (or an HDHomeRun device)
and an **XMLTV** guide at `…/epg.xml`.

## Configuration

`config.json` (see `config.example.json`):

| Key | Description |
|-----|-------------|
| `friendly_name` | Tuner name shown in Plex. |
| `device_id` | Stable device id. Leave `""` to auto-generate & persist. |
| `tuner_count` | Max simultaneous streams (tuner slots). |
| `host` / `port` | Bind address (default `0.0.0.0:5004`). |
| `playlist` | **Required.** URL or path to your M3U playlist. |
| `epg_url` | URL or path to an XMLTV guide (`.xml` or `.xml.gz`). Optional. |
| `stream_mode` | `ffmpeg` (default, robust), `direct` (passthrough), or `redirect`. |
| `filter_epg_to_lineup` | Trim the guide to channels in your lineup (default `true`). |
| `user_agent` | User-Agent sent to playlist/EPG/stream sources. |
| `buffer_chunk` | Stream read size in bytes. |
| `playlist_refresh` / `epg_refresh` | Cache TTLs in seconds. |
| `advertised_url` | Override the base URL given to Plex (reverse-proxy setups). |

Every key can also be set via an environment variable named `TUNAAR_<KEY>` in
uppercase (e.g. `TUNAAR_PLAYLIST`, `TUNAAR_TUNER_COUNT`, `TUNAAR_EPG_URL`). Env
vars override the config file, so no file is required at all. The config file
path itself is set with `TUNAAR_CONFIG` (defaults to `config.json`, or
`/config/config.json` in Docker).

## Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /` | Branded status dashboard. |
| `GET /console` | Admin console: live logs, controls, channel testing. |
| `GET /discover.json` · `/lineup.json` · `/lineup_status.json` · `/device.xml` | HDHomeRun emulation for Plex. |
| `POST /lineup.post` | Channel-scan trigger (no-op). |
| `GET /stream/<n>` | Remuxed/proxied stream for channel `n` (consumes a tuner slot). |
| `GET /epg.xml` | XMLTV guide, filtered to your lineup. |
| `GET /api/status` · `/api/channels` | JSON for the dashboard. |
| `GET /healthz` | Health check. |

## Roadmap

- Editable channel ordering / filtering from the dashboard.
- SSDP broadcast for fully zero-config discovery.
- Per-source playlist merging and channel grouping.

## License

Tunaar is **proprietary software** — © 2026 Muneris Management Ltd, all rights
reserved. It is **not** open-source or free software. Your use is governed by the
[Tunaar End User License Agreement](LICENSE) (see also [NOTICE](NOTICE)). You may
not copy, redistribute, modify, sublicense, or reverse engineer it except as that
agreement (or applicable law) expressly permits; the availability of source code
does not grant any such rights.

**Get a license.** Tunaar offers a free trial, then **£16.99/year** or a one-off
**£49.99 lifetime** license, sold through our store at
<https://muneris.lemonsqueezy.com/>. For volume, OEM, or other licensing
enquiries, contact <info@muneris.co.uk>.

Contributions are accepted only under the project's [Contributor License
Agreement](CONTRIBUTING.md), which assigns the necessary rights to Muneris.
