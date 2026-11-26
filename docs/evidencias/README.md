# Evidencias de ejecución del pipeline

Repositorio: https://github.com/luialacon/webgoat_PAI-4 — workflow `PAI-4 DevSecOps WebGoat (Mejorado)`.

## ejecucion-2026-11-25 (primera ejecución completa, run 45681585355)
https://github.com/luialacon/webgoat_PAI-4/actions/runs/45681585355
Primer pipeline con cinco herramientas. Importó a DefectDojo (capturas en `capturas/`).
Defectos detectados en el análisis y corregidos después:
- Gitleaks ignoró sus parámetros: no generó `gitleaks.json` ni analizó todo el historial.
- Dependency-Check escribió el informe en una carpeta que no se recogía (nunca llegó a DefectDojo).
- Gate B bloqueó con exit code 2 por avisos de ZAP, sin umbral explícito.

## ejecucion-2026-11-26-final (pipeline corregido, run 45833277119)
https://github.com/luialacon/webgoat_PAI-4/actions/runs/45833277119

| Fichero | Herramienta | Resultado |
|---|---|---|
| gitleaks.json | Gitleaks (secretos, historial completo) | 24 detecciones (claves de ejemplo de lecciones y tests) |
| semgrep-report.json | Semgrep (SAST) | 31 hallazgos de severidad ERROR |
| trivy-sca-report.json | Trivy fs (SCA, pom.xml) | 23 (1 crítica, 22 altas), todas en xstream 1.4.5 |
| trivy-iac-report.json | Trivy config (Dockerfile y Dockerfile_desktop) | 1 alta: DS-0002, `Dockerfile_desktop` se ejecuta como root |
| trivy-img-report.json | Trivy image | 23 (1 crítica, 22 altas) |
| zap-report.xml | OWASP ZAP baseline (DAST) | 14 alertas; 3 de riesgo medio bloquean (FAIL-NEW: 3, exit code 1) |
| pipeline-run-45833277119.log | Log completo | Gate A en aviso, Gate B detenido por ZAP |

El job de importación a DefectDojo se detiene con un error explícito cuando el túnel temporal
hacia la instancia local no está activo (HTTP 000).

Ningún fichero contiene credenciales (comprobado). Triaje: `../triaje-vulnerabilidades.md`.
