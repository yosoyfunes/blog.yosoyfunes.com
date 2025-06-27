---
title: "Configuración de Perfiles"
weight: 2
bookToc: true
---

# Configurar perfil AWS para LocalStack

Configurar un perfil personalizado de AWS para LocalStack te permite alternar fácilmente entre tu entorno local y AWS real sin cambiar configuraciones constantemente.

## 🔧 Configuración del perfil LocalStack

### 1. Configurar el archivo de configuración

Agrega el siguiente perfil a tu archivo de configuración de AWS (`~/.aws/config`):

```ini
[profile localstack]
region=us-east-1
output=json
endpoint_url = http://localhost:4566
```

### 2. Configurar las credenciales

Agrega el siguiente perfil a tu archivo de credenciales de AWS (`~/.aws/credentials`):

```ini
[localstack]
aws_access_key_id=test
aws_secret_access_key=test
```

## 🚀 Usar el perfil LocalStack

### Comando específico con perfil

```bash
aws s3 ls --profile localstack
aws dynamodb list-tables --profile localstack
aws lambda list-functions --profile localstack
```

### Establecer como perfil por defecto

```bash
export AWS_DEFAULT_PROFILE=localstack
```

Ahora todos los comandos AWS usarán LocalStack automáticamente:

```bash
aws s3 ls
aws dynamodb list-tables
```

### Volver al perfil por defecto

```bash
unset AWS_DEFAULT_PROFILE
# o
export AWS_DEFAULT_PROFILE=default
```

## 🔄 Alternar entre perfiles

### Script para cambiar perfiles

Crea un script `switch-profile.sh`:

```bash
#!/bin/bash

case $1 in
  "local")
    export AWS_DEFAULT_PROFILE=localstack
    echo "✅ Cambiado a perfil LocalStack"
    ;;
  "aws")
    export AWS_DEFAULT_PROFILE=default
    echo "✅ Cambiado a perfil AWS"
    ;;
  *)
    echo "Uso: $0 {local|aws}"
    echo "Perfil actual: ${AWS_DEFAULT_PROFILE:-default}"
    ;;
esac
```

### Alias útiles

Agrega estos alias a tu `.bashrc` o `.zshrc`:

```bash
# Alias para LocalStack
alias awslocal="aws --profile localstack"
alias s3local="aws s3 --profile localstack"
alias dynamolocal="aws dynamodb --profile localstack"

# Cambiar perfiles rápidamente
alias use-local="export AWS_DEFAULT_PROFILE=localstack"
alias use-aws="export AWS_DEFAULT_PROFILE=default"
```

## ✅ Verificar la configuración

### Comprobar perfiles disponibles

```bash
aws configure list-profiles
```

### Verificar configuración actual

```bash
aws configure list --profile localstack
```

### Probar conectividad

```bash
# Verificar LocalStack
aws s3 ls --profile localstack

# Verificar AWS real
aws s3 ls --profile default
```

## 🎯 Ventajas del uso de perfiles

- **Seguridad**: Evita usar credenciales reales en desarrollo
- **Flexibilidad**: Cambia fácilmente entre entornos
- **Consistencia**: Mismos comandos para local y AWS
- **Automatización**: Integra en scripts y CI/CD

¡Ahora puedes trabajar eficientemente con LocalStack y AWS usando perfiles!
