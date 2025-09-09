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

## Dockerfile

En el archivo `Dockerfile` se definen los pasos para construir la imagen de Docker. Para construir y ejecutar las aplicaciones de esta sesión se definen los siguientes pasos:

1. Iniciar el sistema operativo
2. Instalar el lenguaje de programación
3. Instalar dependencias del proyecto o aplicación
4. Configurar entorno de ejecución
5. Correr la aplicación.

## .dockerignore

Al crear una imagen de docker puedes incluir un archivo `.dockerignore` para ignorar ciertos archivos (por ejemplo la carpeta local `node_modules`) para que no sean copiados a la imagen del contenedor, de esa manera evitamos conflictos e incompatibilidades entre instalaciones del contenedor.

```
node_modules
.npm  
```

## Comandos generales en un Dockerfile

```
FROM: Especifica el sistema operativo (generalmente) para la imagen del contenedor.
RUN: Ejecuta un comando durante la fase de CONSTRUCCIÓN.
COPY: Copia archivos desde la carpeta del proyecto a la imagen del contenedor.
CMD: Provee un comando a ser ejecutado cuando inicie el contenedor.
```

## api-node

1. Imagen básica

``` docker
# Especificamos sis. operativo y dependencias a partir de una imagen ya hecha (node)
FROM node

# Copiamos el contenido del proyecto a la imagen Docker
COPY . .

# Ejecutamos instalación de dependencias al construir la imagen
RUN npm install

# Ejecutamos proyecto al inciar el contenedor
CMD [ "npm", "run", "dev" ]
```

[https://hub.docker.com/_/node](https://hub.docker.com/_/node)

2. Traer una versión especifica para reproducibilidad*

``` docker
# FROM node  
FROM node:24-alpine3.21
```

3. Copiar solo lo que necesitamos en el momento

``` docker
# COPY . .  
COPY package*.json ./

RUN npm install

COPY ./src .
```

4. Definir carpeta de ejecución dentro de la imagen

``` docker
FROM node:24-alpine3.21
WORKDIR /usr/src/app  
```

5. Ejecutar entorno de producción

``` docker
# CMD [ "npm", "run", "dev" ]
CMD ["node", "index.js"]  
```

6. Definir usuario con permisos limitados

``` docker
RUN npm install
USER node  

# COPY ./src .
COPY -chown=node:node ./src .
```

7. Variables de entorno dentro de la imagen

``` docker
# WORKDIR /usr/src/app  
ENV NODE_ENV=production
```

8. Instalación ultra-optimizada de dependencias

``` docker
# RUN npm install
RUN npm ci --only=production
```

9. Conectar puerto al exterior*

``` docker
COPY -chown=node:node ./src .
EXPOSE 3000  
```

## Comando para construir contenedor

La notación usada para definir la VERSION comienza en 0 y a medida que haces cambios al readme, puedes incrementar la versión de tu imagen Docker.

`docker build -t devops-api-node:VERSION .`

> [!IMPORTANT]  
> Revisa el nombre y version de tus imagenes creadas con el comando `docker images`

### Ejecutar contenedor

1. Crear red interna de contenedores

```sh
docker network create devops-network
```

> [!IMPORTANT]  
> Manten la tecla `Shift` al pegar comandos multilinea para que no sea ejecutado automaticamente y lo puedas editar

2. Crear contenedor: base de datos

``` bash
docker run -d `
  --name devops-postgres `
  --network devops-network `
  -e POSTGRES_PASSWORD=foobarbaz `
  -v pgdata:/var/lib/postgresql/data `
  -p 5432:5432 `
  --restart unless-stopped `
  postgres:15.1-alpine
```

3. Crear contenedor: api-node

``` bash
docker run -d `
  --name contenedor-devops-api-node `
  --network devops-network `
  -e DATABASE_URL="postgres://postgres:foobarbaz@devops-postgres:5432/postgres" `
  -p 3000:3000 `
  --restart unless-stopped `
  devops-api-node  
```

> [!TIP]  
> `docker ps` para verificar contenedores en ejecución. `docker ps -a` para incluir contenedores apagados.
