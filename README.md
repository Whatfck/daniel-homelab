# Daniel Homelab

Repositorio de configuración de mi home server. Organizado como infraestructura-como-código para facilitar el despliegue, recuperación y replicación de todos los servicios.

## Descripción General

Stack completo de servicios auto-hospedados sobre Docker Compose, gestionado de forma modular por categorías. El acceso remoto se resuelve con **Tailscale** (sin IP pública ni dominio propio). Los secretos nunca se suben al repositorio.

## Arquitectura

```
daniel-homelab/
├── stacks/               # Docker Composes por categoría (gestionados con Arcane)
│   ├── infra/            # Servicios base: dashboard + gestor de stacks
│   ├── media/            # Servidor multimedia
│   ├── downloader/       # Clientes de descarga
│   ├── management/       # Observabilidad + gestión de archivos
│   └── .env.global       # Secretos globales ⚠️ ignorado por Git
├── config/               # Configuración persistente de cada servicio
│   ├── infra/
│   ├── media/
│   ├── downloader/
│   └── management/
│       ├── grafana/      # Incluye provisioning automático de datasources y dashboards
│       └── prometheus/
└── data/                 # Volúmenes de datos (ignorado por Git)
```

## Servicios

### 🏗️ Infraestructura (`stacks/infra`)
| Servicio | Puerto | Descripción |
|---|---|---|
| [Arcane](https://github.com/getarcaneapp/arcane) | `3552` | Gestión web de stacks Docker |
| [Dashy](https://dashy.to) | `8080` | Dashboard principal |

### 🍿 Medios (`stacks/media`)
| Servicio | Puerto | Descripción |
|---|---|---|
| [Jellyfin](https://jellyfin.org) | `8096` | Servidor multimedia (películas, series, música) |

### ⬇️ Descargas (`stacks/downloader`)
| Servicio | Puerto | Descripción |
|---|---|---|
| [qBittorrent](https://www.qbittorrent.org) | `8085` | Cliente torrent |
| [Aria2](https://aria2.github.io) + AriaNG | `6800` / `6880` | Gestor de descargas + UI web |
| [Unpackerr](https://github.com/Unpackerr/unpackerr) | — | Extracción automática de archivos comprimidos |

### 📊 Gestión y Observabilidad (`stacks/management`)
| Servicio | Puerto | Descripción |
|---|---|---|
| [Grafana](https://grafana.com) | `3000` | Dashboards de métricas del sistema y contenedores |
| [Prometheus](https://prometheus.io) | `9090` | Base de datos de series temporales (backend) |
| [cAdvisor](https://github.com/google/cadvisor) | — | Métricas por contenedor Docker (backend) |
| [Node Exporter](https://github.com/prometheus/node_exporter) | — | Métricas del host: CPU, RAM, disco (backend) |
| [Blackbox Exporter](https://github.com/prometheus/blackbox_exporter) | — | Chequeos HTTP de disponibilidad (backend) |
| [Filebrowser](https://filebrowser.org) | `8090` | Explorador de archivos web |

## Despliegue

### Requisitos
- Linux con Docker y Docker Compose instalados
- Tailscale instalado en el host

### Pasos

**1. Clonar el repositorio**
```bash
git clone <repo-url> ~/daniel-homelab
cd ~/daniel-homelab
```

**2. Crear el archivo de secretos**

Este archivo no está en Git. Créalo manualmente:
```bash
cp stacks/.env.global.example stacks/.env.global  # si existe plantilla
# o créalo desde cero:
nano stacks/.env.global
```

Variables requeridas:
```env
PUID=1000
PGID=1000
TZ=America/Bogota

ARCANE_ENCRYPTION_KEY=<genera con: openssl rand -hex 32>
ARCANE_JWT_SECRET=<genera con: openssl rand -hex 32>
GRAFANA_ADMIN_PASSWORD=<tu-contraseña>
```

**3. Levantar los stacks**

El orden importa: `infra` primero (crea la red `homelab-net`).
```bash
# Stack de infraestructura (crea la red compartida)
cd stacks/infra && docker compose up -d

# Resto de stacks
cd ../media      && docker compose up -d
cd ../downloader && docker compose up -d
cd ../management && docker compose up -d
```

A partir de aquí puedes gestionar los stacks desde la UI de **Arcane**.

## Acceso Remoto con Tailscale Serve

Tailscale Serve permite exponer servicios internos con HTTPS automático a través de tu tailnet, sin necesidad de IP pública ni dominio propio.

### Configuración inicial (una sola vez por servicio)

1. Entra a [login.tailscale.com/admin/services](https://login.tailscale.com/admin/services) y crea el servicio con el nombre correspondiente.
2. Ejecuta el comando `serve` en el host con `sudo`:

```bash
# Patrón general
sudo tailscale serve --service=svc:<nombre> --https=443 http://localhost:<puerto>
```

### Servicios configurados

| Servicio | Comando |
|---|---|
| Dashy | `sudo tailscale serve --service=svc:dashy --https=443 http://localhost:8080` |
| Arcane | `sudo tailscale serve --service=svc:arcane --https=443 http://localhost:3552` |
| Grafana | `sudo tailscale serve --service=svc:grafana --https=443 http://localhost:3000` |
| Filebrowser | `sudo tailscale serve --service=svc:file-browser --https=443 http://localhost:8090` |
| Jellyfin | `sudo tailscale serve --service=svc:jellyfin --https=443 http://localhost:8096` |
| qBittorrent | `sudo tailscale serve --service=svc:q-bittorrent --https=443 http://localhost:8085` |

> Una vez configurado, cada servicio es accesible en `https://<nombre>.<tu-tailnet>.ts.net/`

## Observabilidad

Grafana se aprovisiona automáticamente con:
- **Datasource**: Prometheus (configurado vía `config/management/grafana/provisioning/datasources/`)
- **Dashboards** (en la carpeta `Homelab`):
  - *Node Exporter Full* — CPU, RAM, disco y red del host
  - *cAdvisor Docker* — Métricas por contenedor
  - *Blackbox HTTP* — Estado de disponibilidad de todos los servicios

No se requiere configuración manual en la UI de Grafana.

## Notas

- Los archivos `*.db`, `*.log`, `data/` y `stacks/.env.global` están excluidos de Git (ver `.gitignore`).
- Todos los volúmenes usan rutas relativas, lo que hace el proyecto portable a cualquier ruta del servidor.
- Para evitar errores de permisos en Grafana y Prometheus, ambos corren con `user: root` en el compose (práctica aceptable en homelab privado).

## Licencia

Proyecto privado para uso personal.