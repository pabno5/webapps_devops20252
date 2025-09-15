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

CMD ["npm", "run", "dev"]
