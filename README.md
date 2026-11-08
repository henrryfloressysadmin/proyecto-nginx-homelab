# 🚀 Homelab: Stack de Infraestructura & SysAdmin Seguro con Docker Compose

Este repositorio contiene la arquitectura de un laboratorio de infraestructura (*Homelab*) diseñado bajo estándares profesionales de producción, desplegado sobre **Ubuntu Server** (VirtualBox en modo *Bridge*) y administrado remotamente vía **SSH** desde un entorno anfitrión con Windows 11.

## 🛠️ Stack Tecnológico

* **Servidor Host:** VirtualBox (Adaptador Puente) / Ubuntu Server
* **Gestión de Contenedores:** Docker & Docker Compose v2+
* **Enrutamiento e Ingress:** Nginx Proxy Manager (NPM)
* **Gestión Gráfica:** Portainer CE

---

## 🔒 Arquitectura de Red y Seguridad (Zero-Trust)

A diferencia de configuraciones básicas de laboratorio donde se exponen todos los puertos al Host, este entorno implementa principios de **seguridad defensiva y aislamiento por capas**:

1. **Punto de Entrada Único:** Solo los puertos de **Nginx Proxy Manager (`80` y `443`)** y el panel de gestión del proxy (`81`) están expuestos hacia la red externa.
2. **Puertos del Host Cerrados:** Los servicios de **Portainer (puerto interno `9000`)** y la **Landing Page de Nginx (puerto interno `80`)** no exponen puertos hacia la IP del servidor Ubuntu. Están blindados contra escaneos de puertos externos.
3. **Pasillos Internos (Redes Docker):** Todo el tráfico se resuelve de manera interna a través de una red compartida aislada (`red-publica-shared`). Nginx Proxy Manager actúa como el único intermediario (Proxy Inverso), enrutando las peticiones mediante el DNS interno de Docker utilizando los nombres de los servicios (`http://portainer:9000` y `http://nginx-web:80`).
4. **Seguridad del Socket:** El socket de Docker (`/var/run/docker.sock`) se monta en Portainer bajo el esquema de **Solo Lectura (`:ro`)** para mitigar riesgos de escalada de privilegios si el contenedor se ve comprometido.

---

## ⚙️ Estructura del Proyecto

```text
proyecto-nginx-homelab/
├── .gitignore            # Filtro de seguridad para datos persistentes
├── README.md             # Documentación del proyecto
├── docker-compose.yml    # Declaración de la infraestructura como código (IaC)
├── index.html            # Landing page estática de producción
└── prometheus/
    └── prometheus.yml    # Configuración de recolección de métricas
```

---

## 🚀 Guía de Despliegue Rápido

### 1. Simulación de Dominios Locales (Host Anfitrión)
Para resolver los dominios de pruebas corporativos de este laboratorio, añade las siguientes líneas al archivo `hosts` de tu sistema anfitrión (`C:\Windows\System32\drivers\etc\hosts` en Windows o `/etc/hosts` en macOS/Linux):

```text
<IP_DE_TU_VM_UBUNTU>   portainer.local
<IP_DE_TU_VM_UBUNTU>   miweb.local
<IP_DE_TU_VM_UBUNTU>   grafana.local
```

### 2. Inicialización del Entorno
Clona este repositorio en tu servidor Ubuntu, crea la red puente compartida externa y levanta el stack en segundo plano:

```bash
# Crear la red externa requerida por la infraestructura
docker network create red-publica-shared

# Levantar todos los servicios en modo desatendido
docker compose up -d
```

---

## 📊 Próxima Fase: Capa de Observabilidad
El stack cuenta con las bases listas para acoplar un entorno de monitoreo avanzado utilizando **Prometheus, Grafana y Google cAdvisor**, permitiendo recolectar y visualizar métricas de consumo de CPU, memoria y red de cada contenedor en tiempo real a través de `http://grafana.local`.
