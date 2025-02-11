# 📌 Guía para Configurar y Ejecutar Lambdas con API Gateway en LocalStack

## 🚀 Introducción
Esta guía explica cómo configurar un entorno de desarrollo local usando **LocalStack** para ejecutar **AWS Lambdas** y exponerlas a través de **API Gateway**.

Con esta configuración, cualquier desarrollador podrá probar y ejecutar su API en un entorno controlado y simulado de AWS.

---

## ✅ 1. Pre-requisitos
Antes de comenzar, asegúrese de tener instalados los siguientes requisitos:

### 🛠️ Herramientas necesarias:
- [Docker](https://www.docker.com/get-started) (Requerido para ejecutar LocalStack)
- [AWS CLI](https://aws.amazon.com/cli/) (Configurar con `aws configure` aunque LocalStack no lo requiera directamente)
- [LocalStack](https://localstack.cloud/) (Para simular AWS en local)
- [Python 3.x](https://www.python.org/downloads/) (Para ejecutar las funciones Lambda)
- [Postman](https://www.postman.com/downloads/) (Para probar las APIs expuestas)

---

## 🔥 2. Instalación de LocalStack

Ejecuta el siguiente comando para instalar LocalStack:

```cmd
pip install localstack
```

Verifica que la instalación fue exitosa ejecutando:

```cmd
localstack --version
```

Si usas **Docker**, ejecuta LocalStack con persistencia:

```cmd
docker run --rm -it -p 4566:4566 -v /var/run/docker.sock:/var/run/docker.sock -v %USERPROFILE%\repos\localstackData:/var/lib/localstack -e PERSISTENCE=1 localstack/localstack
```

Para verificar que LocalStack está corriendo correctamente:
```cmd
aws s3 ls --endpoint-url http://localhost:4566
```

---

## ⚙️ 3. Configuración de AWS CLI para LocalStack

Debemos configurar **AWS CLI** para que funcione con LocalStack:

```cmd
aws configure set aws_access_key_id test
aws configure set aws_secret_access_key test
aws configure set region us-east-1
```

Para verificar la conexión con LocalStack, ejecuta:

```cmd
aws s3 ls --endpoint-url http://localhost:4566
```

Debería responder sin errores.

---

## 🚀 4. Despliegue de Lambdas y API Gateway

### 📌 4.1 Crear un Bucket S3 para almacenar el código de las Lambdas
```cmd
aws s3 mb s3://my-bucket-name --endpoint-url http://localhost:4566
```

### 📌 4.2 Subir el código de las Lambdas a S3
```cmd
cd lambda-functions\autenticacion && zip -r autenticacion.zip . && cd ..
cd authorizer && zip -r authorizer.zip . && cd ..
cd canje_estrella && zip -r canje_estrella.zip . && cd ..
cd consulta_estrella && zip -r consulta_estrella.zip . && cd ..
cd devolucion_estrella && zip -r devolucion_estrella.zip . && cd ..
cd reversa_linea_estrella && zip -r reversa_linea_estrella.zip . && cd ..
cd estrellas_layer && zip -r estrellas_layer.zip . && cd ..
```

Subir los archivos comprimidos a S3:
```cmd
aws s3 cp lambda-functions\autenticacion\autenticacion.zip s3://my-bucket-name/lambda-functions/autenticacion/autenticacion.zip --endpoint-url http://localhost:4566

aws s3 cp lambda-functions\authorizer\authorizer.zip s3://my-bucket-name/lambda-functions/authorizer/authorizer.zip --endpoint-url http://localhost:4566

aws s3 cp lambda-functions\canje_estrella\canje_estrella.zip s3://my-bucket-name/lambda-functions/canje_estrella/canje_estrella.zip --endpoint-url http://localhost:4566

aws s3 cp lambda-functions\consulta_estrella\consulta_estrella.zip s3://my-bucket-name/lambda-functions/consulta_estrella/consulta_estrella.zip --endpoint-url http://localhost:4566

aws s3 cp lambda-functions\devolucion_estrella\devolucion_estrella.zip s3://my-bucket-name/lambda-functions/devolucion_estrella/devolucion_estrella.zip --endpoint-url http://localhost:4566

aws s3 cp lambda-functions\reversa_linea_estrella\reversa_linea_estrella.zip s3://my-bucket-name/lambda-functions/reversa_linea_estrella/reversa_linea_estrella.zip --endpoint-url http://localhost:4566

aws s3 cp lambda-functions\estrellas_layer\estrellas_layer.zip s3://my-bucket-name/estrellas_layer.zip --endpoint-url http://localhost:4566
```

### 📌 4.3 Desplegar CloudFormation en dos Stacks
Para evitar problemas con las rutas en API Gateway, se recomienda separar la creación de **Lambdas** y **API Gateway** en dos stacks diferentes:

#### **4.3.1 Desplegar Stack de API Gateway**
```cmd
aws cloudformation create-stack --stack-name api-gateway-stack --template-body file://src/template.yml --endpoint-url http://localhost:4566
```

#### **4.3.2 Desplegar Stack de Lambdas**
```cmd
aws cloudformation create-stack --stack-name lambdas-stack --template-body file://src/template.yml --parameters ParameterKey=S3BucketName,ParameterValue=my-bucket-name --endpoint-url http://localhost:4566
```

Para listar los stacks creados:
```cmd
aws cloudformation list-stacks --endpoint-url http://localhost:4566
```

Para eliminar un stack específico:
```cmd
aws cloudformation delete-stack --stack-name my-stack --endpoint-url http://localhost:4566

aws cloudformation delete-stack --stack-name lambdas-stack --endpoint-url http://localhost:4566

```

### 📌 4.4 Obtener IDs y Construir la URL para Postman
Obtener el ID del API Gateway:
```cmd
aws apigateway get-rest-apis --endpoint-url http://localhost:4566
```
Obtener los recursos disponibles:
```cmd
aws apigateway get-resources --rest-api-id lox5ptui4r --endpoint-url http://localhost:4566
```
Construir la URL de invocación:
```
http://localhost:4566/restapis/d6ti2hlr8o/dev/_user_request_/v1/autenticacion
```

---

## 🎉 Otros comandos

aws lambda get-function-configuration --function-name LambdaAutenticacion --endpoint-url http://localhost:4566

aws lambda invoke --function-name LambdaAutenticacion --payload '{}' --cli-binary-format raw-in-base64-out --endpoint-url http://localhost:4566 response.json

aws lambda update-function-configuration --function-name LambdaAutenticacion --layers arn:aws:lambda:us-east-1:000000000000:layer:estrellasLayer:1 --endpoint-url http://localhost:4566

aws lambda delete-function --function-name LambdaAutenticacion --endpoint-url http://localhost:4566


aws lambda create-function --function-name LambdaAutenticacion --runtime python3.12 --role arn:aws:iam::000000000000:role/LambdaExecutionRole --handler index.lambda_handler --code S3Bucket=my-bucket-name,S3Key=lambda-functions/autenticacion/autenticacion.zip --layers arn:aws:lambda:us-east-1:000000000000:layer:estrellasLayer:1 --endpoint-url http://localhost:4566



$env:AWS_ACCESS_KEY_ID="test"
$env:AWS_SECRET_ACCESS_KEY="test"
$env:AWS_DEFAULT_REGION="us-east-1"