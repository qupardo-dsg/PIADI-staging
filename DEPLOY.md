# Deploy PIADI Staging (Render.com — gratis)

Stack: React+Vite (frontend) + Express/Sequelize (backend) + PostgreSQL.
El backend auto-crea tablas y seed al arrancar (`src/index.js` → `sequelize.sync()` + seeds). No hay migraciones manuales.

## 1. PostgreSQL (gratis)
- Render → `New +` → `PostgreSQL` → plan **Free**
- Copiar el campo **Internal Database URL**

## 2. Backend (gratis)
- `New +` → `Web Service` → conectar repo `PIADI-staging`
- **Root Directory**: `backend`
- **Runtime**: `Node`
- **Build Command**: `npm install`
- **Start Command**: `npm start`
- **Environment**:
  - `DATABASE_URL` = Internal Database URL del paso 1
  - `PORT` = `5000`
- Deploy. Swagger en `https://<backend>.onrender.com/api-docs`

## 3. Frontend (gratis)
- `New +` → `Static Site` → conectar repo `PIADI-staging`
- **Root Directory**: `frontend`
- **Build Command**: `npm install --legacy-peer-deps && npm run build`
- **Publish Directory**: `dist`
- **Environment**:
  - `VITE_API_URL` = `https://<backend>.onrender.com`
- Deploy. URL pública: `https://<frontend>.onrender.com`

## Docker local (opcional)
```bash
cp .env.example .env
docker compose up --build
```
- Frontend: http://localhost
- Backend: http://localhost:5000
- Swagger: http://localhost:5000/api-docs

## Datos seed (usuarios de prueba)
- `educacioncontinua@ecas.cl` / `admin123` (Educación Continua)
- `rectoria@ecas.cl` / `admin123` (Rector)
- `calidad@ecas.cl` / `admin123` (Vicerrectoría de Calidad)

## Notas
- Tier gratis duerme tras ~15 min de inactividad → primera carga tarda 30–60s.
- Disco efímero: archivos subidos vía carga de datos se pierden en cada redeploy.
- CORS abierto → accesible desde cualquier dispositivo.
- Frontend sirve con nginx (producción), no Vite dev server.
