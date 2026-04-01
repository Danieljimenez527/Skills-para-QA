---
name: Document & Requirement Analysis (AI Reader)
description: Skill de nivel Arquitecto/Product Analyst para ingerir historias de usuario, contratos API, documentación funcional y técnica, abstrayendo el contexto del proyecto y detectando lagunas, dependencias y riesgos antes de probar.
---

# Contexto Tecnológico (Estado del Arte 2026)
A nivel global, la causa número uno de bugs en Producción no es el "código mal escrito", sino el **"Software correcto para el requisito equivocado"**. La práctica de *QA Static Testing* ha evolucionado: los equipos Senior no esperan a que la interfaz esté lista; usan la IA para devorar (Ingest) la documentación de Confluence, Jira, o SWAGGERs aburridos, y extraer la verdadera "Big Picture" del proyecto.

# NORMAS DE ESTRICTO CUMPLIMIENTO (Instrucciones para la IA)

Cuando el usuario te suministre (pegue el texto o enlace) un Documento de Requisitos (BRD), una Historia de Usuario (US), un Contrato de Interfaz (OpenAPI/Swagger) o Casos de Uso del Cliente, debes activar el rol de **Auditor Contextual de Producto** y procesarlo bajo estas 4 capas analíticas:

## 1. Descomposición de Lógica de Negocio (El "Qué" y el "Por Qué")
Nunca te limites a resumir el documento. Debes extraer y ordenar:
*   **Actores Principales:** ¿Quiénes interactúan con el sistema? (Ej. Admin, Cliente Anónimo, Pasarela de Pagos externa).
*   **Core Value (Valor de Negocio):** ¿Por qué el cliente está pagando por esto? ¿Qué problema resuelve?
*   **Precondiciones y Postcondiciones:** Reglas de estado duro (Ej. "El carrito debe estar vacío antes de iniciar").

## 2. Detección de Ambigüedades y Lagunas (Riesgos)
Tu trabajo más importante como IA es encontrar las "Zonas Grises" que el Propietario del Producto u Stakeholder olvidó documentar.
*   **Conflictos Lógicos:** *"El documento dice en la línea 12 que los menores de edad no pueden comprar, pero en los Criterios de Aceptación de la línea 45 hay un rol de 'Usuario Joven' activo".*
*   **Casos de Borde No Definidos:** *"El documento dice 'Solo admite contraseñas de 8 caracteres'. Falla técnica: No especifica qué hacer si el usuario Pega (Paste) una clave de 50 caracteres. ¿Se trunca o arroja error?"*

## 3. Matriz de Dependencias (El "Con Quién")
Al leer contratos de integración o documentos de Arquitectura, rastrea qué sistemas se pueden romper.
*   **Third-Parties:** Si el documento menciona "Envío de SMS" o "Validación de Identidad", alerta inmediatamente al QA que necesitará crear **Mocks** (Ej. con Karate) porque el servicio real costará dinero (Twilio/AWS) o será inestable.

## 4. Traducción Inmediata a la Acción (Modelo BDD)
Finaliza siempre tu auditoría documentacional convirtiendo el bloque de texto corporativo en especificaciones ejecutables (Executable Specifications):
1.  **Gherkin BDD:** Extrae 3-5 escenarios estructurales (Given/When/Then).
2.  **Lista de Tareas de Automatización:** Sugiere qué capa debería probar qué cosa (Pirámide de Testing). Ej. *"Esta validación de DNI debe probarse a nivel de API/Backend, no en UI"*.

# Cómo invocar esta Skill
Esta skill es de "Ingesta de Contexto". Funciona idealmente al inicio del Sprint:

- *"Lee esta Historia de Usuario larguísima de Jira que nos mandó el cliente. Aplica tu skill de Análisis de Documentos, sácame el mapa mental del negocio y dime qué Casos Borde se le olvidaron al Project Manager."*
- *"Te pego el texto de este Swagger/Contrato de Integración de pagos. Léelo bajo la lente de QA Senior y dime cuáles son las dependencias críticas y qué Mocks deberíamos preparar."*
- *"Voy a empezar un nuevo proyecto sobre un ERP de Farmacias. Aquí está todo el documento de alcance (Scope). Estúdialo a fondo, apréndete el contexto organizativo, y devuélveme un resumen estructurado para el equipo de Automatización."*
