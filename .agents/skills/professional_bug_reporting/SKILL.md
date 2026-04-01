---
name: Professional Bug Reporting & AI RCA
description: Skill de nivel Senior para documentar bugs con máxima calidad técnica, estructuración perfecta para Jira/Linear, y Análisis de Causa Raíz (RCA) potenciado por IA.
---

# Contexto (Estado del Arte 2026)
En el nivel Senior, documentar un bug ya no se trata de "escribir qué falló". Es una herramienta de **comunicación e ingeniería** diseñada para minimizar el ida y vuelta (Back-and-forth) con el equipo de desarrollo.
Además, la IA ha transformado este proceso mediante la técnica *"Trash In, Gold Out"*: El QA inyecta notas desordenadas y logs al Agente IA, y el Agente genera un ticket estructurado a prueba de balas, acompañado de un Análisis de Causa Raíz preliminar.

# Instrucciones para el Agente (IA) a ejecutar

Cuando el usuario requiera que documentes, evalúes o generes el reporte de un Bug introducido en el proyecto, debes estructurarlo estrictamente bajo el formato global de Alta Ingeniería:

## 1. Plantilla Estricta de Documentación (Generación de Ticket)
Nunca generes un reporte como un bloque de texto plano. Aplica este formato Markdown listo para copiar y pegar en Jira/Linear/Azure DevOps:

```markdown
### 🐛 [Módulo/Componente] - Título descriptivo (Qué, Dónde y Cuándo)
**Severidad:** `[Blocker | Critical | Major | Minor]` | **Prioridad:** `[High | Medium | Low]`

#### 📝 Resumen
(1-2 oraciones explicando el impacto en el negocio o el usuario).

#### 🌍 Entorno
* **OS / Dispositivo:** `(Ej. Windows 11 / iPhone 15)`
* **Navegador / Versión:** `(Ej. Chrome 124)`
* **Entorno:** `(Ej. Staging / Production)`
* **Versión de App:** `(Ej. v2.4.1)`

#### 👣 Pasos para Reproducir (Exactos)
1. Navegar a `[URL o Pantalla]`
2. Ingresar el valor `[Dato de Prueba]` en `[Campo]`
3. ...

#### ❌ Resultado Actual (Lo que pasó)
* (Descripción clara de la falla técnica y visual)

#### ✅ Resultado Esperado (El contrato / La regla de negocio)
* (Lo que debía pasar según los criterios de aceptación)

#### 📎 Evidencia Técnica (Muy Importante)
* (Logs de consola, Peticiones de Red fallidas (cURL/HAR), Enlace a Screenshot/Video).
```

## 2. Asistencia Analítica Avanzada (Root Cause Analysis - RCA)
Si el usuario te envía (como IA) un Log, un Stacktrace o el resultado de un test fallido (Ej. Cypress o Karate), NO te limites a llenar la plantilla de arriba. Debes aplicar la técnica de los **5 Porqués (5 Whys)** para deducir la raíz del problema:

1. **Lee el Log:** Observa dónde se rompió el código de la aplicación.
2. **Genera Hipótesis:** Agrega una sección llamada `💡 Posible Causa Raíz (Generada por IA)` al final del reporte del bug.
3. **Sugiere Fixes al Desarrollador:** Un QA Senior no solo reporta, también ayuda a depurar. Si detectas un `NullReferenceException`, sugiere en el reporte que probablemente falte una validación en la capa del controlador backend antes de renderizar la vista.

## 3. Filosofía "Cero Culpa" y Tono
El agente IA debe asegurar que el tono de los bugs redactados sea completamente neutral, objetivo y colaborativo. 
*   *Inadecuado:* "El desarrollador rompió el login al subir su cambio".
*   *Profesional:* "El endpoint del login está arrojando un Error 500 tras la última integración".

# Cómo invocar esta Skill
- *"Tengo unas notas rápidas: El botón de pago en iOS no hace nada cuando uso la tarjeta terminada en 4242. Arrégla esto y redáctame un Bug Nivel Senior para Jira."*
- *"Evalúa este Bug que voy a reportar y dime si me falta algún campo según las mejores prácticas."*
- *"Te adjunto el log del error 500 que sacó el backend de Java. Sácame el reporte de bug y añádele tu Análisis de Causa Raíz para que el Dev lo arregle más rápido."*
