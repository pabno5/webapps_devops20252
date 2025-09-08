Antes de comenzar verifica que no tengas ningun contenedor corriendo con Docker Desktop, especialmente los que ocupen el puerto `5432`

# Desarrollo manual

## api-node

1. Instalar NVM
  - en Windows: [https://github.com/coreybutler/nvm-windows](https://github.com/coreybutler/nvm-windows)
  - en Linux: [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

2. Lanzar contenedor PostgreSQL `docker run -d -e POSTGRES_PASSWORD=foobarbaz -p 5432:5432 postgres:15.1-alpine`

3. Nos ubicamos en la carpeta `api-node` con una terminal **PowerShell**

```bash
nvm install 19.4
nvm use 19.4
npm install
$env:DATABASE_URL="postgres://postgres:foobarbaz@localhost:5432/postgres"
npm run dev
```

# Desarrollo con Docker

Un Dockerfile es un archivo de texto que contiene todos los comandos para crear una imagen de contenedor, es como una receta para la aplicación.

## .dockerignore

Al crear una imagen de docker puedes incluir un archivo `.dockerignore` para ignorar ciertos archivos (por ejemplo la carpeta local `node_modules`) para que no sean copiados a la imagen del contenedor, de esa manera evitamos conflictos e incompatibilidades entre instalaciones del contenedor.

## Dockerfile

En el archivo `Dockerfile` se definen los pasos para construir la imagen de Docker. Para construir y ejecutar las aplicaciones de esta sesión se definen los siguientes pasos:

1. Iniciar el sistema operativo
2. Instalar el lenguaje de programación
3. Instalar dependencias del proyecto o aplicación
4. Configurar entorno de ejecución
5. Correr la aplicación.

## Comandos generales en un Dockerfile

```
FROM: Especifica el sistema operativo (generalmente) para la imagen del contenedor.
RUN: Ejecuta un comando durante la fase de CONSTRUCCIÓN.
COPY: Copia archivos desde la carpeta del proyecto a la imagen del contenedor.
CMD: Provee un comando a ser ejecutado cuando inicie el contenedor.
```

## Comando para construir contenedor

`docker build .`
