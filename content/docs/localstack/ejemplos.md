---
title: "Ejemplos"
weight: 5
bookToc: true
---

# Ejemplos prácticos con LocalStack

En este post te muestro diferentes formas de usar LocalStack con ejemplos prácticos y configuraciones útiles para diferentes escenarios de desarrollo.

## 🔧 Configurar AWS CLI para LocalStack

### Crear un alias para AWS CLI local

La forma más simple de usar AWS CLI con LocalStack:

```bash
alias aws="aws --endpoint-url http://localhost:4566"
```

Ahora puedes usar comandos AWS normalmente:

```bash
aws s3 ls
aws dynamodb list-tables
aws lambda list-functions
```

### Usar variables de entorno

```bash
export AWS_ENDPOINT_URL=http://localhost:4566
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
```

## 🐳 Ejemplos con Docker

### 1. Imagen Docker personalizada

Si tienes una imagen personalizada con herramientas preinstaladas:

```bash
docker run --rm -ti \
    --network host \
    -v ~/.ssh:/root/.ssh:ro \
    -v $(pwd):/app \
    -w /app \
    yosoyfunes/cbff-local:v1.0.0
```

### 2. Sandbox completo (sin mapear nada)

Para un entorno completamente aislado:

```bash
docker run --rm -ti \
    --network host \
    -w /app \
    yosoyfunes/cbff-local:v1.0.0
```

### 3. Compartir Docker con network local

Para proyectos que requieren acceso a módulos locales:

```bash
docker run --rm -ti -w /data \
    --network host \
    -v ~/.ssh:/root/.ssh:ro \
    -v /Users/matias/Projects/terraform_modules:/terraform_modules \
    -v $(pwd):/app -w /app \
    yosoyfunes/cbff-local:v1.0.0
```

## 📝 Scripts útiles para automatización

### Script de inicialización

Crea un archivo `init-localstack.sh`:

```bash
#!/bin/bash

echo "🚀 Iniciando LocalStack..."

# Verificar si LocalStack está corriendo
if ! curl -s http://localhost:4566/health > /dev/null; then
    echo "❌ LocalStack no está corriendo. Iniciando..."
    localstack start -d
    
    # Esperar a que LocalStack esté listo
    echo "⏳ Esperando a que LocalStack esté listo..."
    while ! curl -s http://localhost:4566/health > /dev/null; do
        sleep 2
    done
fi

echo "✅ LocalStack está listo!"

# Configurar alias
alias aws="aws --endpoint-url http://localhost:4566"

# Crear recursos iniciales
echo "📦 Creando recursos iniciales..."

# Crear bucket S3
aws s3 mb s3://my-dev-bucket

# Crear tabla DynamoDB
aws dynamodb create-table \
    --table-name users \
    --attribute-definitions AttributeName=id,AttributeType=S \
    --key-schema AttributeName=id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST

echo "🎉 Configuración completada!"
```

### Script de limpieza

Crea un archivo `cleanup-localstack.sh`:

```bash
#!/bin/bash

echo "🧹 Limpiando recursos de LocalStack..."

# Configurar alias
alias aws="aws --endpoint-url http://localhost:4566"

# Eliminar buckets S3
echo "🗑️ Eliminando buckets S3..."
for bucket in $(aws s3 ls | awk '{print $3}'); do
    aws s3 rb s3://$bucket --force
done

# Eliminar tablas DynamoDB
echo "🗑️ Eliminando tablas DynamoDB..."
for table in $(aws dynamodb list-tables --query 'TableNames[]' --output text); do
    aws dynamodb delete-table --table-name $table
done

# Eliminar funciones Lambda
echo "🗑️ Eliminando funciones Lambda..."
for function in $(aws lambda list-functions --query 'Functions[].FunctionName' --output text); do
    aws lambda delete-function --function-name $function
done

echo "✅ Limpieza completada!"
```

## 🔄 Ejemplo de workflow completo

### 1. Docker Compose para desarrollo

