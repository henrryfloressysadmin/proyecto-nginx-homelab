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

## 🛠️ Bitácora de Diagnóstico y Solución de Problemas (Troubleshooting)

Durante el despliegue y hardening (aseguramiento) de esta infraestructura, surgieron retos técnicos reales de comunicación en redes virtuales. A continuación se documenta el diagnóstico y la solución aplicada para cada uno:

### 1. Error de Sintaxis en Redes de Docker Compose (Línea `comment:`)
*   **Síntoma:** Al ejecutar `docker compose ps` o levantar el entorno, el motor de Docker devolvía el error: `networks.proxy-public additional properties 'comment' not allowed`.
*   **Causa:** Se intentaron incluir comentarios descriptivos usando la propiedad no soportada `comment` dentro de la especificación nativa del esquema de redes de Docker Compose.
*   **Diagnóstico:** Inspección directa del esquema con la terminal arrojando error de validación. La red `mgmt-private` no se creaba en el Daemon de Docker (`network not found`).
*   **Solución:** Se editó el archivo `docker-compose.yml` para remover la propiedad inválida, limpiando la declaración de redes bajo el driver `bridge` estándar y forzando el despliegue limpio.

### 2. Error 502 Bad Gateway en `portainer.local` (Aislamiento Excesivo de Interfaces)
*   **Síntoma:** Nginx Proxy Manager respondía correctamente, pero al intentar acceder a Portainer devolvía un código de estado HTTP 502.
*   **Causa:** Por motivos de seguridad nativos de su imagen, el binario de Portainer se inicializó escuchando únicamente sobre la interfaz de bucle invertido local (`127.0.0.1`), bloqueando los reenvíos de red internos del Proxy Inverso.
*   **Diagnóstico:** Auditoría de logs en tiempo real mediante el comando `docker logs portainer_management`, localizando la línea: `server listening on http://127.0.0.1:9000`.
*   **Solución:** Se inyectó el parámetro `command` en el servicio de Portainer dentro del archivo Compose: `command: -H unix:///var/run/docker.sock --address 0.0.0.0:9000`. Esto forzó al contenedor a escuchar en todas las interfaces virtuales (`0.0.0.0`), abriendo el pasillo de comunicación seguro con NPM.

### 3. Bloqueo de Inicialización por Expiración de Tiempo (Contador de 5 Minutos)
*   **Síntoma:** Al refrescar la pantalla tras un intento fallido de contraseña, Portainer mostraba el mensaje: *“Your Portainer instance timed out for security reasons”*.
*   **Causa:** Portainer implementa un mecanismo de seguridad estricto que cierra la ventana de configuración si el usuario administrador no es creado en los primeros 5 minutos desde su arranque inicial.
*   **Solución:** Se aplicó un reinicio directo del contenedor desde la terminal del host (`docker compose restart portainer`), restableciendo el temporizador de seguridad y permitiendo la creación exitosa del usuario maestro usando una política de contraseña robusta (mínimo 12 caracteres).
