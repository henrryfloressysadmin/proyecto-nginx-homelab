# 🚀 Homelab: Nginx + Portainer con Docker Compose

Este repositorio contiene la arquitectura de un laboratorio de infraestructura (*Homelab*) desplegado sobre **Ubuntu Server** (VirtualBox en modo *Bridge*) y administrado remotamente vía **SSH** desde macOS.

## 🛠️ Stack Tecnológico
* **Servidor Host:** VirtualBox (Adaptador Puente) / Ubuntu Server
* **Gestión de Contenedores:** Docker & Docker Compose
* **Servicios Desplegados:**
  * **Nginx** (Puerto `8880`): Servidor web que expone una landing page estática.
  * **Portainer CE** (Puerto `9000`): Interfaz gráfica web para la administración y monitorización del entorno Docker.

## ⚙️ Estructura del Proyecto
```text
proyecto-nginx-homelab/
├── docker-compose.yml
├── index.html
└── README.md# 🚀 Homelab: Despliegue de Nginx con Docker Compose

Este repositorio documenta un entorno de laboratorio local (homelab) implementado sobre una máquina virtual con *Ubuntu Server* gestionada mediante *VirtualBox, utilizando **Docker* y *Docker Compose* para la contenerización de servicios web.

## 🛠️ Stack Tecnológico
* *Virtualización:* VirtualBox (Red configurada en modo Adaptador Puente)
* *Sistema Operativo:* Ubuntu Server (Acceso remoto vía SSH)
* *Contenerización:* Docker & Docker Compose
* *Servidor Web:* Nginx (Mapeo de puertos 8880:80)

## ⚙️ Estructura del Proyecto
```text
proyecto-nginx-homelab/
├── docker-compose.yml
├── index.html
└── README.md
