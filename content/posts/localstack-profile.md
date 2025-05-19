---
title: "Perfil LocalStack"
date: 2024-06-10T00:00:00+00:00
draft: false
---

# Configuración de un perfil personalizado

Puedes configurar un perfil personalizado para usar con LocalStack. Agrega el siguiente perfil a tu archivo de configuración de AWS (por defecto, este archivo está en ~/.aws/config):

```
[profile localstack]
region=us-east-1
output=json
endpoint_url = http://localhost:4566
```

Agrega el siguiente perfil a tu archivo de credenciales de AWS (por defecto, este archivo está en ~/.aws/credentials):

```
[localstack]
aws_access_key_id=test
aws_secret_access_key=test
```

Ahora puedes usar el perfil localstack con la CLI de AWS:

```
aws s3 ls --profile localstack
```

Establecer el perfil por defecto

```
export AWS_DEFAULT_PROFILE=localstack
```