```yaml
version: '3.8'

services:
  localstack:
    image: localstack/localstack
    ports:
      - "4566:4566"
    environment:
      - SERVICES=s3,dynamodb,lambda,iam,cloudformation
      - DEBUG=1
      - DATA_DIR=/tmp/localstack/data
      - DOCKER_HOST=unix:///var/run/docker.sock
    volumes:
      - "/tmp/localstack:/tmp/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
    
  app:
    build: .
    depends_on:
      - localstack
    environment:
      - AWS_ENDPOINT_URL=http://localstack:4566
      - AWS_ACCESS_KEY_ID=test
      - AWS_SECRET_ACCESS_KEY=test
      - AWS_DEFAULT_REGION=us-east-1
    volumes:
      - .:/app
    working_dir: /app
```

### 2. Makefile para automatización

```makefile
.PHONY: start stop test deploy clean

start:
	@echo "🚀 Iniciando LocalStack..."
	docker-compose up -d localstack
	@echo "⏳ Esperando a LocalStack..."
	@while ! curl -s http://localhost:4566/health > /dev/null; do sleep 2; done
	@echo "✅ LocalStack listo!"

stop:
	@echo "🛑 Deteniendo LocalStack..."
	docker-compose down

test:
	@echo "🧪 Ejecutando tests..."
	pytest tests/ -v

deploy:
	@echo "🚀 Desplegando infraestructura..."
	terraform init
	terraform plan -out=tfplan
	terraform apply tfplan

clean:
	@echo "🧹 Limpiando recursos..."
	./cleanup-localstack.sh

setup: start
	@echo "📦 Configurando recursos iniciales..."
	./init-localstack.sh
```

## 🧪 Testing con LocalStack

### Ejemplo de test con pytest

```python
import boto3
import pytest
from moto import mock_s3

@pytest.fixture
def s3_client():
    return boto3.client(
        's3',
        endpoint_url='http://localhost:4566',
        aws_access_key_id='test',
        aws_secret_access_key='test',
        region_name='us-east-1'
    )

def test_create_bucket(s3_client):
    bucket_name = 'test-bucket'
    
    # Crear bucket
    s3_client.create_bucket(Bucket=bucket_name)
    
    # Verificar que existe
    response = s3_client.list_buckets()
    bucket_names = [bucket['Name'] for bucket in response['Buckets']]
    
    assert bucket_name in bucket_names

def test_upload_file(s3_client):
    bucket_name = 'test-bucket'
    key = 'test-file.txt'
    content = 'Hello LocalStack!'
    
    # Crear bucket
    s3_client.create_bucket(Bucket=bucket_name)
    
    # Subir archivo
    s3_client.put_object(
        Bucket=bucket_name,
        Key=key,
        Body=content
    )
    
    # Verificar contenido
    response = s3_client.get_object(Bucket=bucket_name, Key=key)
    assert response['Body'].read().decode() == content
```

## 🎯 Consejos y mejores prácticas

### 1. Variables de entorno para configuración

```bash
# .env file
LOCALSTACK_ENDPOINT=http://localhost:4566
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test
AWS_DEFAULT_REGION=us-east-1
DEBUG=1
```

### 2. Healthcheck personalizado

```bash
#!/bin/bash
check_localstack() {
    local max_attempts=30
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        if curl -s http://localhost:4566/health | grep -q "running"; then
            echo "✅ LocalStack está listo!"
            return 0
        fi
        
        echo "⏳ Intento $attempt/$max_attempts - Esperando LocalStack..."
        sleep 2
        ((attempt++))
    done
    
    echo "❌ LocalStack no respondió después de $max_attempts intentos"
    return 1
}
```

### 3. Configuración para CI/CD

```yaml
# GitHub Actions example
name: Test with LocalStack

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      localstack:
        image: localstack/localstack
        ports:
          - 4566:4566
        env:
          SERVICES: s3,dynamodb,lambda
          DEBUG: 1
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Wait for LocalStack
        run: |
          while ! curl -s http://localhost:4566/health; do
            sleep 2
          done
      
      - name: Run tests
        run: |
          export AWS_ENDPOINT_URL=http://localhost:4566
          pytest tests/
```

¡Con estos ejemplos puedes aprovechar al máximo LocalStack en tus proyectos de desarrollo!
