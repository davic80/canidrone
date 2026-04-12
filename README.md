# Can I Drone?

Comprueba al instante si puedes volar tu dron en España. Restricciones de espacio aéreo de [ENAIRE](https://drones.enaire.es) en tiempo real, meteorología de [Open-Meteo](https://open-meteo.com) y estado de luz solar — todo en una sola pantalla.

**Live:** [canidrone.f1madrid.win](https://canidrone.f1madrid.win)

## Qué hace

1. **Tres modos de ubicación:**
   - **Mi ubicación** — geolocalización GPS del navegador (requiere HTTPS + permiso del browser)
   - **Elegir en el mapa** — arrastra el mapa y pulsa "Comprobar aquí" para analizar cualquier punto
   - **Introducir dirección** — escribe una dirección española; geocodificación via [Nominatim](https://nominatim.openstreetmap.org/)
2. **Consulta ENAIRE** — 6 capas ArcGIS REST: espacio aéreo controlado (CTR/FIZ), zonas restringidas, entorno aeroportuario, infraestructuras críticas, avisos y NOTAMs activos
3. **Meteorología en tiempo real** — viento, rachas, precipitación, visibilidad, nubosidad, temperatura, humedad
4. **Luz solar** — amanecer/atardecer, minutos hasta el anochecer, barra de progreso del día
5. **Veredicto claro** — banner verde (puedes volar), amarillo (precaución), rojo (no vueles)
6. **Mapa dark** con zonas ENAIRE pintadas como polígonos de colores (Leaflet + CARTO)

## Stack

- **Frontend**: HTML/CSS/JS vanilla — sin build tools, single `index.html`
- **Mapa**: [Leaflet.js](https://leafletjs.com) + CARTO dark tiles
- **Espacio aéreo**: ENAIRE ArcGIS REST API (público, sin API key)
- **Meteorología**: [Open-Meteo](https://open-meteo.com) (gratuito, sin API key)
- **Geocodificación**: [Nominatim](https://nominatim.openstreetmap.org/) (OpenStreetMap)
- **Contenedor**: nginx:alpine, multi-arch (`linux/amd64` + `linux/arm64`)
- **Despliegue**: Raspberry Pi + Cloudflare Tunnel

## Despliegue en Raspberry Pi

### Requisitos

- Docker & Docker Compose instalados
- Token de [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) apuntando a `http://canidrone-app:80`

### Inicio rápido

```bash
git clone https://github.com/davic80/canidrone.git
cd canidrone

# Crear .env con tu token del tunnel
echo "CLOUDFLARE_TUNNEL_TOKEN=tu-token-aqui" > .env

# Arrancar
docker compose up -d
```

La app estará disponible en:
- Local: `http://<ip-raspberry>:8090`
- Pública: `https://canidrone.f1madrid.win` (vía Cloudflare Tunnel)

> **Nota**: La geolocalización GPS requiere HTTPS. Solo funciona a través de la URL de Cloudflare Tunnel, no vía HTTP plano.

### Actualizar a la última versión

```bash
docker compose pull && docker compose up -d
```

### Usar una versión específica

```bash
IMAGE_TAG=1.2.0 docker compose up -d
```

## CI/CD

Cualquier push de tag dispara el workflow de GitHub Actions, que:
1. Construye imagen multi-arch (`linux/amd64` + `linux/arm64`)
2. Publica en `ghcr.io/davic80/canidrone`
3. Crea un GitHub Release con instrucciones de despliegue

```bash
git tag v1.2.0
git push origin v1.2.0
```

## Google AdSense

La app incluye dos slots publicitarios responsivos (horizontales):

- **Slot superior** — entre el panel de modo/dirección y el mapa
- **Slot inferior** — entre la tarjeta de luz solar y el botón Actualizar

Para activar los anuncios, edita `index.html` y reemplaza los tres placeholders:

| Placeholder | Dónde | Por qué |
|-------------|-------|---------|
| `ca-pub-XXXXXXXXXXXXXXXX` | `<head>` (script) + 2 `<ins>` | Tu Publisher ID de AdSense |
| `XXXXXXXXXX` (slot top) | `<ins>` superior | ID del slot de anuncio 1 |
| `YYYYYYYYYY` (slot bottom) | `<ins>` inferior | ID del slot de anuncio 2 |

Ver guía completa de configuración abajo.

## Capas ENAIRE

| Capa | Servicio | Severidad |
|------|----------|-----------|
| Espacio aéreo controlado / FIZ | `Drones_ZG_Aero_V3/MapServer/1` | Rojo |
| Avisos | `Drones_ZG_Aero_V3/MapServer/2` | Naranja |
| Zonas restringidas | `Drones_ZG_Aero_V3/MapServer/10` | Rojo |
| Entorno aeroportuario | `Drones_ZG_Aero_V3/MapServer/6` | Rojo |
| Infraestructuras críticas | `Drones_ZG_Infra_V0/MapServer/11` | Amarillo |
| NOTAMs | `NOTAM_UAS_APP_V3/MapServer/1` | Naranja |

## Umbrales meteorológicos

| Parámetro | OK | Precaución | No volar |
|-----------|----|------------|----------|
| Viento | < 25 km/h | 25–40 km/h | > 40 km/h |
| Rachas | < 35 km/h | 35–50 km/h | > 50 km/h |
| Precipitación | 0 mm | 0–2 mm | > 2 mm |
| Visibilidad | > 3 km | 1–3 km | < 1 km |

## Aviso legal

Esta app es **orientativa** y no sustituye la consulta oficial de [ENAIRE Drones](https://drones.enaire.es) ni la normativa de [AESA](https://www.seguridadaerea.gob.es/es/ambitos/drones). Verifica siempre las restricciones por los canales oficiales antes de volar.

## Licencia

MIT
