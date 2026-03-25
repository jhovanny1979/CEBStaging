# Documentacion Completa Proyecto CEB

Fecha de corte: 2026-03-25  
Ubicacion principal del workspace: `C:\Proyectos Codex\CEB`

---

## 1) Resumen ejecutivo

Este proyecto implementa la plataforma **Comercio e-Bogota** en tres capas:

1. **Frontend Web** (HTML/CSS/JS) para landing, directorio, registro, panel cliente y panel admin.
2. **Backend API** (FastAPI + SQLAlchemy + Alembic) con reglas de negocio, autenticacion, promociones, suscripciones y administracion.
3. **App movil Android** (Capacitor/WebView) que consume la misma plataforma via red local (Wi-Fi).

Adicionalmente existen scripts de operacion para:

1. arranque y parada local web/backend,
2. arranque para modo movil con apertura de firewall,
3. build e instalacion de APK,
4. verificacion de codificacion UTF-8 para evitar texto corrupto.

---

## 2) Alcance y carpetas del proyecto

### 2.1 Carpetas principales en raiz

Ruta: `C:\Proyectos Codex\CEB`

1. `backend`  
   Backend base (legacy/capa tecnica principal).
2. `comercioebogota_3\comercioebogota`  
   Frontend base (legacy/look and feel previo).
3. `mobile-apk`  
   Modulo mobil en raiz.
4. `docs`  
   Documentacion tecnica y funcional.
5. `CEB_v1`  
   **Copia de trabajo integral** (backend + frontend + mobile + scripts) usada para evolucion de version nueva.

### 2.2 Carpeta recomendada para trabajo activo

Ruta recomendada: `C:\Proyectos Codex\CEB\CEB_v1`

Motivo: concentra backend, frontend, mobile y scripts integrados con runtime dinamico (`.runtime\ceb-runtime.env`).

---

## 3) Arquitectura tecnica

## 3.1 Backend

Stack:

1. FastAPI
2. SQLAlchemy 2
3. Alembic
4. Pydantic v2 / pydantic-settings
5. psycopg (PostgreSQL)
6. SQLite fallback para iteracion local
7. APScheduler (job de recordatorios de vencimiento)

Archivo principal:

1. `C:\Proyectos Codex\CEB\CEB_v1\backend\app\main.py`

Elementos clave:

1. `lifespan` con scheduler cada 1 hora.
2. Mount de archivos subidos en `/uploads`.
3. CORS abierto en `development` para evitar friccion local.

## 3.2 Frontend web

Ubicacion:

1. `C:\Proyectos Codex\CEB\CEB_v1\comercioebogota_3\comercioebogota`

Paginas principales:

1. `index.html` (landing + planes + navegacion principal)
2. `registro.html` (alta de clientes)
3. `panel.html` (panel cliente)
4. `admin.html` (panel administrador)
5. `negocio.html` (directorio y pagina publica de negocios)

JS clave:

1. `js/api-client.js`: cliente API, token handling, fallback de base URL.
2. `js/app.js`: logica UI global, formato local, modo app (`?app=1`), fixes responsivos.
3. `js/index-plans.js`: render y calculo de planes segun configuracion admin.
4. `js/panel-whatsapp-module.js`: modulo CRM/WA (contactos, promos, envio).

## 3.3 App movil Android

Ubicacion:

1. `C:\Proyectos Codex\CEB\CEB_v1\mobile-apk`

Tecnologia:

1. Capacitor (Android WebView)

Comportamiento:

1. usa `www/host-config.json` para host/puertos en runtime,
2. permite conexion por Wi-Fi al host local,
3. inyecta navegacion inferior movil en modo `?app=1`.

---

## 4) Configuracion y variables de entorno

Archivo base:

1. `C:\Proyectos Codex\CEB\CEB_v1\backend\.env.example`

Variables mas importantes:

1. `DATABASE_URL` (obligatoria)
2. `TEST_DATABASE_URL`
3. `SECRET_KEY`
4. `API_V1_PREFIX` (default `/api/v1`)
5. `APP_LOCALE` (default `es-419`)
6. `APP_TIMEZONE` (default `America/Bogota`)
7. `UPLOAD_DIR` (default `../var/uploads`)
8. `ALLOWED_ORIGINS`
9. `RATE_LIMIT_LOGIN_PER_MINUTE`
10. `RATE_LIMIT_RECOVER_PER_MINUTE`

