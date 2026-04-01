---
name: Cypress AI & Visual E2E
description: Skill de nivel Senior para implementar Cypress con integraciones de IA guiadas, pruebas de regresión visual deterministas y estabilización avanzada mediante Network Stubbing.
---

# Contexto Tecnológico (Estado del Arte 2026)
**Cypress** mantiene su liderazgo en E2E gracias a su arquitectura única que se ejecuta directamente en el mismo Loop de eventos del navegador. Basado en su documentación oficial actual y prácticas globales nivel Senior (SRE/QA Architect), las estrategias para Cypress se concentran en:

1. **Auto-Curación Guiada (Self-Healing):** A nivel mundial, el enfoque de IA en Cypress no es tanto la navegación "a oscuras", sino la **reparación semántica de localizadores rotos** en el pipeline y la re-generación predictiva del árbol DOM, pero **exigiendo revisión humana (Human-in-the-loop)** para evitar aserciones falsos positivos.
2. **Determinismo Visual (Visual Regression):** Las pruebas visuales basadas en píxeles son inútiles si la aplicación es dinámica. La práctica global obliga a "congelar el estado y el tiempo" de la app antes de la captura de pantalla (`cy.intercept` y `cy.clock`).
3. **Sincronía mediante Red:** Prohibición absoluta y penalizada del `cy.wait(ms)` genérico. El estándar es enclavar el DOM a las resoluciones asíncronas de la red (API).

# Instrucciones para el Agente (IA) a ejecutar

Cuando sea invocado para generar, refactorizar o evaluar código de Cypress, debes imponer de inmediato estos patrones de arquitectura:

## 1. Patrón: Anti-Flakiness (Sincronización por Red)
Nunca insertes ni permitas `cy.wait(2000)`. Para un perfil Senior, debes escanear el requerimiento e implementar `cy.intercept` como promesa de estabilidad.

**Plantilla Base:**
```javascript
describe('Estrategia de Estabilidad QA Senior', () => {
  it('Validación enlazada al Backend (Network Stubbing)', () => {
    // 1. Apuntar y declarar el alias a la petición incierta
    cy.intercept('POST', '**/auth/login').as('loginRequest');
    
    // 2. Disparar Acción UI
    cy.get('[data-cy="btn-login"]').click();
    
    // 3. Estabilizar: Esperar evento XHR, NO el tiempo.
    cy.wait('@loginRequest').its('response.statusCode').should('eq', 200);
    
    // 4. Aserción segura en el DOM (El elemento ya renderizó)
    cy.get('[data-cy="dashboard-title"]').should('be.visible');
  });
});
```

## 2. Patrón de Localización: AI Self-Healing vs Atributos Duros
Las herramientas de IA actuales (o tú como Agente) pueden reparar selectores rotos. Pero como Arquitecto, la guía oficial dicta la primacía de los selectores explícitos.
*   **Regla de Generación:** Al crear código Cypress desde cero, ignora XPath y selectores anidados. Usa **SÓLO** el estándar `data-cy` o `data-testid`.
*   **Regla de Refactorización (AI Action):** Si un usuario pega un log `DefaultError: element not found` por `.nav > ul > li:nth-child(2)`, debes reparar el código migrándolo preferiblemente a localizadores basados en roles o texto semántico (ej: `cy.contains('Perfil')`).

## 3. Patrón Avanzado: Regresión Visual Inmutable
Si el entorno requiere integración con plugins visuales (Percy, Applitools, o cypress-image-diff-js), la IA debe advertir y configurar el entorno estático o de lo contrario la prueba fallará estrepitosamente.

**Plantilla Base Visual:**
```javascript
it('Garantiza la inmutabilidad Visual del Reporte', () => {
  // Congelar Reloj del navegador para evitar que la "Hora actual" arruine la captura
  const fixedDate = new Date(2026, 3, 15).getTime();
  cy.clock(fixedDate);

  // Aislar de la base de datos real simulando un FIXTURE (JSON estático)
  cy.intercept('GET', '**/charts/data', { fixture: 'reportProfileData.json' }).as('getChart');

  cy.visit('/report');
  cy.wait('@getChart'); // Estabilización
  
  // Ahora el Pixel Matching es 100% confiable
  cy.compareSnapshot('dashboard-view', 0.0);
});
```

# Cómo invocar esta Skill
- *"Mi Cypress está muy intermitente. Refactoriza el test de 'Creación de Órdenes' reemplazando tiempos por manejo de interceptores."*
- *"Corrige este test caído (te adjunto el error de Cypress) implementando la skill de Localizadores semánticos y quita esos CSS feos."*
- *"Genere el esqueleto base combinando Cypress y tu mejor entendimiento de Mocking para poder aplicar Regresión Visual estricta en mi aplicación de ventas."*
