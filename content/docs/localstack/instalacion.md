---
title: "Instalación"
weight: 1
bookToc: true
---

# Cómo instalar LocalStack

LocalStack es un proyecto OpenSource que nos permite tener localmente los mismos servicios que podemos utilizar en la nube de AWS. Es perfecto para desarrollo y testing sin incurrir en costos de AWS.

## 📋 Requisitos previos

- Docker instalado en tu sistema
- Python 3.7+ (para instalación local)
- AWS CLI configurado

## 🚀 Métodos de instalación

### Método 1: Instalación local con pip

```bash
pip install localstack
```

### Método 2: Usando Docker Compose (Recomendado)

Crea un archivo `docker-compose.yaml`:

```yaml
version: "3.8"

services:
  localstack:
    container_name: "${LOCALSTACK_DOCKER_NAME-localstack_main}"
    image: localstack/localstack
    network_mode: bridge
    ports:
      - "127.0.0.1:53:53"                # only required for Pro (DNS)
      - "127.0.0.1:53:53/udp"            # only required for Pro (DNS)
      - "127.0.0.1:443:443"              # only required for Pro (LocalStack HTTPS Edge Proxy)
      - "127.0.0.1:4510-4559:4510-4559"  # external service port range
      - "127.0.0.1:4566:4566"            # LocalStack Edge Proxy
    environment:
      - DEBUG=${DEBUG-}
      - DATA_DIR=${DATA_DIR-}
      - LAMBDA_EXECUTOR=${LAMBDA_EXECUTOR-}
      - HOST_TMP_FOLDER=${TMPDIR:-/tmp/}localstack
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "${TMPDIR:-/tmp}/localstack:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
```

## ▶️ Iniciar LocalStack

### Por consola (instalación local)

```bash
localstack start -d
```

### Con Docker Compose

```bash
docker-compose up -d
```

### Con Docker run

```bash
docker run --rm -it -p 4566:4566 -p 4510-4559:4510-4559 localstack/localstack
```

## ✅ Verificar la instalación

Una vez que LocalStack esté ejecutándose, puedes verificar que funciona correctamente:

```bash
# Verificar el estado de LocalStack
curl http://localhost:4566/health

# Listar servicios disponibles
aws --endpoint-url=http://localhost:4566 s3 ls
```

## 🔗 Enlaces útiles

- [Documentación oficial de LocalStack](https://docs.localstack.cloud/getting-started/installation/)
- [Repositorio en GitHub](https://github.com/localstack/localstack)

¡Ahora tienes LocalStack funcionando localmente y puedes empezar a desarrollar con servicios AWS sin costo!
