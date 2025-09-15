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

# Usamos nginx para desplegar el frontend dentro del contenedor
FROM nginxinc/nginx-unprivileged:1.23-alpine-perl

COPY --link nginx.conf /etc/nginx/conf.d/default.conf
COPY --link --from=build usr/src/app/dist/ /usr/share/nginx/html

EXPOSE 8080
