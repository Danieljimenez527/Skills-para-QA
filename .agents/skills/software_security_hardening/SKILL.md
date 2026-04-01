---
name: Software Security Hardening & Supply Chain Defense
description: >
  Skill de nivel Arquitecto de Seguridad / CISO para prevenir vulnerabilidades críticas en el ciclo
  de desarrollo de software. Cubre seguridad de cadena de suministro (como el ataque a axios NPM del
  31-Mar-2026), OWASP Top 10:2025, DevSecOps Shift-Left, gestión segura de dependencias, SAST/DAST/SCA,
  hardening de CI/CD y cumplimiento con SLSA/SBOM/OpenSSF. Opera como capa de defensa activa en todo
  el ciclo de vida del software (SDLC), complementando la skill qa_devsecops_security.
---

# 🛡️ Software Security Hardening & Supply Chain Defense

## Contexto: ¿Por qué existe esta Skill?

El **31 de marzo de 2026**, el paquete **`axios`** (>100M descargas/semana en npm) fue comprometido
mediante un ataque de **Account Takeover** al mantenedor principal. Los atacantes publicaron versiones
maliciosas (`axios@1.14.1` y `axios@0.30.4`) que inyectaban la dependencia `plain-crypto-js@4.2.1`,
la cual descargaba un **Remote Access Trojan (RAT)** multiplataforma vía un script `postinstall`.
El ataque estuvo activo **~3 horas** y fue atribuido al actor norcoreano **UNC1069** (Google TI).

Este incidente ilustra una amenaza sistémica: **las dependencias open source son vectores de ataque
de primer nivel**. Esta skill convierte esa lección en controles accionables y permanentes.

---

## NORMAS DE ESTRICTO CUMPLIMIENTO (Instrucciones para la IA)

Cuando asistas en generación de código, revisión de arquitectura, configuración de pipelines o
evaluación de dependencias, **DEBES** aplicar sin excepción los siguientes controles:

---

## 1. 🔗 Defensa de Cadena de Suministro (Supply Chain Security)

### 1.1 Gestión de Dependencias NPM — Lecciones del caso Axios

> **REGLA CRÍTICA:** Nunca uses `npm install` en entornos CI/CD. Siempre usa `npm ci`.

```bash
# ❌ PROHIBIDO EN CI/CD — instala versiones según rangos semver (vulnerable a ataques)
npm install

# ✅ OBLIGATORIO EN CI/CD — respeta el lockfile exacto, falla si hay discrepancias
npm ci --ignore-scripts   # --ignore-scripts deshabilita postinstall maliciosos

# ✅ Verificación de integridad de dependencias
npm audit --audit-level=high
npm audit --audit-level=critical --json > audit-report.json
```