Nota:

1. si falta `DATABASE_URL`, el backend no inicia.
2. en modo sqlite los scripts inyectan `DATABASE_URL` temporal automaticamente.

---

## 5) Arranque, parada y operacion diaria

## 5.1 Arranque web local recomendado (CEB_v1)

Desde `C:\Proyectos Codex\CEB\CEB_v1`:

1. `START_CEB.bat`

Este script:

1. asigna puertos libres (frontend 5500..5510, backend 8000..8010),
2. levanta backend y frontend,
3. detecta IP LAN,
4. guarda runtime en `.runtime\ceb-runtime.env`,
5. actualiza `mobile-apk\www\host-config.json`.

## 5.2 Parar servicios

Desde `C:\Proyectos Codex\CEB\CEB_v1`:

1. `STOP_CEB.bat`

## 5.3 Modo movil (Wi-Fi)

Desde `C:\Proyectos Codex\CEB\CEB_v1`:

1. ejecutar `START_CEB_MOVIL.bat` como Administrador.

Acciones del script:

1. fuerza perfil de red privada,
2. abre reglas firewall para puertos frontend/backend activos,
3. actualiza configuracion host de app movil,
4. valida `http://LAN_IP:FRONT_PORT/index.html?app=1`.

## 5.4 Build APK

Desde `C:\Proyectos Codex\CEB\CEB_v1`:

1. `BUILD_APK.bat`
2. `INSTALL_APK.bat` (si se instala por USB)

Salida esperada:

1. `C:\Proyectos Codex\CEB\CEB_v1\CEB-mobile-debug.apk`

---

## 6) URLs operativas

Con runtime dinamico, validar `.runtime\ceb-runtime.env`.

Patrones:

1. Front local: `http://127.0.0.1:<FRONT_PORT>/index.html`
2. Backend local: `http://127.0.0.1:<BACK_PORT>`
3. Swagger: `http://127.0.0.1:<BACK_PORT>/docs`
4. Front LAN: `http://<LAN_IP>:<FRONT_PORT>/index.html`
5. App mode LAN: `http://<LAN_IP>:<FRONT_PORT>/index.html?app=1`
6. Health backend LAN: `http://<LAN_IP>:<BACK_PORT>/health/live`

---

## 7) API funcional (v1)

Prefijo base: `/api/v1`

## 7.1 Auth

1. `POST /auth/register`
2. `POST /auth/login`
3. `POST /auth/verify-email`
4. `POST /auth/recover`
5. `POST /auth/reset-password`
6. `GET /auth/me`

## 7.2 Cliente (`/me`)

Negocio:

1. `GET /me/business`
2. `PUT /me/business`
3. `POST /me/business/images`
4. `DELETE /me/business/images`

Promociones:

1. `POST /me/promotions/images`
2. `GET /me/promotions`
3. `POST /me/promotions`
4. `PUT /me/promotions/{promotion_id}`
5. `POST /me/promotions/{promotion_id}/publish`
6. `POST /me/promotions/{promotion_id}/relaunch`
7. `DELETE /me/promotions/{promotion_id}`

Suscripciones y pagos:

1. `GET /me/plans`
2. `GET /me/subscriptions/current`
3. `POST /me/subscriptions/activate-code`
4. `POST /me/subscriptions/upgrade`
5. `POST /me/payments/receipts`

Modulo WA:

1. `GET /me/wa/contacts`
2. `POST /me/wa/contacts`
3. `DELETE /me/wa/contacts/{contact_id}`
4. `GET /me/wa/promotions`
5. `POST /me/wa/promotions`
6. `DELETE /me/wa/promotions/{promotion_id}`
7. `POST /me/wa/promotions/images`
8. `GET /me/wa/sent`
9. `POST /me/wa/sent`

## 7.3 Admin (`/admin`)

Acceso:

1. `POST /admin/login`

Usuarios y credenciales:

1. `GET /admin/users`
2. `POST /admin/users`
3. `DELETE /admin/users/{user_id}`
4. `POST /admin/users/{user_id}/set-password`
5. `POST /admin/clients/{user_id}/set-password`

Configuracion plataforma:

1. `GET /admin/plans`
2. `PUT /admin/plans`
3. `PUT /admin/limits`
4. `GET /admin/platform-settings`
5. `PUT /admin/platform-settings`

