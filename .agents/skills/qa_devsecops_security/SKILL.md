---
name: QA DevSecOps & AI Data Privacy
description: Skill de máxima prioridad (Nivel Arquitecto/CISO) para asegurar automatizaciones, prevenir fugas de datos (PII), ataques de cadena de suministro (Malware) e inyecciones en procesos de IA.
---

# Contexto Tecnológico (Estado del Arte 2026)
A medida que la automatización de QA se ha fusionado con la Inteligencia Artificial, la superficie de ataque ha crecido exponencialmente. Los repositorios de Testing son a menudo el eslabón más débil, ya que los equipos suelen trabajar con réplicas de bases de datos de producción o instalan librerías de terceros (NPM/Maven) sin filtros. A nivel global, la práctica **DevSecOps en QA** exige que toda automatización (Playwright, Karate, k6, Cypress) cuente con barreras inquebrantables de seguridad.

# NORMAS DE ESTRICTO CUMPLIMIENTO (Instrucciones para la IA)

Cuando actúes como Agente de IA para automatizar, generar código, o revisar infraestructuras, **DEBES** hacer cumplir sin excepción las siguientes reglas de ciberseguridad corporativa:

## 1. Patrón de Privacidad Absoluta (Zero-PII & Sintetización)
Nunca generes ni permitas el uso de Identificadores Personales reales (PII: Nombres reales, correos de clientes, tarjetas de crédito, IDs de salud) dentro del código fuente ni en los Prompts.
*   **Enmascaramiento (Data Masking):** Si el usuario te pide crear datos de prueba basados en un JSON de producción, **altera inmediatamente** todos los datos sensibles antes de generar el código.
*   **Generación Sintética:** Obliga el uso de librerías como `faker.js` o Data Fakers nativos de Java para poblar los `.spec.ts` de Playwright o los `.feature` de Karate.
    *   *Malo (Prohibido):* `cy.get('#cc-number').type('4556-9999-0000-1234')`
    *   *Seguro:* `cy.get('#cc-number').type(faker.finance.creditCardNumber())`

## 2. Prevención de Ransomware / Malware (Supply Chain Security)
La IA suele alucinar e inventar nombres de librerías, o recomendar herramientas que podrían haber sido tomadas por atacantes (Ej. paquetes de NPM con nombres similares a los originales - *Typosquatting*).
*   **Auditoría de Dependencias:** Cada vez que recomiendes ejecutar `npm install paquete-x` o añadir un `pom.xml` dependency, debes advertir al usuario que verifique la autenticidad del paquete (Check de vulnerabilidades).
*   **Aislamiento de Mocks:** En Karate y Cypress, los entornos de `Mock` no deben tener permisos de escritura hacia bases de datos de producción, previniendo así un borrado accidental masivo desde una canalización CI infectada.

## 3. Barrera de Secretos (Evitar fuga de Credenciales)
Es la causa #1 de hackeos a nivel mundial en repositorios de Testing.
*   Nunca estructures configuraciones (`playwright.config.ts`, `karate-config.js`) quemando tokens, contraseñas o URLs de Producción ("Hardcoded secrets").
*   Siempre debes inyectar llamadas a variables de entorno (`process.env.TEST_PASSWORD` o llamadas directas a KMS/HashiCorp Vault / AWS Secrets).
*   **Auditoría de Logs (Cypress/Playwright):** Instruye apagar las grabaciones en video o screenshots explícitamente en los pasos donde se digitan contraseñas. Un screenshot de un login a menudo filtra secretos en el pipeline.

## 4. Fugas por IA (Cuidado con los Contextos)
Como tu Agente IA evaluará logs caídos de CI/CD:
*   Si el usuario pega un Log proveniente de un ambiente de Producción que contenga el número de cuenta de un cliente, debes detener el análisis, pedirle al usuario que anonimice el texto, y luego continuar con la recomendación.

# Cómo invocar esta Skill
Esta skill **opera de manera pasiva (Always-On)**, lo que significa que debe auditar silenciosamente todo el código y respuestas generadas en el área de trabajo. Sin embargo, el usuario puede invocarla explícitamente:

- *"Audita mi archivo `playwright.config.ts` aplicando la skill de seguridad QA DevSecOps."*
- *"Genera una fábrica de usuarios sintéticos y seguros (fake data) para inyectar en las pruebas de regresión de Karate sin usar bases reales."*
- *"Revisa las librerías NPM actuales del framework y verifica si hay alguna práctica insegura en el manejo de credenciales antes de subir el Pull Request."*
