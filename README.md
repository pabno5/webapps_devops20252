# api-node

1. Instalar NVM
  - en Windows: [https://github.com/coreybutler/nvm-windows](https://github.com/coreybutler/nvm-windows)

2. Lanzar contenedor PostgreSQL `docker run -e POSTGRES_PASSWORD=foobarbaz -p 5432:5432 postgres:15.1-alpine`

3. Nos ubicamos en la carpeta `api-node`

```bash
nvm install 19.4
nvm use 19.4
npm install
$env:DATABASE_URL=postgres://postgres:foobarbaz@localhost:5432/postgres npm run dev
```
