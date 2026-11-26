# Triaje y priorización de hallazgos (ejecución 45833277119)

Criterio: WebGoat es una aplicación vulnerable a propósito. Los hallazgos en las lecciones
(`src/main/java/.../lessons`, `src/main/resources/lessons`) y en los tests son el material
didáctico y **no se corrigen** (ver `AGENTS.md`). Se triajan en DefectDojo como *False Positive*
(el hallazgo no es un defecto no intencionado) o *Risk Accepted* (se acepta de forma justificada).
Los hallazgos fuera de ese ámbito (CI/CD, Dockerfile_desktop, cabeceras HTTP) son reales y se priorizan.

| Herramienta | Hallazgo | Nº | Decisión DefectDojo | Justificación |
|---|---|---|---|---|
| Gitleaks | generic-api-key / jwt / private-key en lecciones, documentación y tests | 24 | False Positive | Claves y tokens de ejemplo de las lecciones JWT y criptografía; no son credenciales reales |
| Semgrep | detected-jwt-token | 13 | False Positive | JWT de ejemplo en documentación y tests |
| Semgrep | formatted/tainted-sql-string, XXE, ProcessBuilder, path traversal, SSRF en `lessons/` | 17 | Risk Accepted | Vulnerabilidades deliberadas de las lecciones |
| Semgrep | `run-shell-injection` en `.github/workflows/release.yml:24` | 1 | **Real, P3** | Workflow heredado del proyecto; se corrige pasando el valor por `env:` |
| Trivy SCA / imagen | xstream 1.4.5: CVE-2013-7285 (crítica) y 22 altas | 23 + 23 (duplicado entre SCA e imagen) | Risk Accepted | Componente vulnerable deliberado: lo usa `VulnerableComponentsLesson` para enseñar la explotación de CVE-2013-7285. Actualizarlo rompería la lección (`AGENTS.md`). Se vigila que no aparezcan otras dependencias vulnerables |
| Trivy IaC | DS-0002 `Dockerfile_desktop` ejecuta como root | 1 | **Real, P2** | Añadir `USER` no privilegiado; imagen solo de escritorio de formación |
| ZAP | CSP (10038), anti-clickjacking (10020), anti-CSRF (10202) | 3 | **Real, P2 (bloquea Gate B)** | Cabeceras de seguridad ausentes en la aplicación |
| ZAP | Cookie SameSite, Permissions-Policy, X-Content-Type-Options, COEP/COOP/CORP | 6 | Real, P4 (aviso) | Endurecimiento de cabeceras |
| ZAP | 10031, 10111, 10112 | 3 | False Positive / IGNORE en `.zap/rules.tsv` | Reglas pasivas informativas sin evidencia de ataque |

Prioridad por SLA de DefectDojo (crítica 7 días, alta 30, media 90, baja 120).
