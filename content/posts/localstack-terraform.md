---
title: "Usar Terraform con LocalStack para pruebas locales"
date: 2024-06-27T01:00:00+00:00
draft: false
tags: ["terraform", "localstack", "aws", "iac"]
categories: ["DevOps", "Infrastructure as Code"]
---

Una de las grandes ventajas de LocalStack es poder probar nuestros scripts de Terraform localmente antes de desplegarlos en AWS. Esto nos permite ahorrar costos y acelerar el desarrollo.

## 🎯 Configuración del provider AWS para LocalStack

Para usar Terraform con LocalStack, necesitas configurar el provider AWS para que apunte a los endpoints locales:

### Archivo `main.tf`

```hcl
provider "aws" {
  region                      = "us-east-1"
  access_key                  = "mock_access_key"
  secret_key                  = "mock_secret_key"
  s3_force_path_style         = true
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true
  
  endpoints {
    s3       = "http://localhost:4566"
    dynamodb = "http://localhost:4566"
    lambda   = "http://localhost:4566"
    iam      = "http://localhost:4566"
    ec2      = "http://localhost:4566"
    # Agrega más servicios según necesites
  }
}

# Ejemplo: Crear un bucket S3
resource "aws_s3_bucket" "example" {
  bucket = "my-tf-test-bucket"
  
  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}

# Ejemplo: Configurar versionado del bucket
resource "aws_s3_bucket_versioning" "example" {
  bucket = aws_s3_bucket.example.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

## 🚀 Ejecutar Terraform

### 1. Inicializar Terraform

```bash
terraform init
```

### 2. Planificar los cambios

```bash
terraform plan -out=tfplan
```

### 3. Aplicar los cambios

```bash
terraform apply "tfplan"
```

## 📦 Ejemplo avanzado: Backend con S3 y DynamoDB

### Archivo `provider.tf`

```hcl
provider "aws" {
  region = "us-east-1"

  # Configuración para LocalStack
  access_key                  = "test"
  secret_key                  = "test"
  s3_force_path_style         = true
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true

  endpoints {
    s3        = "http://s3.localhost.localstack.cloud:4566"
    dynamodb  = "http://localhost:4566"
    kms       = "http://localhost:4566"
  }
}
```

### Archivo `backend.tf`

```hcl
module "backend" {
  source = "/terraform_modules/s3_dynamo_backend"

  ## Variables requeridas
  bucket_name   = "my_terraform_state_bucket_name"
  dynamodb_name = "my_dynamodb_lock_state_table_name"

  ## Variables opcionales
  ## Encriptación KMS o AES, por defecto es AES
  use_kms_encryption = true

  ## block_all_public_access - Por defecto está en false
  block_all_public_access = true
}
```

## 🔧 Configuración de backend remoto

Una vez creado el backend, puedes configurar Terraform para usarlo:

```hcl
terraform {
  backend "s3" {
    bucket         = "my_terraform_state_bucket_name"
    key            = "terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "my_dynamodb_lock_state_table_name"
    encrypt        = true
    
    # Configuración para LocalStack
    endpoint                    = "http://localhost:4566"
    force_path_style           = true
    skip_credentials_validation = true
    skip_metadata_api_check     = true
    skip_requesting_account_id  = true
  }
}
```

## ✅ Verificar recursos creados

Puedes verificar que los recursos se crearon correctamente usando AWS CLI:

```bash
# Listar buckets S3
aws --endpoint-url=http://localhost:4566 s3 ls

# Listar tablas DynamoDB
aws --endpoint-url=http://localhost:4566 dynamodb list-tables

# Describir un bucket específico
aws --endpoint-url=http://localhost:4566 s3api head-bucket --bucket my-tf-test-bucket
```

## 🎉 Ventajas de usar Terraform con LocalStack

- **Desarrollo rápido**: Prueba cambios sin esperar deployments en AWS
- **Sin costos**: No pagas por recursos de prueba
- **Debugging fácil**: Logs locales y acceso directo a los servicios
- **CI/CD**: Integra pruebas automatizadas en tu pipeline

¡Ahora puedes desarrollar y probar tu infraestructura como código de forma local y eficiente!
