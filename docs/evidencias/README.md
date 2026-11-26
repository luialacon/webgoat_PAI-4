# Evidencias de la ejecución del 25-11-2026

Ejecución de GitHub Actions nº 45681585355 (workflow "PAI-4 DevSecOps WebGoat (Mejorado)").

| Fichero | Contenido |
|---|---|
| pipeline-run-45681585355.log | Log completo de los tres jobs |
| semgrep-report.json | SAST: 31 hallazgos de severidad alta |
| trivy-img-report.json | Imagen Docker: 23 hallazgos (1 crítica, 22 altas) |
| trivy-iac-report.json | IaC (Dockerfile): 0 hallazgos CRITICAL/HIGH |
| zap-report.xml | DAST: 14 alertas (3 medias, 6 bajas, 5 informativas) |
| gitleaks-results.sarif | Gitleaks de esa ejecución: solo analizó el último commit (fallo corregido después) |

**Limitaciones de esta ejecución (corregidas en el pipeline actual, pendientes de volver a ejecutar):**
- No hay informe de Dependency-Check: se escribió en una carpeta que el pipeline no recogía.
- Gitleaks ignoró sus parámetros: no generó `gitleaks.json` ni analizó todo el historial.
- Gate B bloqueó con ZAP (exit code 2) por avisos; ahora el umbral está definido en `.zap/rules.tsv`.

Las capturas de DefectDojo y de la ejecución fallida por quality gate deben añadirse a esta carpeta.
Los ficheros no contienen credenciales (comprobado).
