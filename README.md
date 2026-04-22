# Daniel Homelab

Este repositorio contiene la configuración base de mi home server, organizada como un backup completo para facilitar el despliegue y la recuperación de servicios.

## Descripción General

Este proyecto configura un home server utilizando Docker Compose, con servicios para descarga de archivos, infraestructura, gestión de archivos, servidor de medios y monitorización. Está diseñado para ser desplegado en un servidor Linux, específicamente en el directorio `/home/daniel/daniel-homelab`.

## Estructura del Proyecto

- **`stacks/`**: Composes de Docker separados por categoría para despliegues modulares (gestionados con Arcane).
- **`config/`**: Directorio con configuraciones específicas de cada servicio.
  - **`downloader/`**: Configuraciones para herramientas de descarga.
    - `aria2/`: Configuración de Aria2 (descargador de archivos).
    - `qbittorrent/`: Configuración de qBittorrent (cliente torrent).
    - `unpackerr/`: Configuración de Unpackerr (desempaquetador automático).
  - **`infra/`**: Servicios de infraestructura.
    - `arcane/`: Configuración de Arcane.
    - `dashy/`: Configuración de Dashy (dashboard personalizable).
    - `filebrowser/`: Configuración de Filebrowser para infraestructura.
  - **`management/`**: Servicios de gestión.
    - `filebrowser/`: Configuración de Filebrowser para gestión de archivos.
  - **`media/`**: Configuraciones para servidor de medios.
      - `config/`, `log/`, `metadata/`: Archivos de configuración, logs y metadata de Jellyfin (servidor de medios).
      - `jellyfin/`: Configuración específica de Jellyfin.
      - `plugins/`: Plugins adicionales.
      - `root/`: Raíz de medios con carpetas para Películas y Series.
  - `downloader/docker-compose.yml`
  - `infra/docker-compose.yml`
  - `management/docker-compose.yml`
  - `media/docker-compose.yml`
- **`stacks/.env.global`**: Variables de entorno y secretos compartidos entre todos los stacks. ⚠️ Ignorado por Git.
- **`.gitignore`**: Archivos ignorados por Git, incluyendo datos y secretos.

## Servicios Incluidos

- **Descargadores**: Aria2, qBittorrent y Unpackerr para automatizar descargas y desempaquetado.
- **Infraestructura**: Dashy para dashboard y Arcane para gestión de stacks de Docker.
- **Gestión**: Filebrowser para administración de archivos y Uptime Kuma para monitorización de servicios.
- **Medios**: Jellyfin para streaming de películas, series y música.

## Despliegue

1. Clona este repositorio en tu servidor en `/home/daniel/daniel-homelab`.
2. Asegúrate de tener Docker y Docker Compose instalados.
3. Ejecuta `docker-compose up -d` en el directorio raíz para iniciar todos los servicios.
4. Para despliegues modulares, la ejecución de stacks se hace a través de arcane. Navega a `stacks/` y utiliza arcane para gestionar los subdirectorios correspondientes.

## Exposición de Servicios con Tailscale Serve

Tailscale Serve permite exponer servicios con HTTPS usando subdominios de Tailscale, sin necesidad de IP pública ni dominio propio. Primero crea el servicio en la consola de admin de Tailscale (`login.tailscale.com/admin/services`) y luego ejecuta el comando `serve` con `sudo` en el host.

- Para Dashy: `tailscale serve --service=svc:dashy --https=443 http://localhost:8080` (accesible en https://dashy.tailbb818c.ts.net/)
- Para Arcane: `tailscale serve --service=svc:arcane --https=443 http://localhost:3552` (accesible en https://arcane.tailbb818c.ts.net/)
- Para Uptime Kuma: `tailscale serve --service=svc:uptime-kuma --https=443 http://localhost:3001` (accesible en https://uptime-kuma.tailbb818c.ts.net/)

## Notas

- Este es un backup de la configuración base; ajusta variables de entorno en `stacks/.env.global` si es necesario.
- Los archivos de configuración incluyen datos sensibles; maneja con cuidado y no subas a repositorios públicos sin encriptar.
- La metadata de medios incluye información de películas, series y personas, generada por Jellyfin.

## Licencia

Este proyecto es privado y para uso personal.