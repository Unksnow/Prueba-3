# Registro de Monitorización y Respuesta a Incidentes de Seguridad

## 1. Actividades de Monitorización Ejecutadas
- **Despliegue de pila de observabilidad:** Configuración de Prometheus (`scrape_interval: 15s`), cAdvisor y Grafana mediante contenedores Docker para supervisar los entornos `produccion` (`jenkins:5000`) y `staging` (`jenkins:5001`).
- **Supervisión de disponibilidad (`up`):** Regla de alerta configurada en Grafana con evaluación cada `10s` para detectar caídas de servicio (`up < 1`).
- **Análisis de tráfico HTTP (`http_requests_created`):** Monitoreo en ventana de tiempo real de 5 minutos desglosado por `endpoint` (`/`, `/alumnos`, `/alumnos/<int:alumno_id>`, `/alumnos/buscar`), método y código de estado (`200`, `404`).
- **Control de autenticación (`login_fallidos`):** Seguimiento continuo del contador de intentos fallidos de inicio de sesión en ambos entornos.
- **Métricas de infraestructura (cAdvisor):** Control de consumo de CPU, memoria RAM y tráfico de red de los contenedores activos.

## 2. Incidentes Detectados y Acciones de Respuesta

| ID | Fecha / Entorno | Incidente / Anomalía Detectada | Evidencia en Métricas | Acción de Respuesta Aplicada | Estado |
|---|---|---|---|---|---|
| INC-01 | `produccion` (`jenkins:5000`) | Indisponibilidad del servicio web en producción | `up{instance="jenkins:5000", job="produccion"} = 0` (Alerta `Firing` en Grafana) | Inspección de contenedores y logs del servicio, reinicio controlado de la instancia en puerto `5000` y validación de retorno a `up = 1` (`Normal`). | Resuelto |
| INC-02 | `staging` (`jenkins:5001`) | Solicitudes a recursos inexistentes (posible enumeración de IDs) | `http_requests_created` con `status="404"` en `/alumnos/<int:alumno_id>` | Validación de tipado estricto en rutas Flask, monitoreo de tasa de errores 4xx y planificación de *rate limiting* (`Flask-Limiter`). | Mitigado / En seguimiento |
| INC-03 | `produccion` y `staging` | Supervisión preventiva de intentos de acceso no autorizado | Serie temporal `login_fallidos` monitoreada en Grafana | Verificación de hashes seguros (`werkzeug.security`), cookies `HttpOnly` y alerta temprana ante incrementos súbitos de fallos de login. | Controlado |
