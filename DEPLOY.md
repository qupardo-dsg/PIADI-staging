# Deploy PIADI Staging (Render.com — gratis)

Stack: React+Vite (frontend) + Express/Sequelize (backend) + PostgreSQL.
El backend auto-crea tablas y seed al arrancar (`src/index.js` → `sequelize.sync()` + seeds). No hay migraciones manuales.

## 1. PostgreSQL (gratis)
- Render → `New +` → `PostgreSQL` → plan **Free**
- Copiar el campo **Internal Database URL**

## 2. Backend (gratis)
- `New +` → `Web Service` → conectar repo `PIADI-staging`
- **Root Directory**: `backend`
- **Runtime**: `Node` (auto, usa `package-lock.json`)
- **Build Command**: `npm install`
- **Start Command**: `npm start`
- **Environment** (en el Web Service):
  - `DATABASE_URL` = Internal Database URL del paso 1
  - `PORT` = `5000`
- Deploy. Swagger en `https://<backend>.onrender.com/api-docs`

## 3. Frontend (gratis)
- `New +` → `Static Site` → conectar mismo repo `PIADI-staging`
- **Root Directory**: `frontend`
- **Build Command**: `npm install --legacy-peer-deps && npm run build`
- **Publish Directory**: `dist`
- **Environment**:
  - `VITE_API_URL` = `https://<backend>.onrender.com`
- Deploy. URL pública: `https://<frontend>.onrender.com`

## Notas
- Tier gratis duerme tras ~15 min de inactividad → primera carga tarda 30–60s.
- Disco efímero en Render: archivos subidos vía carga de datos se pierden en cada redeploy. Normal para testing.
- El backend no necesita cambios: lee `DATABASE_URL` y `VITE_API_URL` vía env (ya está en el código).
- CORS abierto en `backend/src/index.js` → accesible desde cualquier dispositivo.