# Registro de seguridad · Sistema Alumnos

Estudiante: [Tu Nombre Completo]   Repositorio: [URL de tu repo Git]   Fecha: 27-11-2026

Completa este registro a medida que avanzas. Cada fila debe poder seguirse hasta un **build de Jenkins**
y un **commit** (usa el identificador `VULN-xx` en el mensaje del commit).

## 1. Revisiones de seguridad realizadas

| # | Etapa del SDLC | Herramienta / técnica | Build | Resultado (resumen) |
|---|---|---|---|---|
| R1 | Código (SAST) | Bandit (PowerShell / pipeline) | #__ | 1 hallazgo High (`B201`), 3 Medium (`B324`, `B608`, `B104`), 2 Low (`B105`) -> 0 tras mitigar |
| R2 | Dependencias (SCA) | pip-audit / Dependabot | #__ | Vulnerabilidades detectadas en Flask, Werkzeug, Jinja2 y Gunicorn |
| R3 | Pruebas dinámicas (DAST) | OWASP ZAP sobre staging | #__ | Alertas de cabeceras HTTP faltantes y parámetros reflejados -> Mitigadas |
| R4 | Revisión manual | Lectura de `app.py` | — | Se identificaron 6 vulnerabilidades (`VULN-01` a `VULN-06`) en rutas, auth y errores |
| R5 | Producción | Grafana + Prometheus | — | Monitoreo activo de `login_fallidos_total` y `peticiones_sospechosas_total` |

## 2. Vulnerabilidades identificadas y corregidas

| ID | Vulnerabilidad (CWE) | Detectada por | Ubicación | Riesgo | Corrección aplicada | Commit | Build que lo verifica |
|---|---|---|---|---|---|---|---|
| VULN-01 | Inyección SQL (`CWE-89`) | Bandit (`B608`) / Revisión manual | `app.py:192` (`/alumnos/buscar`) | Crítico: extracción o manipulación de la BD SQLite mediante el parámetro `nombre` | Reemplazo de *f-string* por consulta parametrizada (`WHERE nombre LIKE ?`) | `[hash]` | #__ |
| VULN-02 | Cross-Site Scripting Reflejado (`CWE-79`) | Revisión manual / OWASP ZAP | `app.py:142` (`/hello`) | Alto: ejecución de JavaScript arbitrario en el navegador vía parámetro `name` | Escapado de caracteres HTML con `markupsafe.escape()` | `[hash]` | #__ |
| VULN-03 | Credenciales duras y hash débil MD5 (`CWE-798`, `CWE-327`) | Bandit (`B105`, `B324`) | `app.py:18-20, 210` (`/login`) | Alto: compromiso de `SECRET_KEY` y contraseña administrativa crackeable por colisión MD5 | Externalización a variables de entorno (`os.environ.get`), hash PBKDF2 (`werkzeug.security`) y `hmac.compare_digest` | `[hash]` | #__ |
| VULN-04 | Modo Debug activo en producción (`CWE-489`) | Bandit (`B201`) | `app.py:230` (`__main__`) | Alto: exposición del depurador interactivo Werkzeug (posible RCE ante excepciones) | Desactivación por defecto (`debug=False`), configurable solo vía `FLASK_DEBUG` | `[hash]` | #__ |
| VULN-05 | Exposición de información en errores (`CWE-209`) | Revisión manual | `app.py:195-196` (`/alumnos/buscar`) | Medio: filtración de esquema y consulta SQL exacta en respuestas HTTP 500 | Registro interno con `app.logger.exception()` y retorno de mensaje genérico al cliente | `[hash]` | #__ |
| VULN-06 | Ausencia de cabeceras de seguridad HTTP (`CWE-693`) | OWASP ZAP / Revisión manual | `app.py:120-129` (`after_request`) | Medio: exposición a *clickjacking*, *MIME-sniffing* e inyección de recursos | Inyección de `X-Content-Type-Options`, `X-Frame-Options` y `Content-Security-Policy` | `[hash]` | #__ |

## 3. Resultados de OWASP ZAP

