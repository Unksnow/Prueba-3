# Documentación del Proceso de Gestión de Dependencias

## 1. Flujo de Gestión Automatizada (DevSecOps)
Para evitar el uso de librerías obsoletas o con vulnerabilidades conocidas (CVE), se implementó un flujo continuo compuesto por:
1. **Detección y monitoreo:** Activación de *Dependency graph*, *Dependabot alerts* y *Dependabot security/version updates* en el repositorio de GitHub (`Prueba-3`).
2. **Automatización mediante manifiesto:** Configuración de `.github/dependabot.yml` (`version: 2`) para auditar semanalmente el ecosistema `pip` y generar *Pull Requests* aislados por cada paquete (`chore(deps)`).
3. **Integración en CI/CD:** Validación automática de dependencias en el `Jenkinsfile` mediante `pip-audit` antes de promover cambios a los entornos `staging` y `produccion`.

## 2. Actualizaciones de Dependencias Realizadas
Se aplicaron las actualizaciones propuestas por `dependabot[bot]` sobre `requirements-dev.txt` y se sincronizó `requirements.txt`:

| Paquete | Versión Anterior | Versión Actualizada | Archivo(s) | Objetivo de la Actualización |
|---|---|---|---|---|
| `Werkzeug` | `2.2.2` | `3.1.9` | `requirements.txt`, `requirements-dev.txt` | Mitigación de vulnerabilidades en servidor WSGI y manejo de rutas/archivos |
| `Flask` | `2.2.2` | `3.1.3` | `requirements.txt`, `requirements-dev.txt` | Actualización del framework web principal a rama estable soportada |
| `gunicorn` | `21.2.0` | `27.0.1` | `requirements.txt`, `requirements-dev.txt` | Corrección de fallos de seguridad en el servidor HTTP de producción |
| `prometheus-client` | `0.17.1` | `0.26.0` | `requirements.txt`, `requirements-dev.txt` | Actualización del exportador de métricas de observabilidad |
| `Jinja2` | `3.1.2` | `3.1.6` | `requirements.txt`, `requirements-dev.txt` | Parches de seguridad en el motor de plantillas HTML (prevención XSS/SSTI) |
| `pytest` | `8.3.4` | `9.2.0` | `requirements-dev.txt` | Actualización del framework de pruebas automatizadas |

## 3. Controles para Asegurar Ausencia de Nuevas Vulnerabilidades y Regresiones
Para garantizar que las actualizaciones no introdujeran nuevas brechas de seguridad ni incompatibilidades (especialmente tras cambios de versión mayor en `Flask` y `Werkzeug`), se aplicaron cuatro controles de verificación:
- **Fusión controlada por Pull Request:** Cada dependencia se actualizó y revisó de forma individual mediante los PR generados por Dependabot, manteniendo trazabilidad en el historial de Git.
- **Auditoría contra bases de vulnerabilidades (`pip-audit`):** Ejecución de `python -m pip_audit -r requirements.txt` y `python -m pip_audit -r requirements-dev.txt`, verificando contra las bases de datos PyPA/OSV el resultado `No known vulnerabilities found`.
- **Verificación de alertas en GitHub Security:** Comprobación de que el panel *Security -> Dependabot* y *Dependency graph* no reportan alertas activas tras la fusión.
- **Pruebas de regresión funcional (`pytest`):** Ejecución de `python -m pytest` para validar que las rutas, autenticación y lógica del sistema de alumnos operan correctamente con las nuevas versiones.
