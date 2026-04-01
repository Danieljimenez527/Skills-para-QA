---
name: Tech Radar & Monthly Skills Auditor
description: Skill de nivel CTO/QA Architect para ejecutar auditorías mensuales de todas las skills del repositorio, rastrear cambios en versiones oficiales de herramientas (Playwright, Karate, k6, Cypress) y emitir un reporte ejecutivo con recomendaciones de actualización para decisión humana.
---

# Contexto y Filosofía
Esta es la **skill de gobernanza y evolución continua** del repositorio `Skills-para-QA`. Su propósito es garantizar que ningún conocimiento se vuelva un pasivo tecnológico.

Inspirada en el modelo **ThoughtWorks Technology Radar** (el estándar mundial de vigilancia tecnológica adoptado por empresas Fortune 500), esta skill clasifica las herramientas y prácticas en 4 anillos:

| Anillo | Significado |
|--------|-------------|
| 🟢 **ADOPT** | La práctica documentada sigue siendo el estándar mundial. Sin cambios urgentes. |
| 🟡 **TRIAL** | Hay cambios relevantes. Se recomienda evaluar e incorporar en la próxima sesión. |
| 🟠 **ASSESS** | Breaking changes detectados o nueva versión mayor disponible. Requiere revisión urgente. |
| 🔴 **HOLD** | La práctica o herramienta documentada está deprecada o fue reemplazada. Actualización crítica. |

---

# Instrucciones para el Agente (IA) a ejecutar

Cuando el usuario invoque esta skill o el mensaje contenga frases como "Ejecuta auditoría mensual", "Revisa el Tech Radar" o "¿Hay actualizaciones para las skills?", debes ejecutar las siguientes fases en orden:

## FASE 1 — Inventario de Skills del Repositorio
Enumera todas las skills activas registradas en `.agents/skills/`:

| # | Nombre Skill | Herramienta Principal | Última Revisión |
|---|---|---|---|
| 1 | Playwright AI Integration | Playwright + ZeroStep + AutoPlaywright | [fecha auditoría anterior] |
| 2 | Karate AI & Stateful Contract Mocking | Karate Labs | [fecha auditoría anterior] |
| 3 | k6 Performance, Chaos & AI Observability | Grafana k6 + xk6-disruptor | [fecha auditoría anterior] |
| 4 | Cypress AI & Visual E2E | Cypress.io | [fecha auditoría anterior] |
| 5 | Professional Bug Reporting & AI RCA | Metodología QA Global | [fecha auditoría anterior] |
| 6 | QA DevSecOps & AI Data Privacy | OWASP / DevSecOps | [fecha auditoría anterior] |
| 7 | Advanced Manual QA & Exploratory Testing | SBTM / BDD / WCAG | [fecha auditoría anterior] |
| 8 | Document & Requirement Analysis | AI Reader / BDD / OpenAPI | [fecha auditoría anterior] |
| 9 | QA Senior Recommender | Stack Global QA | [fecha auditoría anterior] |

## FASE 2 — Investigación Proactiva de Cambios
Para cada herramienta, consulta sus fuentes oficiales de cambios y clasifica el nivel de impacto:

- **Playwright:** https://playwright.dev/docs/release-notes
- **Karate Labs:** https://github.com/karatelabs/karate/releases
- **Grafana k6:** https://github.com/grafana/k6/releases
- **Cypress:** https://docs.cypress.io/app/references/changelog
- **ZeroStep:** https://github.com/zerostep-ai/zerostep/releases
- **OWASP Top 10 for AI:** https://owasp.org/www-project-top-10-for-large-language-model-applications/
- **W3C WCAG Accessibility:** https://www.w3.org/WAI/standards-guidelines/wcag/

## FASE 3 — Emisión del Reporte Ejecutivo Mensual
Genera el reporte en este formato estricto y profesional:

```markdown
# 📡 REPORTE DE AUDITORÍA TECH RADAR — [MES AÑO]
> Generado automáticamente por la skill: Tech Radar & Monthly Skills Auditor

## Resumen Ejecutivo
- Total de skills auditadas: [N]
- 🟢 Sin cambios críticos: [N]
- 🟡 Con cambios recomendados: [N]
- 🟠 Con breaking changes: [N]
- 🔴 Con prácticas deprecadas: [N]

---

## Detalle por Skill

### 🟢/🟡/🟠/🔴 [Nombre de la Skill] — [Estado]
**Versión Documentada en la Skill:** `[X.X]`
**Última Versión Oficial:** `[Y.Y]`
**Tipo de Cambio:** `[Minor Patch | New Feature | Breaking Change | Deprecation]`

**¿Qué cambió?**
(Descripción técnica concisa del cambio relevante para QA)

**Impacto en la skill actual:**
(¿Seguirá funcionando el código de ejemplo? ¿Cambiaron nombres de funciones?)

**Recomendación de acción:**
- [ ] Actualizar ejemplo de código en la sección X
- [ ] Añadir advertencia sobre API deprecada
- [ ] Crear nuevo patrón para la feature Y

---
## Decisión del QA Architect
> ☑ Pendiente de tu aprobación. Responde "Actualiza [nombre skill]" para que proceda.
```

## FASE 4 — Ejecución Sujeta a Aprobación Humana
**REGLA CRÍTICA:** Esta skill **NUNCA modifica automáticamente** ningún archivo `.SKILL.md` del repositorio. Solo presenta el reporte y espera la aprobación explícita del usuario para cada skill que requiera cambios.

Cuando el usuario diga **"Actualiza [nombre skill]"**, entonces y solo entonces, procederás a editar el SKILL.md correspondiente preservando la estructura e inyectando los cambios aprobados.

---

# Cadencia de Activación Recomendada
| Evento | Acción |
|--------|--------|
| **Inicio de cada mes** | Ejecutar auditoría completa de las 9 skills |
| **Inicio de proyecto nuevo** | Auditar solo las herramientas relevantes al stack del proyecto |
| **Cuando falle un script** | Verificar si el fallo se debe a una versión nueva incompatible |
| **Cuando llegue una nueva herramienta** | Solicitar al Recomendador Global que evalúe si merece una nueva skill |

---

# Cómo invocar esta Skill
- *"Ejecuta la auditoría mensual de Tech Radar sobre todas las skills."*
- *"¿Ha cambiado algo importante en Playwright o Karate desde la última auditoría?"*
- *"Revisa si hay breaking changes en k6 v0.55 que afecten nuestra skill de performance."*
- *"Actualiza la skill de Cypress con los cambios del reporte que aprobé."*
