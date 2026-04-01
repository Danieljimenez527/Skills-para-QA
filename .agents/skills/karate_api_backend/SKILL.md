---
name: Karate AI & Stateful Contract Mocking (Backend)
description: Skill de nivel Senior para automatización Backend iterativa, Mocks con estado (Stateful), validación estricta de Contratos OpenAPI, y generación dinámica de datos mediante IA.
---

# Contexto Tecnológico (Estado del Arte)
A nivel mundial, **Karate Labs** es el líder unificado para pruebas Backend porque elimina la necesidad de escribir paso a paso el código Java/JS para realizar peticiones HTTP. La documentación oficial vigente destaca tres grandes fortalezas para un QA Senior:
1. **Mocking Stateful (Mocks con Estado):** La capacidad de levantar un servidor falso muy ligero (basado en Netty) que *recuerda* las transacciones anteriores de la prueba. 
2. **Contract Testing:** Usar los mismos archivos `.feature` para validar que los servicios reales cumplen exactamente con las especificaciones OpenAPI (Swagger) sin tener que depender de herramientas extra como Pact.
3. **Mantenimiento guiado por IA:** El uso de asistentes de Inteligencia Artificial para inferir la estructura esperada (JSON/XML) y aplicar técnicas de *Self-Healing* en las aserciones de Karate cuando los contratos cambian.

# Instrucciones para el Agente (IA) a ejecutar

Cuando el usuario active esta skill para generar o refactorizar código de Karate, deberás aplicar los siguientes patrones Senior:

## 1. Patrón: Stateful Mocks (Simulación Realista de APIS)
Cuando el usuario te pida crear un Mock para una API que no está lista o que es inestable, NUNCA devuelvas respuestas estáticas ("Happy Path"). Debes generar un mock que guarde información.

**Plantilla a generar (Ejemplo):**
```gherkin
Feature: API Mock de Pagos (Stateful)

Background:
  # Definir memoria compartida para el estado
  * def balance = 1000

Scenario: pathMatches('/api/payment') && methodIs('post')
  * def amount = request.amount
  * if (amount > balance) karate.abort()
  * eval balance = balance - amount
  * def response = { success: true, remaining: '#(balance)' }
  * def responseStatus = 200
```
*Instrucción AI:* Explica siempre al usuario cómo levantar este mock temporalmente (`java -jar karate.jar -m mock.feature`).

## 2. Patrón: Contract Testing (Validación basada en OpenAPI)
En arquitectura orientada a microservicios, verificar los campos JSON manuales ("escribe aserción por aserción") es una práctica Junior. Como Senior, usa la validación de esquemas difusos (Fuzzy Matchers).

**Plantilla a generar (Ejemplo):**
```gherkin
Feature: Contract Testing contra Swagger

Background:
  # Cargar el contrato esperado dinámicamente
  * url 'https://api.empresa.com'
  * def userSchema = read('classpath:schemas/user-schema.json')

Scenario: Validar Contrato de Usuario
  Given path '/users/1'
  When method GET
  Then status 200
  # Validación estricta con Fuzzy Matcher "#(^userSchema)" 
  # Valida que todos los campos y tipos coincidan sin importar la data específica
  And match response == "#(^userSchema)" 
```

## 3. Generación Asistida por IA (Manejo de Datos y "Self-Healing")
Karate puede volverse caótico en la gestión de Data-Driven tests. Como IA, tu función principal aquí será:
- **Refactorizar referencias mutables:** Si el usuario tiene JSON compartidos que fallan en bucles, aplica `* def myData = karate.copy(originalData)` para evitar colisiones de memoria.
- **Self-Healing de Peticiones:** Si el usuario te pasa un Log de CI donde falló un `.feature` de Karate debido a un cambio de esquema en BD, detecta la aserción rota (`match response.id == '#string'`) y actualízala inmediatamente de forma automática (ej. a `#number`).

# Cómo invocar esta Skill
El usuario o el sistema deben activar esta skill indicando:
- *"Levanta un Stateful Mock en Karate para simular una pasarela de Stripe."*
- *"Refactoriza estos asertos estáticos de Karate y transfórmalos a Contract Testing validando mi archivo user.json."*
- *"Aplica la Skill de IA a mi reporte de fallo de Karate, identifica el nodo del JSON afectado y repara el código."*
