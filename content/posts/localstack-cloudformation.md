---
title: "Usar CloudFormation con LocalStack localmente"
date: 2024-06-27T02:00:00+00:00
draft: false
tags: ["cloudformation", "localstack", "aws", "iac"]
categories: ["DevOps", "Infrastructure as Code"]
---

CloudFormation es el servicio nativo de AWS para Infrastructure as Code. Con LocalStack puedes probar tus templates de CloudFormation localmente antes de desplegarlos en AWS.

## 📝 Ejemplo básico: Crear un bucket S3

### Archivo `s3.yaml`

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Template básico para crear un bucket S3"

Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${AWS::StackName}-bucket"
      Tags:
        - Key: TestKey
          Value: "my bucket 1"
        - Key: Environment
          Value: "Development"
        - Key: CreatedBy
          Value: "LocalStack"

Outputs:
  BucketName:
    Description: "Nombre del bucket S3 creado"
    Value: !Ref MyS3Bucket
    Export:
      Name: !Sub "${AWS::StackName}-BucketName"
  
  BucketArn:
    Description: "ARN del bucket S3"
    Value: !GetAtt MyS3Bucket.Arn
    Export:
      Name: !Sub "${AWS::StackName}-BucketArn"
```

## 🚀 Desplegar el stack

### Crear el stack

```bash
aws cloudformation create-stack \
    --endpoint-url http://localhost:4566 \
    --stack-name samplestack \
    --template-body file://s3.yaml \
    --region us-east-1
```

### Verificar el estado del stack

```bash
aws cloudformation describe-stacks \
    --endpoint-url http://localhost:4566 \
    --stack-name samplestack \
    --region us-east-1
```

### Listar recursos del stack

```bash
aws cloudformation list-stack-resources \
    --endpoint-url http://localhost:4566 \
    --stack-name samplestack \
    --region us-east-1
```

## 📦 Ejemplo avanzado: Stack completo con múltiples recursos

### Archivo `complete-stack.yaml`

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Stack completo con S3, DynamoDB y Lambda"

Parameters:
  Environment:
    Type: String
    Default: "dev"
    AllowedValues: ["dev", "staging", "prod"]
    Description: "Ambiente de despliegue"

Resources:
  # Bucket S3
  DataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${Environment}-data-bucket-${AWS::AccountId}"
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # Tabla DynamoDB
  UserTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: !Sub "${Environment}-users"
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: userId
          AttributeType: S
      KeySchema:
        - AttributeName: userId
          KeyType: HASH
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # Rol IAM para Lambda
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
      Policies:
        - PolicyName: DynamoDBAccess
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - dynamodb:GetItem
                  - dynamodb:PutItem
                  - dynamodb:UpdateItem
                  - dynamodb:DeleteItem
                Resource: !GetAtt UserTable.Arn

  # Función Lambda
  ProcessorFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: !Sub "${Environment}-processor"
      Runtime: python3.9
      Handler: index.handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: |
          import json
          import boto3
          
          def handler(event, context):
              print(f"Received event: {json.dumps(event)}")
              return {
                  'statusCode': 200,
                  'body': json.dumps('Hello from LocalStack!')
              }
      Environment:
        Variables:
          TABLE_NAME: !Ref UserTable
          BUCKET_NAME: !Ref DataBucket

Outputs:
  BucketName:
    Description: "Nombre del bucket S3"
    Value: !Ref DataBucket
    Export:
      Name: !Sub "${AWS::StackName}-BucketName"
  
  TableName:
    Description: "Nombre de la tabla DynamoDB"
    Value: !Ref UserTable
    Export:
      Name: !Sub "${AWS::StackName}-TableName"
  
  LambdaFunctionArn:
    Description: "ARN de la función Lambda"
    Value: !GetAtt ProcessorFunction.Arn
    Export:
      Name: !Sub "${AWS::StackName}-LambdaArn"
```

### Desplegar el stack completo

```bash
aws cloudformation create-stack \
    --endpoint-url http://localhost:4566 \
    --stack-name complete-stack \
    --template-body file://complete-stack.yaml \
    --parameters ParameterKey=Environment,ParameterValue=dev \
    --capabilities CAPABILITY_IAM \
    --region us-east-1
```

## 🔍 Comandos útiles para gestión de stacks

### Actualizar un stack

```bash
aws cloudformation update-stack \
    --endpoint-url http://localhost:4566 \
    --stack-name samplestack \
    --template-body file://s3.yaml \
    --region us-east-1
```

### Eliminar un stack

```bash
aws cloudformation delete-stack \
    --endpoint-url http://localhost:4566 \
    --stack-name samplestack \
    --region us-east-1
```

### Obtener outputs del stack

```bash
aws cloudformation describe-stacks \
    --endpoint-url http://localhost:4566 \
    --stack-name samplestack \
    --query 'Stacks[0].Outputs' \
    --region us-east-1
```

### Validar template

```bash
aws cloudformation validate-template \
    --endpoint-url http://localhost:4566 \
    --template-body file://s3.yaml \
    --region us-east-1
```

## ✅ Verificar recursos creados

```bash
# Verificar bucket S3
aws s3 ls --endpoint-url http://localhost:4566

# Verificar tabla DynamoDB
aws dynamodb list-tables --endpoint-url http://localhost:4566

# Verificar función Lambda
aws lambda list-functions --endpoint-url http://localhost:4566
```

## 🎯 Ventajas de usar CloudFormation con LocalStack

- **Validación rápida**: Prueba templates sin costos
- **Debugging local**: Logs y errores accesibles localmente
- **Desarrollo iterativo**: Cambios rápidos sin esperar deployments
- **Testing automatizado**: Integra en pipelines de CI/CD

¡Ahora puedes desarrollar y probar tus templates de CloudFormation de forma local y eficiente!