Operacion:

1. `GET /admin/businesses`
2. `GET /admin/clients`
3. `GET /admin/promotions`
4. `GET /admin/receipts`
5. `POST /admin/receipts/{receipt_id}/approve`
6. `POST /admin/receipts/{receipt_id}/reject`
7. `GET /admin/dashboard`
8. `GET /admin/audit-logs`
9. `POST /admin/promo-codes`
10. `GET /admin/outbox`

## 7.4 Sistema / publico

1. `GET /health/live`
2. `GET /health/ready`
3. `GET /public/plans`
4. `GET /public/platform-settings`
5. `GET /public/businesses`
6. `GET /public/businesses/{slug}`

---

## 8) Modelo de datos

Archivo:

1. `C:\Proyectos Codex\CEB\CEB_v1\backend\app\models\entities.py`

Tablas de dominio:

1. `users`
2. `email_verification_tokens`
3. `password_reset_tokens`
4. `businesses`
5. `business_hours`
6. `business_images`
7. `promotions`
8. `promotion_images`
9. `wa_contacts`
10. `wa_promotion_templates`
11. `wa_promotion_sends`
12. `subscription_plans`
13. `platform_settings`
14. `subscriptions`
15. `promo_codes`
16. `promo_code_redemptions`
17. `payment_receipts`
18. `notification_outbox`
19. `audit_logs`

Reglas/constraints relevantes:

1. email usuario unico (`users.email`).
2. slug de negocio unico (`businesses.slug`).
3. 1 negocio por owner (`businesses.owner_user_id` unico).
4. horario unico por dia (`business_hours`).
5. posicion unica por imagen de negocio/promocion.
6. contacto WA unico por negocio + telefono.
7. redencion de codigo promo unica por negocio/codigo.

---

## 9) Migraciones

Carpeta:

1. `C:\Proyectos Codex\CEB\CEB_v1\backend\alembic\versions`

Historial:

1. `20260310_0001_initial.py` (schema inicial)
2. `20260312_0002_wa_module.py` (tablas WA)
3. `20260312_0003_platform_settings.py` (settings globales)
4. `20260313_0004_promotion_images.py` (imagenes por promocion)

Comando base:

1. `python -m alembic upgrade head`

---

## 10) Seeds y credenciales de desarrollo

Seed script:

1. `C:\Proyectos Codex\CEB\CEB_v1\backend\scripts\seed_data.py`

Datos iniciales:

1. Admin local: `admin@comercioebogota.com` / `admin123`
2. Usuario demo: `demo@comercioebogota.com` / `Demo12345!`
3. Planes base (`PLAN_1M`, `PLAN_3M`, `PLAN_6M`, `PLAN_12M`)
4. Codigos promo demo (`DEMO30D1`, `PROMO2024`, `BOGO2024A`)

Nota:

1. seed conserva configuracion de planes ya ajustada por admin cuando corresponde.

---

## 11) Reglas de negocio implementadas (resumen)

1. periodo de prueba configurable por admin (`platform_settings.trial_days`).
2. promociones operativas con limite mensual (4/mes) y control semanal.
3. publicacion/relaunch de promociones con estados (`draft`, `published`, etc).
4. codigos promocionales para activar dias de prueba.
5. suscripciones por plan y carga de soporte de pago.
6. aprobacion/rechazo admin de comprobantes.
7. recordatorios de vencimiento por job programado.
8. RBAC basico (`client`, `admin`) + JWT.

---

## 12) Frontend: paginas y responsabilidades

### 12.1 `index.html`

1. landing principal,
2. seccion de caracteristicas,
3. seccion planes (datos desde admin),
4. CTA y footer.

### 12.2 `registro.html`

1. registro de cliente,
2. creacion inicial negocio,
3. activacion opcional por codigo promo.

### 12.3 `panel.html`

1. mi cuenta y suscripcion,
2. mi negocio (datos, logo, imagenes, horario, redes),
3. promociones del negocio,
4. modulo WA (contactos, plantillas, envio).

### 12.4 `admin.html`

1. dashboard,
2. negocios/usuarios/suscripciones/promociones,
3. codigos promo,
4. configuracion global de planes y limites.

### 12.5 `negocio.html`

1. directorio publico de negocios,
2. filtros por nombre/localidad/categoria,
3. vista publica de detalle por negocio.

---

