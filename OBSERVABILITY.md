# Stack de Observabilidad (Grafana, Prometheus, Loki, Tempo, OTel)

Este proyecto incluye una pila completa de observabilidad basada en Docker para monitoreo de métricas, logs y trazas.

## 🚀 Requisitos Previos

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/) o Colima/Docker Desktop.

## 🛠️ Cómo Iniciar el Stack

Para levantar todos los servicios en segundo plano, ejecuta el siguiente comando en la raíz del proyecto:

```bash
docker-compose up -d
```

Para verificar que todos los contenedores están corriendo:

```bash
docker-compose ps
```

## 🌐 Accesos y Credenciales

| Servicio | URL Local | Puerto OTLP (gRPC) |
| :--- | :--- | :--- |
| **Grafana** | [http://localhost:3000](http://localhost:3000) (admin/admin) | - |
| **Prometheus** | [http://localhost:9090](http://localhost:9090) | - |
| **Loki** | [http://localhost:3100](http://localhost:3100) | - |
| **Tempo** | [http://localhost:3200](http://localhost:3200) | `4320` (Internal: 4317) |
| **OTel Collector** | `localhost:4317` (gRPC) | `4317` |

## 📂 Estructura de Configuración

Los archivos de configuración se encuentran en la carpeta `./observability/`:

- `prometheus.yml`: Configuración de scrape y remote write.
- `loki-config.yml`: Configuración de almacenamiento y retención de logs.
- `tempo-config.yml`: Configuración de ingesta de trazas (OTLP).
- `otel-config.yaml`: Pipeline del colector (Receivers, Processors, Exporters).
- `datasources.yml`: Configuración automática de data sources para Grafana.

## ⚙️ Configuración en Grafana (Data Sources)

**Nota:** Los Data Sources ya están pre-configurados automáticamente. Al entrar a Grafana, solo tienes que ir a la sección **Explore** y seleccionar **Prometheus**, **Loki** o **Tempo**.

- **Correlación:** Las trazas (Tempo) están conectadas con los logs (Loki) y viceversa mediante `traceID`.

## 💻 Ejecución Local de la Aplicación

Para que la aplicación envíe datos al stack local, asegúrate de ejecutarla con:

```bash
make web
```

O si usas **GoLand**, utiliza la configuración de ejecución **"Run Local"** que ya tiene las variables de entorno configuradas:
- `ENV=dev`
- `OTEL_EXPORTER_OTLP_ENDPOINT=localhost:4317`
- `OTEL_EXPORTER_OTLP_PROTOCOL=grpc`
- `OTEL_EXPORTER_OTLP_INSECURE=true`

## 🛑 Cómo Detener el Stack

Para detener y eliminar los contenedores:

```bash
docker-compose down
```

Para re-crear todo ante un cambio de configuración:

```bash
docker-compose down && docker-compose up -d
```
