# Evidencias de ejecución del pipeline

Repositorio: https://github.com/luialacon/webgoat_PAI-4 — workflow `PAI-4 DevSecOps WebGoat (Mejorado)`.

## ejecucion-2026-11-25 (primera ejecución completa, run 45681585355)
Primer pipeline con las cinco herramientas. Importó a DefectDojo (ver capturas en el informe).
Limitaciones detectadas en el análisis y corregidas después:
- Gitleaks ignoró sus parámetros: no generó `gitleaks.json` ni analizó todo el historial.
- Dependency-Check escribió el informe en una carpeta que no se recogía (nunca llegó a DefectDojo).
- Gate B bloqueó con exit code 2 por avisos de ZAP, sin umbral explícito.

## ejecucion-2026-11-26 (pipeline corregido, run 45829247962)
| Fichero | Herramienta | Resultado |
|---|---|---|
| gitleaks.json | Gitleaks (secretos, 14 commits) | 24 detecciones (claves de ejemplo de las lecciones y tests) |
| semgrep-report.json | Semgrep (SAST) | 31 hallazgos de severidad ERROR |
| trivy-sca-report.json | Trivy fs (SCA, pom.xml) | 23 (1 crítica, 22 altas), todo xstream 1.4.5 |
| trivy-iac-report.json | Trivy config (Dockerfile) | 0 CRITICAL/HIGH |
| trivy-img-report.json | Trivy image | 23 (1 crítica, 22 altas) |
| zap-report.xml | OWASP ZAP baseline (DAST) | 14 alertas; 3 de riesgo medio bloquean (FAIL-NEW: 3, exit code 1) |
| pipeline-run-45829247962.log | Log completo | Gate A en aviso (verde), Gate B detenido por ZAP |

En esta ejecución el job de importación a DefectDojo se detuvo con un error explícito porque el
túnel temporal hacia la instancia local de DefectDojo no estaba activo (HTTP 000).

Ningún fichero contiene credenciales (comprobado). Las capturas de DefectDojo están en `capturas/`.
