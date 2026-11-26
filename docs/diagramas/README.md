# Diagramas del pipeline DevSecOps (PAI-4)

Se visualizan directamente en GitHub (Mermaid). Corresponden a
`.github/workflows/devsecops-webgoat-improved.yml`.

## 1.a Diagrama del pipeline

```mermaid
flowchart LR
    DEV[Desarrollador<br/>push / pull request a main] --> J1

    subgraph J1["Job 1 · Build + análisis estático<br/>GATE A — MODO AVISO (continue-on-error)"]
        direction TB
        B1[Compilar con Maven<br/>Java 21] --> B2[Construir imagen Docker<br/>webgoat-local:pai4]
        B2 --> S1["Gitleaks · Secretos<br/>umbral: cualquier secreto<br/>→ gitleaks.json"]
        S1 --> S2["Semgrep · SAST<br/>umbral: severidad ERROR<br/>→ semgrep-report.json"]
        S2 --> S3["Trivy fs · SCA (pom.xml)<br/>umbral: CRITICAL, HIGH<br/>→ trivy-sca-report.json"]
        S3 --> S4["Trivy config · IaC (Dockerfile)<br/>umbral: CRITICAL, HIGH<br/>→ trivy-iac-report.json"]
    end

    J1 -- imagen Docker<br/>como artefacto --> J2

    subgraph J2["Job 2 · Imagen + staging + DAST<br/>GATE B — BLOQUEANTE"]
        direction TB
        T1["Trivy image · imagen Docker<br/>MODO AVISO · CRITICAL, HIGH<br/>→ trivy-img-report.json"] --> T2[Arrancar WebGoat en contenedor<br/>localhost:8080]
        T2 --> T3["OWASP ZAP baseline · DAST<br/>BLOQUEA si hay alertas de riesgo MEDIO o superior<br/>(reglas FAIL en .zap/rules.tsv)<br/>→ zap-report.xml"]
    end

    J2 -- "siempre (if: always)" --> J3
    J1 -- "siempre (if: always)" --> J3

    subgraph J3["Job 3 · Gestión de vulnerabilidades"]
        D1["Importar a DefectDojo vía API REST<br/>/api/v2/import-scan/<br/>comprueba conexión y falla si no se suben los 6 informes"]
    end

    J2 -- "si Gate B pasa" --> PROD[Despliegue a producción<br/>fuera del alcance del PAI]
    J3 --> DD[(DefectDojo<br/>triaje · deduplicación · priorización)]
```

| Etapa | Herramienta | Tipo | Informe | Gate | Umbral |
|---|---|---|---|---|---|
| Job 1 | Gitleaks | Secretos | gitleaks.json | A · aviso | cualquier secreto |
| Job 1 | Semgrep | SAST | semgrep-report.json | A · aviso | severidad ERROR |
| Job 1 | Trivy fs (pom.xml) | SCA | trivy-sca-report.json | A · aviso | CRITICAL / HIGH |
| Job 1 | Trivy config | IaC | trivy-iac-report.json | A · aviso | CRITICAL / HIGH |
| Job 2 | Trivy image | Imagen | trivy-img-report.json | aviso | CRITICAL / HIGH |
| Job 2 | OWASP ZAP | DAST | zap-report.xml | **B · bloqueante** | riesgo medio o superior |
| Job 3 | DefectDojo | Gestión | — | falla si faltan informes | 6 informes |

## 1.b Diagrama de flujo ante una vulnerabilidad

```mermaid
flowchart TD
    A[Una herramienta detecta un hallazgo] --> B[El informe se guarda como artefacto<br/>del pipeline]
    B --> C{¿Supera el umbral<br/>de esa herramienta?}
    C -- No --> R1[Se registra en DefectDojo<br/>severidad baja / informativa]
    C -- Sí --> D{¿La etapa es<br/>bloqueante?}
    D -- "No · Gate A / Trivy image<br/>(modo aviso)" --> W[El pipeline continúa<br/>Aviso en el log y en el resumen del job]
    D -- "Sí · Gate B (ZAP)" --> X[El pipeline se DETIENE<br/>job en rojo · GitHub avisa al autor del commit]
    W --> R2
    X --> R2[El informe se importa igualmente a DefectDojo<br/>job 3 con if: always]
    R1 --> T
    R2 --> T[Triaje en DefectDojo<br/>deduplicación y priorización por severidad / SLA]
    T --> E{¿Es un falso positivo<br/>o riesgo aceptado?}
    E -- Sí --> FP[Se marca False Positive / Risk Accepted<br/>con justificación escrita<br/>y se añade IGNORE en .zap/rules.tsv<br/>o suppression en la herramienta]
    E -- No --> F[Se asigna al responsable<br/>con fecha límite según SLA]
    F --> G[Corrección en el código<br/>o actualización de la dependencia]
    G --> H[Nuevo commit → el pipeline se vuelve a ejecutar]
    H --> I{¿La herramienta ya<br/>no lo detecta?}
    I -- Sí --> M[Hallazgo Mitigated / Closed en DefectDojo]
    I -- No --> F
```
