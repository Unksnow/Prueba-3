# Documentación y Trazabilidad Integral del Pipeline DevSecOps

## 1. Arquitectura y Flujo del Pipeline (CI/CD/Sec)
El flujo de integración, seguridad y despliegue continuo del **Sistema de Alumnos** está orquestado mediante **Jenkins**, **GitHub** (`Prueba-3`, rama `main`) y contenedores **Docker**, integrando controles automatizados en cada fase:

| Orden | Etapa del Pipeline | Herramienta / Archivo | Actividad Ejecutada | Criterio de Aprobación / Trazabilidad |
|---|---|---|---|---|
| 1 | **Control de Versiones (SCM)** | Git / GitHub (`main`) | Obtención del código fuente, historial de commits y validación de manifiestos | Commit verificado en repositorio `Prueba-3` |
| 2 | **Gestión de Dependencias** | Dependabot (`.github/dependabot.yml`) | Detección semanal automatizada de paquetes desactualizados o vulnerables en `pip` | 6 *Pull Requests* (`chore(deps)`) revisados y fusionados |
| 3 | **Pruebas Automatizadas** | `pytest` (`requirements-dev.txt`) | Ejecución de suite de pruebas unitarias y funcionales sobre rutas, CRUD y autenticación | 100 % de tests aprobados (`passed`) sin regresiones |
| 4 | **Revisión de Seguridad Estática (SAST)** | `Bandit` | Análisis estático del código fuente Python en busca de malas prácticas e inyecciones | Sin hallazgos de severidad alta/crítica tras mitigación |
| 5 | **Auditoría de Dependencias (SCA)** | `pip-audit` | Verificación de `requirements.txt` y `requirements-dev.txt` contra bases CVE (PyPA/OSV) | Salida `No known vulnerabilities found` |
| 6 | **Despliegue Staging** | Docker / Gunicorn (`jenkins:5001`) | Construcción y despliegue automático en entorno de preproducción | Servicio activo (`up{job="staging"} = 1`) |
| 7 | **Despliegue Producción** | Docker / Gunicorn (`jenkins:5000`) | Promoción controlada del artefacto validado al entorno productivo | Servicio activo (`up{job="produccion"} = 1`) |
| 8 | **Observabilidad y Respuesta** | Prometheus + cAdvisor + Grafana | Supervisión en tiempo real (`up`, `http_requests_created`, `login_fallidos`) y alertas (`10s`) | Reglas activas en carpeta `Seguridad` y bitácora de incidentes |

---

## 2. Trazabilidad de Revisiones de Seguridad Aplicadas
- **Endurecimiento de código fuente (Remediación SAST):** Sustitución de almacenamiento inseguro de contraseñas por hashes criptográficos (`werkzeug.security`), configuración de cookies de sesión con bandera `HttpOnly`, desactivación del modo `debug` en entornos desplegados y validación estricta de parámetros en rutas Flask.
- **Auditoría de componentes de terceros (SCA):** Actualización integral de `Werkzeug` (`3.1.9`), `Flask` (`3.1.3`), `gunicorn` (`27.0.1`), `prometheus-client` (`0.26.0`), `Jinja2` (`3.1.6`) y `pytest` (`9.2.0`), validada con `pip-audit` y GitHub Security (`0` alertas abiertas).
- **Detección continua de incidentes:** Regla de alerta configurada en Grafana (`Seguridad / Monitoreo-10s`) evaluando disponibilidad cada `10s`, complementada con paneles de tráfico HTTP por código de estado (`200`, `404`) y contador de `login_fallidos`.

---

## 3. Índice de Documentación Trazable en el Repositorio
- `Jenkinsfile`: Definición declarativa de las etapas de construcción, pruebas, auditoría y despliegue.
- `.github/dependabot.yml`: Política automatizada de actualización y seguridad de dependencias Python.
- `docs/REGISTRO_INCIDENTES_MONITORIZACION.md`: Bitácora de actividades de monitoreo e incidentes (`INC-01` a `INC-03`).
- `docs/GESTION_DEPENDENCIAS.md`: Detalle de actualizaciones aplicadas y controles contra nuevas vulnerabilidades.
- `docs/TRAZABILIDAD_PIPELINE_DEVSECOPS.md`: Consolidado general de trazabilidad del ciclo DevSecOps.
