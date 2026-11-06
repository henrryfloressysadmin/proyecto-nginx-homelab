# 🚀 Homelab: Stack de Infraestructura & SysAdmin con Docker Compose

Este repositorio contiene la arquitectura de un laboratorio de infraestructura (*Homelab*) desplegado sobre **Ubuntu Server** (VirtualBox en modo *Bridge*) y administrado remotamente vía **SSH** desde macOS.

## 🛠️ Stack Tecnológico

* **Servidor Host:** VirtualBox (Adaptador Puente) / Ubuntu Server
* **Gestión de Contenedores:** Docker & Docker Compose
* **Servicios Desplegados:**
  * **Nginx** (Puerto `8880`): Servidor web que expone una landing page estática.
  * **Portainer CE** (Puerto `9000`): Interfaz gráfica web para la administración y monitorización del entorno Docker.
  * **Nginx Proxy Manager** (Puertos `80`, `81`, `443`): Proxy Inverso centralizado para enrutamiento de tráfico y gestión de certificados SSL/TLS.

## ⚙️ Estructura del Proyecto

```text
proyecto-nginx-homelab/
├── .gitignore
├── README.md
├── docker-compose.yml
└── index.html
