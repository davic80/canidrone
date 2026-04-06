# Can I Drone?

Check instantly if you can fly your drone in Spain. Real-time airspace restrictions from [ENAIRE](https://drones.enaire.es), weather conditions from [Open-Meteo](https://open-meteo.com), and daylight status -- all in one screen.

**Live:** [canidrone.f1madrid.win](https://canidrone.f1madrid.win)

## What it does

1. **Geolocates you** (requires HTTPS + browser permission)
2. **Queries ENAIRE ArcGIS services** -- controlled airspace (CTR/FIZ), restricted zones, airport surroundings, infrastructure zones, active NOTAMs, and warnings
3. **Fetches live weather** -- wind, gusts, precipitation, visibility, cloud cover, temperature, humidity
4. **Checks daylight** -- sunrise/sunset times, minutes until dark
5. **Gives a clear verdict**: green (fly), yellow (caution), red (don't fly)
6. **Shows a map** with all nearby restriction zones rendered as colored polygons

## Screenshots

| Verdict | Map |
|---------|-----|
| Clear YES/NO/CAUTION banner | Leaflet dark map with ENAIRE zones |

## Tech stack

- **Frontend**: Vanilla HTML/CSS/JS (zero dependencies, zero build step)
- **Map**: [Leaflet](https://leafletjs.com) + CARTO dark tiles
- **Airspace data**: ENAIRE ArcGIS REST API (public, no key needed)
- **Weather**: [Open-Meteo API](https://open-meteo.com) (free, no key needed)
- **Container**: nginx:alpine
- **Deployment**: Raspberry Pi + Cloudflare Tunnel

## Deploy on Raspberry Pi

### Prerequisites

- Docker & Docker Compose installed
- A [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) token pointing to `http://canidrone-app:80`

### Quick start

```bash
git clone https://github.com/davic80/canidrone.git
cd canidrone

# Create .env with your tunnel token
echo "CLOUDFLARE_TUNNEL_TOKEN=your-token-here" > .env

# Run
docker compose up -d
```

The app will be available at:
- Local: `http://<pi-ip>:8090`
- Public: `https://canidrone.f1madrid.win` (via Cloudflare Tunnel)

> **Note**: Geolocation requires HTTPS. It will only work through the Cloudflare Tunnel URL, not via plain HTTP.

### Update to latest version

```bash
docker compose pull && docker compose up -d
```

### Pin a specific version

```bash
IMAGE_TAG=1.1.0 docker compose up -d
```

## ENAIRE data layers

| Layer | Service | Severity |
|-------|---------|----------|
| Controlled airspace / FIZ | `Drones_ZG_Aero_V3/MapServer/1` | Red |
| Warnings (Avisos) | `Drones_ZG_Aero_V3/MapServer/2` | Orange |
| Restricted zones | `Drones_ZG_Aero_V3/MapServer/10` | Red |
| Airport surroundings | `Drones_ZG_Aero_V3/MapServer/6` | Red |
| Critical infrastructure | `Drones_ZG_Infra_V0/MapServer/11` | Yellow |
| NOTAMs | `NOTAM_UAS_APP_V3/MapServer/1` | Orange |

## Weather thresholds

| Parameter | OK | Caution | No fly |
|-----------|----|---------|--------|
| Wind | < 25 km/h | 25-40 km/h | > 40 km/h |
| Gusts | < 35 km/h | 35-50 km/h | > 50 km/h |
| Precipitation | 0 mm | 0-2 mm | > 2 mm |
| Visibility | > 3 km | 1-3 km | < 1 km |

## CI/CD

Pushing a tag triggers a GitHub Actions workflow that:
1. Builds a multi-arch image (`linux/amd64` + `linux/arm64`)
2. Pushes to `ghcr.io/davic80/canidrone`
3. Creates a GitHub Release

```bash
git tag v1.1.0
git push origin v1.1.0
```

## Disclaimer

This app is **informational only** and does not replace the official [ENAIRE Drones](https://drones.enaire.es) tool or [AESA](https://www.seguridadaerea.gob.es/es/ambitos/drones) regulations. Always verify airspace restrictions through official channels before flying.

## License

MIT
