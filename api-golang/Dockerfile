# ESTO ES EL CONTENIDO DEL ARCHIVO Dockerfile EN LA CARPETA api-golang
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