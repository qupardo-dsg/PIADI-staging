# Problemas de valores hardcodeados — PIADI Staging

Documento para revisión del equipo. Cada ítem sigue el formato del tablero Jira (PIADI).

---

## Seguridad (crítico)

### [Seguridad] JWT_SECRET con fallback hardcodeado
- **Tipo:** Error
- **Prioridad:** Highest
- **Archivo:** `backend/src/middleware/authMiddleware.js:4`
- **Problema:** `JWT_SECRET = process.env.JWT_SECRET || 'secret-secreto-super-seguro'`. Si la variable de entorno no está configurada, cualquiera puede forjar tokens JWT válidos.
- **Impacto:** En producción, si Render no tiene la env var `JWT_SECRET` seteada, el sistema de autenticación está comprometido.
- **Solución:** Eliminar el fallback. Si `JWT_SECRET` no existe, el backend debe fallar al iniciar (no usar un secreto conocido).

---

### [Seguridad] DATABASE_URL con credenciales en código fuente
- **Tipo:** Error
- **Prioridad:** High
- **Archivo:** `backend/src/config/database.js:4`
- **Problema:** `DATABASE_URL = process.env.DATABASE_URL || 'postgresql://postgres:postgres@db:5432/mydb'`. Credenciales reales (`postgres:postgres`) expuestas en el código.
- **Impacto:** Si el repo es público o el código se comparte, las credenciales de la base de datos quedan expuestas.
- **Solución:** Eliminar el fallback. Usar solo `process.env.DATABASE_URL` y fallar si no existe.

---

### [Seguridad] CORS completamente abierto
- **Tipo:** Error
- **Prioridad:** High
- **Archivo:** `backend/src/index.js:23`
- **Problema:** `app.use(cors())` sin restricción de origen. Cualquier dominio puede hacer requests al backend.
- **Impacto:** En producción, cualquier sitio web puede consumir la API y acceder a datos (si tiene un token válido o explota otra vulnerabilidad).
- **Solución:** Configurar CORS con origen específico vía env var `CORS_ORIGIN`. Ejemplo: `cors({ origin: process.env.CORS_ORIGIN })`.

---

## Rendimiento

### [Backend] Pool de conexiones a BD hardcodeado
- **Tipo:** Tarea
- **Prioridad:** Medium
- **Archivo:** `backend/src/config/database.js:10-14`
- **Problema:** Pool configurado como `max: 5, min: 0, acquire: 30000, idle: 10000`. No es configurable por variables de entorno.
- **Impacto:** Con tráfico concurrente, `max: 5` puede ser cuello de botella. Si se necesita escalar, hay que modificar código y redeplegar.
- **Solución:** Leer de env vars: `DB_POOL_MAX`, `DB_POOL_MIN`, `DB_POOL_ACQUIRE`, `DB_POOL_IDLE` con valores por defecto razonables.

---

### [Backend] Sequelize loggea todas las queries en producción
- **Tipo:** Tarea
- **Prioridad:** Medium
- **Archivo:** `backend/src/config/database.js:8`
- **Problema:** `logging: console.log` imprime cada query SQL en consola. Siempre activo, sin importar el entorno.
- **Impacto:** En producción, genera ruido en logs, consume I/O y puede exponer información sensible de la base de datos.
- **Solución:** Desactivar en producción: `logging: process.env.NODE_ENV === 'development' ? console.log : false`.

---

### [Backend] Multer sin límite de tamaño de archivo
- **Tipo:** Tarea
- **Prioridad:** Medium
- **Archivo:** `backend/src/routes/plantillaCargaRoutes.js`
- **Problema:** Multer configurado sin `limits.fileSize`. Acepta archivos de cualquier tamaño.
- **Impacto:** Un usuario malintencionado (o un error) podría subir archivos enormes que agoten memoria o disco.
- **Solución:** Agregar límite configurable: `multer({ dest: 'uploads/', limits: { fileSize: 10 * 1024 * 1024 } })` (10 MB por defecto).

---

## Mantenimiento

### [Frontend] API_URL duplicado en 7 archivos
- **Tipo:** Tarea
- **Prioridad:** Low
- **Archivos:**
  - `frontend/src/modules/auth/services/authService.js:1`
  - `frontend/src/modules/carga_datos/views/CargaDatos.hooks.js:5`
  - `frontend/src/modules/central_dashboards/views/CentralDashboards.hooks.js:5`
  - `frontend/src/modules/auditoria/views/Auditoria.hooks.js:303`
  - `frontend/src/context/AuthContext.jsx:121`
  - `frontend/src/services/piadiApi.js:1`
  - `frontend/src/App.jsx:45`
- **Problema:** `const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:5000'` repetido en cada módulo. Ya existe `piadiApi.js` como servicio centralizado, pero no todos lo usan.
- **Impacto:** Al cambiar la URL hay que modificar 7 archivos. Fácil de olvidar uno.
- **Solución:** Centralizar en `piadiApi.js` y que todos los módulos importen desde ahí.

---

### [Backend] Swagger URL hardcodeada a localhost
- **Tipo:** Tarea
- **Prioridad:** Low
- **Archivo:** `backend/src/config/swagger.js:17`
- **Problema:** `url: 'http://localhost:${port}'` hardcodeado. En producción, Swagger apunta a localhost (no al dominio real).
- **Impacto:** Los usuarios de Swagger UI no pueden probar los endpoints desde la URL de producción.
- **Solución:** Usar env var o detectar el host automáticamente: `url: process.env.SWAGGER_URL || 'http://localhost:${port}'`.

---

### [Backend] JWT expiration hardcodeada
- **Tipo:** Tarea
- **Prioridad:** Low
- **Archivo:** `backend/src/controllers/authController.js:79`
- **Problema:** `jwt.sign(payload, JWT_SECRET, { expiresIn: '2h' })`. El tiempo de expiración no es configurable.
- **Impacto:** Para testing prolongado o sesiones de demo, 2h puede ser poco. Para producción estricta, puede ser mucho. No se puede ajustar sin tocar código.
- **Solución:** Leer de env var: `expiresIn: process.env.JWT_EXPIRATION || '2h'`.

---

## Resumen

| Categoría | Cantidad | Prioridad |
|---|---|---|
| Seguridad | 3 | Highest/High |
| Rendimiento | 3 | Medium |
| Mantenimiento | 4 | Low |

**Recomendación:** Los 3 de seguridad deben resolverse antes del deploy a producción. Los de rendimiento y mantenimiento pueden hacerse en sprints posteriores.