## 13) Scripts de soporte (CEB_v1)

En `C:\Proyectos Codex\CEB\CEB_v1`:

1. `START_CEB.bat`  
   arranque backend + frontend (puertos dinamicos).
2. `STOP_CEB.bat`  
   detencion de procesos y puertos.
3. `START_CEB_MOVIL.bat`  
   arranque para movil con red/firewall.
4. `BUILD_APK.bat`  
   compilar APK debug.
5. `INSTALL_APK.bat`  
   instalar APK en dispositivo USB.
6. `CHECK_CEB_MOVIL.bat`  
   validacion de conectividad movil.
7. `FIX_FORMATO_FRONT.bat`  
   apoyo para correcciones de formato/codificacion.
8. `VER_BD_WEB.bat`  
   inspeccion DB por vista web local.
9. `VER_DATOS.bat`  
   inspeccion rapida de datos.

---

## 14) Pruebas

Carpeta:

1. `C:\Proyectos Codex\CEB\CEB_v1\backend\tests`

Suites existentes:

1. `test_auth.py`
2. `test_business_promotions.py`
3. `test_subscription_admin.py`
4. `test_admin_settings.py`
5. `test_admin_panel_data.py`
6. `test_wa_module.py`
7. `test_promotion_images.py`

Comando recomendado:

1. `python -m pytest .\tests -q -p no:cacheprovider`

---

## 15) Codificacion y prevencion de caracteres raros

Problema historico:

1. mojibake por mezcla de codificaciones al editar HTML/CSS/JS.

Mecanismos implementados:

1. `encoding_guard.py` para detectar/corregir.
2. `start-front.ps1` ejecuta check/fix antes de servir.
3. `serve_front_utf8.py` fuerza headers UTF-8 en contenido textual.

Buenas practicas:

1. guardar siempre en UTF-8.
2. evitar editores con codificacion ANSI/Windows-1252 en estos archivos.
3. mantener strings criticos sin simbolos ambiguos si se requiere robustez extra.

---

## 16) Operacion movil y troubleshooting rapido

Checklist cuando no abre en celular:

1. ejecutar `START_CEB_MOVIL.bat` como Administrador.
2. confirmar misma red Wi-Fi entre PC y celular.
3. validar URL impresa por script (IP + puerto actual, no asumir valores viejos).
4. probar en PC: `http://LAN_IP:FRONT_PORT/index.html?app=1`.
5. probar backend: `http://LAN_IP:BACK_PORT/health/live`.
6. revisar firewall de Windows (reglas creadas por script).
7. si cambia IP, recompilar APK (`BUILD_APK.bat`) e instalar.

Archivos clave movil:

1. `mobile-apk\capacitor.config.json`
2. `mobile-apk\www\host-config.json`
3. `mobile-apk\scripts\update-capacitor-config.ps1`

---

## 17) Git y repos remotos

Estado actual del repo local (`C:\Proyectos Codex\CEB`):

1. rama activa: `main`
2. `origin`: `https://github.com/jhovanny1979/CEBStaging.git`
3. `gitlab`: `https://bitlab.bitram.co/jhovanny.duenas/CEBStaging.git`

Flujo sugerido:

1. `git add -A`
2. `git commit -m "mensaje"`
3. `git push origin main`
4. `git push gitlab main` (si tambien se sincroniza Bitlab)

---

## 18) Estado funcional esperado (referencia)

Si todo esta arriba correctamente:

1. login admin debe ingresar con `admin/admin123` o `admin@comercioebogota.com/admin123`.
2. login cliente debe autenticar y redirigir a `panel.html`.
3. `negocio.html` debe listar negocios publicados segun API.
4. dashboard admin debe reflejar metricas coherentes con tabla de suscripciones.
5. cambios en configuracion admin (planes/limites) deben verse en landing y panel cliente.

---

## 19) Archivo de referencia funcional ya existente

1. `C:\Proyectos Codex\CEB\docs\matriz-validacion.md`
2. `C:\Proyectos Codex\CEB\docs\estructura-proyecto.md`

Este documento complementa y amplifica esas referencias.

---

## 20) Proxima recomendacion

Para evitar perdida de contexto futura:

1. mantener este archivo como documento maestro,
2. actualizar fecha de corte y secciones afectadas en cada release relevante,
3. agregar un `CHANGELOG.md` con cambios por version (`v1`, `v1.x`).

