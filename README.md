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

`docker run -d --name devops-postgres --network devops-network -e POSTGRES_PASSWORD=foobarbaz -v pgdata:/var/lib/postgresql/data -p 5432:5432 --restart unless-stopped postgres:15.1-alpine`

3. Crear contenedor: api-node

`docker run -d --name contenedor-devops-api-node --network devops-network -e DATABASE_URL="postgres://postgres:foobarbaz@devops-postgres:5432/postgres" -p 3000:3000 --restart unless-stopped devops-api-node:0`

> [!TIP]  
> `docker ps` para verificar contenedores en ejecución. `docker ps -a` para incluir contenedores apagados.

# Objetivos

Ya teniendo los contenedores de `postgres` y `api-node`, construye las imagenes (y ejecuta los contenedores) para los proyectos `api-golang` y `client-react` a partir de los siguientes Dockerfile:

## api-golang

```docker
FROM golang:1.19-bullseye AS build

# Creamos usuario con privilegios minimos
RUN useradd -u 1001 nonroot

WORKDIR /app 

# Copia solo los archivos requeridos para instalar (construcción de imagen más rapida)
COPY go.mod go.sum ./

# Instalación de dependencias en Go con cache para no repetir instalación de las dependencias existentes
RUN --mount=type=cache,target=/go/pkg/mod \
  --mount=type=cache,target=/root/.cache/go-build \
  go mod download

COPY . .

# # Compila verificación
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o healthcheck \
  ./healthcheck/healthcheck.go

# Compilamos la aplicación durante creación de la imagen y no en la ejecución del contenedor
RUN go build \
  -ldflags="-linkmode external -extldflags -static" \
  -tags netgo \
  -o api-golang

# Ya compilado creamos un entorno aislado que solo ejecute el binario compilado
FROM scratch

# Configuramos variable de entorno
ENV GIN_MODE=release

WORKDIR /

# # Copia de archivos necesarios (contraseña de usuario, healthcheck, archivo compilado)
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /app/healthcheck/healthcheck healthcheck
COPY --from=build /app/api-golang api-golang

# Creamos usuario con privilegios minimos
USER nonroot

# Exponemos el puerto 8080 para comunicación
EXPOSE 8080

CMD ["/api-golang"]
```

Crear contenedor

`docker run -d --name contenedor-devops-api-golang --network devops-network -e DATABASE_URL="postgres://postgres:foobarbaz@devops-postgres:5432/postgres" -p 8080:8080 --restart unless-stopped devops-api-golang:0`

## client-react

```docker
FROM node:19.4-bullseye AS build

# Especificamos directorio de trabajo (si no todo quedaria en la / raiz lo cual es mala práctica)
WORKDIR /usr/src/app

# Copiamos solo el archivo sobre declaración de dependencias necesarias, para optimizar construcción de la imagen
COPY package*.json ./

# Utilizamos funcionalidad de npm cache para agilizar construcción de imagen
RUN --mount=type=cache,target=/usr/src/app/.npm \
  npm set cache /usr/src/app/.npm && \
  npm install

COPY . .

RUN npm run build

# Usamos una base diferente para desplegar frontend dentro del contenedor
FROM nginxinc/nginx-unprivileged:1.23-alpine-perl

COPY --link nginx.conf /etc/nginx/conf.d/default.conf
COPY --link --from=build usr/src/app/dist/ /usr/share/nginx/html

EXPOSE 8080
```

Crear contenedor

`docker run -d --name contenedor-devops-client-react --network devops-network -p 80:8080 --restart unless-stopped devops-client-react:0`

## Resultado final

En tu navegador dirigete a la ruta `localhost` y veras el resultado.

![react page screenshot](screenshot.webp)