**Controles obligatorios para dependencias NPM:**
- [ ] Verificar mantenedores del paquete en [npmjs.com](https://www.npmjs.com) antes de instalar
- [ ] Revisar el `package-lock.json` en cada Pull Request (especialmente cambios de sub-dependencias)
- [ ] Habilitar **npm provenance** para verificar que el paquete fue publicado desde un pipeline oficial (GitHub Actions OIDC), no manualmente con un token de usuario
- [ ] Usar **`--ignore-scripts`** por defecto y solo habilitar scripts confiables explícitamente
- [ ] Configurar alertas de Dependabot o Snyk para notificaciones en tiempo real de nuevas CVEs

### 1.2 Detección de Typosquatting y Paquetes Fantasma

La IA puede **alucinar nombres de paquetes** que no existen o que han sido registrados por atacantes.

```bash
# Antes de instalar CUALQUIER paquete recomendado por IA:
# 1. Verifica que existe en npmjs.com
# 2. Compara el nombre exacto (ej: "axios" vs "axois", "node-axios", "axioss")
# 3. Revisa fecha de creación, mantenedores y número de descargas
# 4. Busca el repositorio GitHub oficial vinculado al paquete
npm info <nombre-paquete> --json | Select-String "maintainers|repository|created"
```

**Señales de alerta en un paquete NPM:**
- Publicado hace menos de 30 días con muchas versiones
- Sin repositorio GitHub vinculado o repositorio vacío
- Nombre muy similar a una librería popular (typosquatting)
- Scripts `postinstall` o `preinstall` que descarguen recursos externos
- Dependencias transitivas desconocidas añadidas de forma reciente

### 1.3 SBOM — Software Bill of Materials

Genera y mantén un catálogo de todas las dependencias del proyecto:

```bash
# Generar SBOM en formato CycloneDX (estándar internacional)
npx @cyclonedx/cyclonedx-npm --output-file sbom.json

# Generar SBOM en formato SPDX
npx spdx-sbom-generator

# Para proyectos Java/Maven
mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom
```

**El SBOM debe:**
- Actualizarse automáticamente en cada build del CI/CD
- Incluirse como artefacto en cada release
- Ser escaneado contra bases de datos de CVEs (OSV, NVD) en tiempo real

---

## 2. 🚀 DevSecOps Shift-Left — Seguridad desde el Commit

### 2.1 Pre-commit Hooks (Primera línea de defensa)

```bash
# Instalar pre-commit localmente
pip install pre-commit

# Configuración .pre-commit-config.yaml
```

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.4
    hooks:
      - id: gitleaks          # Detecta secretos hardcodeados ANTES del commit

  - repo: https://github.com/trufflesecurity/trufflehog
    rev: v3.70.0
    hooks:
      - id: trufflehog        # Escaneo profundo de secretos y tokens

  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.7
    hooks:
      - id: bandit            # SAST para Python (si aplica)
        args: ["-r", ".", "-ll"]

  - repo: https://github.com/returntocorp/semgrep
    rev: v1.68.0
    hooks:
      - id: semgrep           # SAST agnóstico de lenguaje
        args: ["--config=auto", "--error"]
```

### 2.2 Pipeline CI/CD Seguro (GitHub Actions Template Hardened)

```yaml
# .github/workflows/secure-ci.yml
name: Secure CI Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

# 🔐 Principio de Mínimo Privilegio — siempre declarar permisos explícitamente
permissions:
  contents: read
  security-events: write   # Solo para subir resultados SARIF
  pull-requests: read

jobs:
  security-scan:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout (con historial limitado)
        uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae1  # SHA fijo, NO tag mutable
        with:
          fetch-depth: 1

      - name: Setup Node.js
        uses: actions/setup-node@60edb5dd545a775178f52524783378180af0d1f8  # SHA fijo
        with:
          node-version: '20'
          cache: 'npm'

      # ✅ npm ci + deshabilitar scripts postinstall
      - name: Install dependencies (locked)
        run: npm ci --ignore-scripts

      # 🔍 SCA — Análisis de Composición de Software
      - name: Dependency vulnerability audit
        run: npm audit --audit-level=high

      # 🔍 SAST — Análisis Estático con CodeQL
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript, typescript

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3

      # 🔑 Secret Scanning con Gitleaks
      - name: Secret scanning
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      # 📋 Generar SBOM en cada build
      - name: Generate SBOM
        run: npx @cyclonedx/cyclonedx-npm --output-file sbom.json
        
      - name: Upload SBOM as artifact
        uses: actions/upload-artifact@v4
        with:
          name: sbom-${{ github.sha }}
          path: sbom.json
          retention-days: 90
```

---

## 3. 🔎 OWASP Top 10:2025 — Controles por Categoría

| # | Vulnerabilidad | Control Obligatorio |
|---|----------------|---------------------|
| A01 | **Broken Access Control** | Implementar RBAC/ABAC; test de autorización en cada endpoint |
| A02 | **Security Misconfiguration** | IaC scanning con Checkov; hardening de headers HTTP |
| A03 | **Supply Chain Failures** ⭐ | SBOM + SCA + `npm ci --ignore-scripts` + npm provenance |
| A04 | **Cryptographic Failures** | TLS 1.3 mínimo; AES-256; nunca MD5/SHA1 para datos sensibles |
| A05 | **Injection** | Queries parametrizadas; sanitización de inputs; CSP headers |
| A06 | **Insecure Design** | Threat Modeling en fase de diseño; STRIDE/DREAD |
| A07 | **Authentication Failures** | MFA obligatorio; OAuth 2.0 + PKCE; rotación de tokens |
| A08 | **Software & Data Integrity** | Firma digital de artefactos (Sigstore/Cosign); verificar hashes |
| A09 | **Security Logging Failures** | Logs inmutables; alertas SIEM; no loggear datos sensibles |
| A10 | **Exceptional Conditions** | Fail-closed por defecto; manejo de errores sin exponer stack traces |

---

## 4. 🔑 Gestión Segura de Secretos

```bash
# ❌ PROHIBIDO — Secrets hardcodeados
const API_KEY = "sk-prod-a1b2c3d4e5f6";
DB_PASSWORD=SuperSecreta123

# ✅ CORRECTO — Variables de entorno inyectadas
const API_KEY = process.env.API_KEY;
// En CI/CD: configurar como GitHub Secrets o AWS Secrets Manager

# ✅ Para entornos locales — usar .env con .gitignore
# Archivo .gitignore DEBE incluir:
.env
.env.local
.env.production
*.pem
*.key
```

**Herramientas de gestión de secretos recomendadas:**

| Entorno | Herramienta |
|---------|-------------|
| Local Dev | `.env` + `dotenv` (nunca comitear) |
| CI/CD | GitHub Actions Secrets / GitLab CI Variables |
| Producción | HashiCorp Vault / AWS Secrets Manager / Azure Key Vault |
| Detección | Gitleaks, TruffleHog, GitHub Secret Scanning |

---

## 5. 🏗️ SLSA Framework — Niveles de Madurez de la Cadena de Suministro

**SLSA (Supply-chain Levels for Software Artifacts)** es el framework internacional para verificar
la integridad del proceso de construcción de software:

| Nivel | Requisito | Beneficio |
|-------|-----------|-----------|
| **SLSA 1** | Pipeline automatizado (sin builds manuales) | Trazabilidad básica |
| **SLSA 2** | Build en servicio controlado + provenance firmado | Previene tampering post-build |
| **SLSA 3** | Build reproducible + entorno efímero | Previene insider threats en el CI |
| **SLSA 4** | Revisión de código dual + hermetic builds | Nivel máximo; estándar para releases críticos |

**Meta mínima recomendada para proyectos de producción: SLSA 2**

```bash
# Verificar provenance de un artefacto npm con Sigstore
npx @sigstore/cli verify --bundle provenance.bundle dist/app.js

# Verificar provenance de imagen Docker
cosign verify --certificate-identity-regexp="github" \
  --certificate-oidc-issuer="https://token.actions.githubusercontent.com" \
  ghcr.io/mi-org/mi-app:v1.0.0
```

---

## 6. 🧰 Herramientas Recomendadas (Ecosistema 2026)

### Análisis Estático (SAST)
```bash
# CodeQL — gratuito para repos públicos, integrado en GitHub
gh extension install github/gh-codeql

# Semgrep — agnóstico de lenguaje, >3000 reglas
semgrep --config=p/owasp-top-ten --config=p/supply-chain .

# SonarQube Community Edition — self-hosted
docker run -d --name sonarqube -p 9000:9000 sonarqube:community
```

### Análisis de Composición (SCA)
```bash
# Trivy — especialista en containers y filesystems
trivy fs --scanners vuln,secret,license .
trivy image --severity HIGH,CRITICAL mi-app:latest

# Snyk — integración nativa con IDEs y CI
npx snyk test --severity-threshold=high
npx snyk monitor  # Monitoreo continuo en la nube de Snyk

# Grype — alternativa open source a Snyk
grype dir:.
```

### Gestión de Secretos y Rotación
```bash
# Detección de secretos en historial Git completo
trufflehog git file://. --only-verified

# Rotación de secretos comprometidos — URGENTE si hubo exposición
gh secret set API_KEY --body "$(openssl rand -hex 32)"
```

---

## 7. 🚨 Plan de Respuesta a Incidentes de Supply Chain

**Si detectas que instalaste una versión comprometida (ej. caso Axios):**

```
NIVEL DE RESPUESTA: CRÍTICO — Ejecutar en los próximos 30 minutos

PASO 1 - CONTENCIÓN (0-5 min):
  □ Aislar la máquina o runner de CI/CD afectado de la red
  □ Revocar TODOS los tokens, API keys y credenciales del entorno afectado
  □ Notificar al equipo de seguridad y líder técnico inmediatamente

PASO 2 - EVALUACIÓN (5-15 min):
  □ Verificar qué versión fue instalada:  npm list axios
  □ Revisar el package-lock.json en control de versiones
  □ Identificar la ventana de tiempo de exposición
  □ Determinar qué secretos/datos estuvieron expuestos

PASO 3 - ERRADICACIÓN (15-30 min):
  □ Actualizar a la versión segura: npm install axios@latest
  □ Regenerar package-lock.json limpio
  □ Reconstruir el entorno de CI/CD desde cero (runner efímero)
  □ Rotar TODOS los secretos que estuvieron en el entorno

PASO 4 - RECUPERACIÓN Y FORTALECIMIENTO:
  □ Activar monitoreo de red para detectar conexiones salientes anómalas
  □ Auditar otros proyectos que puedan haber instalado el paquete
  □ Reportar el paquete malicioso en:
      - npm: https://www.npmjs.com/support (Report a package)
      - CISA: https://www.cisa.gov/report
  □ Documentar el incidente con timeline y acciones tomadas
  □ Revisar y fortalecer controles preventivos (esta skill)
```

---

## 8. 📋 Checklist de Auditoría Rápida

Ejecuta este checklist antes de cada release o cuando incorpores nuevas dependencias:

### Dependencias y Supply Chain
- [ ] `npm ci --ignore-scripts` configurado en el pipeline de CI/CD
- [ ] `npm audit` sin vulnerabilidades HIGH/CRITICAL
- [ ] `package-lock.json` commiteado y revisado en PRs
- [ ] npm provenance habilitado para tus propias publicaciones
- [ ] Verificación de mantenedores activos en dependencias críticas
- [ ] SBOM generado y archivado para el release actual

### Secretos y Credenciales
- [ ] Sin hardcoded secrets en el código (`grep -rE "(password|secret|api_key)\s*=" .`)
- [ ] `.env` y archivos de configuración en `.gitignore`
- [ ] Secretos en GitHub Secrets / Vault, no en variables de texto plano
- [ ] Rotación de tokens de acceso cada 90 días máximo
- [ ] Secret scanning habilitado en el repositorio

### Pipeline CI/CD
- [ ] Actions/Steps fijadas a SHA de commit específico (no tags mutables)
- [ ] Permisos mínimos declarados en cada job (`permissions: contents: read`)
- [ ] Environments con aprobación manual para deploys a producción
- [ ] Runners efímeros (no persistentes) para builds de producción
- [ ] Logs del CI/CD no exponen valores de secretos

### Código y Arquitectura
- [ ] SAST integrado (CodeQL, Semgrep o SonarQube) sin hallazgos críticos
- [ ] DAST ejecutado contra ambiente de staging antes de cada release
- [ ] Inputs validados y sanitizados (prevención de Injection)
- [ ] Headers de seguridad HTTP configurados (CSP, HSTS, X-Frame-Options)
- [ ] Dependencias actualizadas en los últimos 6 meses

---

## 9. 🔗 Referencias y Frameworks Globales

| Recurso | URL | Relevancia |
|---------|-----|------------|
| **OWASP Top 10:2025** | https://owasp.org/Top10 | Vulnerabilidades web más críticas |
| **OWASP ASVS** | https://owasp.org/ASVS | Estándar de verificación de seguridad aplicaciones |
| **SLSA Framework** | https://slsa.dev | Niveles de seguridad de la cadena de suministro |
| **OpenSSF Scorecard** | https://scorecard.dev | Puntuación de seguridad para repositorios OSS |
| **OpenSSF Best Practices** | https://bestpractices.coreinfrastructure.org | Criterios de madurez de seguridad |
| **NIST SSDF** | https://csrc.nist.gov/projects/ssdf | Framework seguro de desarrollo de software |
| **CycloneDX** | https://cyclonedx.org | Estándar SBOM (BOM de dependencias) |
| **Sigstore** | https://sigstore.dev | Firma criptográfica de artefactos OSS |
| **Google OSV** | https://osv.dev | Base de datos de vulnerabilidades OSS |
| **CISA KEV** | https://cisa.gov/known-exploited-vulnerabilities | Vulnerabilidades explotadas activamente |

---

## 10. Cómo Invocar esta Skill

Esta skill opera de manera **proactiva y preventiva**. Invócala explícitamente con:

- *"Audita las dependencias de este `package.json` buscando riesgos de supply chain."*
- *"Revisa esta GitHub Action y aplica el hardening de mínimo privilegio y SHA pinning."*
- *"Genera el checklist de seguridad pre-release para este proyecto Node.js."*
- *"Analiza si esta librería recomendada tiene señales de paquete malicioso o typosquatting."*
- *"¿Qué acciones inmediatas tomar si instalamos una versión de NPM comprometida?"*
- *"Genera una configuración de pre-commit hooks con detección de secretos y SAST."*
- *"Revisa este pipeline de CI/CD y aplica todos los controles de seguridad del SLSA nivel 2."*
- *"Crea el SBOM del proyecto y configura el escaneo automático de vulnerabilidades."*

> ⚠️ Esta skill es **complementaria** a `qa_devsecops_security`. La distinción es:
> - **`qa_devsecops_security`** → Seguridad en el contexto específico de frameworks de QA (Playwright, Karate, K6, Cypress, datos de prueba, PII).
> - **`software_security_hardening`** → Seguridad en el ciclo de desarrollo completo (SDLC): código, dependencias, pipeline, infraestructura, respuesta a incidentes.
