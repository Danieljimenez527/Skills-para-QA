---
name: Playwright AI Integration (ZeroStep & AutoPlaywright)
description: Skill de nivel Senior para implementar agentes de IA (como ZeroStep o AutoPlaywright) dentro del ecosistema Playwright, logrando Self-Healing y pruebas E2E mediante lenguaje natural sin selectores frágiles.
---

# Contexto Tecnológico (Estado del Arte 2026)
A nivel global, la comunidad y documentación de **Playwright** apuntan hacia pruebas más resilientes y orientadas a la accesibilidad (`getByRole`). Sin embargo, el paradigma ha evolucionado hacia la **Automatización Impulsada por IA**. 
Herramientas e integraciones como **ZeroStep** (`@zerostep/playwright`) y **Auto Playwright** han cobrado mucha fuerza, permitiendo invocar la toma de decisiones dinámicas basadas en LLMs dentro de la ejecución de Playwright para resolver problemas de *Self-Healing* y abstracción del código.

# Instrucciones para el Agente (IA) a ejecutar

Cuando se active esta skill para transformar o crear scripts de Playwright, debes adherirte a las siguientes reglas:

## 1. Patrones de Diseño AI-Driven en Playwright

### A. Integración Oficial/Recomendada de terceros: ZeroStep
ZeroStep está diseñado para trabajar nativamente con Playwright. Cuando el usuario solicite implementarlo, usa la función `ai()` que actúa como un Agente evaluador visual/semántico:

```typescript
import { test, expect } from '@playwright/test';
import { ai } from '@zerostep/playwright';

test('Flujo de pago resiliente con IA', async ({ page }) => {
  await page.goto('https://tienda.com/checkout');
  
  // ZeroStep infiere interacciones leyendo el DOM sin selectores estrictos
  await ai('Fill the shipping form with John Doe, 123 Main St', { page, test });
  await ai('Click on the primary checkout button', { page, test });
  
  // Utilizar asserts nativos de Playwright (Híbrido)
  await expect(page.locator('.order-success')).toBeVisible();
});
```

### B. Opciones Alternativas Abiertas: Auto Playwright
Para integraciones custom open source usando la API de OpenAI (GPT-4) de forma directa:
```typescript
import { test, expect } from '@playwright/test';
import { auto } from 'auto-playwright';

test('Búsqueda compleja con IA', async ({ page }) => {
  await page.goto('https://mi-aplicacion.com');
  // Ejecución dinámica 
  await auto('Busca el reporte mensual de ventas y descárgalo', { page, test });
});
```

## 2. Configuración y Reglas para el QA Senior
- **Enfoque Híbrido (Best Practice):** Como QA Senior, nunca debes automatizar 100% el script con IA. La IA debe usarse para **navegación compleja, relleno de formularios masivos, y evasión de selectores dinámicos**. Los assertions (`expect()`) y validaciones de negocio **DEBEN** mantenerse estrictos usando las APIs nativas de Playwright.
- **Timeouts:** Las llamadas de LLM agregan latencia. Incrementa el config `timeout` e `expect.timeout` en `playwright.config.ts` (ej. a 60-90 segundos) en proyectos con integración de Agentes de IA.
- **Gestión de variables:** No quemar los tokens (`ZEROSTEP_TOKEN` u `OPENAI_API_KEY`) en el código. Exigir siempre el uso de `process.env`.

## 3. Acciones que la IA puede ejecutar por el usuario:
1. **Refactorización Self-Healing:** Tomar un script viejo de Playwright (con Xpaths y selectores frágiles) y reescribirlo usando la función `ai()`.
2. **Setup:** Generar los comandos `npm install` y los ajustes del `playwright.config.ts` necesarios para usar la herramienta.

# Cómo invocar esta Skill
El usuario o el sistema deben activar esta skill indicando:
- *"Integra ZeroStep en el test de mi carrito de compras."*
- *"Refactoriza este código viejo de Playwright usando la skill de IA Agent para que sea a prueba de fallos (Self-Healing)."*
- *"Crea el archivo de configuración para usar Auto Playwright en el proyecto actual."*
