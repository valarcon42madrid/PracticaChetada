<header>

# Practica final de DevOps

</header>

# CI/CD con GitHub Actions: Build, Lint, Scan y Push de imagen Docker

Este proyecto incluye una configuración automatizada de CI/CD utilizando **GitHub Actions** que se activa en los siguientes casos:

- ✅ Cuando se hace `push` a la rama `main`
- ✅ Cuando se abre un `pull_request`

---

## 🔧 ¿Qué hace exactamente el workflow?

Cada vez que se ejecuta, GitHub Actions realiza los siguientes pasos:

### 1. ✅ Checkout del repositorio
Descarga el contenido del repositorio en el runner.

### 2. 🧹 Validación de HTML
Usa [`tidy`](https://www.html-tidy.org/) para verificar que `index.html` esté correctamente formado y sin errores.

### 3. 🏷️ Generación de metadatos
Utiliza [`docker/metadata-action`](https://github.com/docker/metadata-action) para generar automáticamente tags como:
- `latest`
- `main` (nombre de la rama)
- `sha-xxxxxx` (hash del commit)

### 4. 🔐 Login a Docker Hub
Autentica contra Docker Hub usando credenciales almacenadas como secretos y variables del repositorio:
- `DOCKER_USERNAME` (variable)
- `DOCKER_TOKEN` (secreto)

### 5. 🐳 Build y Push de la imagen Docker
Construye la imagen con `docker/build-push-action` y la sube a Docker Hub bajo el nombre:

docker.io/<DOCKER_USERNAME>/imagev2

yaml
Copiar
Editar

Con los tags generados automáticamente.

### 6. 🔒 Escaneo de seguridad con Trivy
La imagen recién construida es escaneada por [`Trivy`](https://github.com/aquasecurity/trivy), detectando vulnerabilidades conocidas en paquetes del sistema.

#### El workflow realiza 2 escaneos:
- **Normal**, para generar un reporte de vulnerabilidades.
- **Estricto**, que falla si se detecta al menos una vulnerabilidad crítica (`CRITICAL`).

### 7. 📄 Subida de reportes
Se generan y suben automáticamente los siguientes reportes:
- ✅ **SARIF report** (compatible con GitHub Security)
- ✅ **Artifact descargable** (`trivy-results.sarif`)

### 8. 📣 Notificación por Slack
Si alguna parte del workflow falla, se envía automáticamente un mensaje a un canal de Slack utilizando un webhook definido en el secreto `SLACK_WEBHOOK_URL`.

## 🤝 Créditos

Workflow basado en acciones oficiales de:
- Docker
- Trivy (Aqua Security)
- GitHub Actions