| Alerta de ZAP | Riesgo | URL de ejemplo | VULN relacionada | Cómo se mitiga | ¿Mitigada? Build |
|---|---|---|---|---|---|
| Missing Anti-clickjacking Header | Medium | `http://localhost:5000/` | `VULN-06` | Cabecera `X-Frame-Options: DENY` y `CSP frame-ancestors` | Sí - #__ |
| X-Content-Type-Options Header Missing | Low | `http://localhost:5000/alumnos` | `VULN-06` | Cabecera `X-Content-Type-Options: nosniff` en `@app.after_request` | Sí - #__ |
| Content Security Policy (CSP) Header Not Set | Medium | `http://localhost:5000/` | `VULN-06` | Cabecera `Content-Security-Policy: default-src 'self'` | Sí - #__ |
| Cross Site Scripting (Reflected) | High | `http://localhost:5000/hello?name=<script>...` | `VULN-02` | Sanitización de entrada con `escape(name)` | Sí - #__ |
| SQL Injection | High | `http://localhost:5000/alumnos/buscar?nombre=' OR 1=1--` | `VULN-01` | Consulta parametrizada SQLite con marcadores `?` | Sí - #__ |

## 4. Monitoreo e incidentes

| Hora | Panel / métrica de Grafana | Qué se observó | Entorno | Clasificación | Acción de respuesta | Verificación |
|---|---|---|---|---|---|---|
| HH:MM | `peticiones_sospechosas_total` (`tipo="sqli"` / `"xss"`) | Incremento súbito de peticiones con cargas `' OR 1=1` y `<script>` hacia `/alumnos/buscar` y `/hello` | Producción | Intento de explotación (SQLi / XSS) | Se verifica que los parches `VULN-01` y `VULN-02` bloquean el impacto y se registra IP origen en logs | Respuesta controlada sin filtración de datos ni ejecución de scripts |
| HH:MM | `login_fallidos_total` | Múltiples intentos fallidos consecutivos contra `/login` (fuerza bruta) | Producción | Incidente de autenticación | Se valida robustez del hash PBKDF2 (`VULN-03`) y monitoreo de alertas en Grafana | Sin accesos no autorizados (`login_exitosos_total` estable) |

## 5. Gestión de dependencias

| Paquete | Versión inicial | Vulnerabilidades (IDs) | Versión final | Origen del cambio (PR de Dependabot / manual) | ¿Pruebas, pip-audit y ZAP siguen OK? Build |
|---|---|---|---|---|---|
| Flask | 2.2.2 | `CVE-2023-30861` (divulgación de cookie de sesión) | 3.0.3 (o >=2.3.2) | PR de Dependabot #__ | Sí - #__ |
| Werkzeug | 2.2.2 | `CVE-2023-25577`, `CVE-2023-46136`, `CVE-2024-34069` (DoS / RCE debugger) | 3.0.6 | PR de Dependabot #__ | Sí - #__ |
| Jinja2 | 3.1.2 | `CVE-2024-22195`, `CVE-2024-34064`, `CVE-2024-56326` (XSS / Sandbox) | 3.1.5 | PR de Dependabot #__ | Sí - #__ |
| gunicorn | 21.2.0 | `CVE-2024-1135`, `CVE-2024-6827` (HTTP Request Smuggling) | 23.0.0 | PR de Dependabot #__ | Sí - #__ |
| prometheus-client | 0.17.1 | Sin vulnerabilidades conocidas (`0` CVEs) | 0.21.0 (o 0.17.1) | PR de Dependabot #__ / Sin cambios | Sí - #__ |

## 6. Trazabilidad

| Actividad | Hallazgo | Build que lo detectó | Commit / PR que lo resolvió | Build que lo verificó | Versión en producción |
|---|---|---|---|---|---|
| Revisión de seguridad (Bandit) | `VULN-01` a `VULN-06` (SQLi, XSS, MD5, Debug, Headers) | #__ | `fix: mitiga VULN-01 a VULN-06` (`[hash]`) | #__ | `1.0.__+[hash]` |
| Prueba automatizada (OWASP ZAP) | Falta de cabeceras CSP/X-Frame-Options y vectores XSS/SQLi | #__ | `fix: mitiga VULN-01, VULN-02 y VULN-06` (`[hash]`) | #__ | `1.0.__+[hash]` |
| Gestión de dependencias (Dependabot) | CVEs en Flask 2.2.2, Werkzeug 2.2.2, Jinja2 3.1.2 y Gunicorn 21.2.0 | #__ | PR #__ (Dependabot) | #__ | `1.0.__+[hash]` |
