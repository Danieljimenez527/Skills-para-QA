---
name: JMeter Performance Engineering & Observability Senior
description: Skill de nivel Senior para diseñar, ejecutar y analizar pruebas de rendimiento de clase mundial con Apache JMeter, aplicando patrones avanzados de carga, protocolos modernos, integración CI/CD y RCA guiado por IA.
---

# Contexto Tecnológico (Estado del Arte 2026)

Apache JMeter sigue siendo la herramienta de referencia global para pruebas de rendimiento en entornos enterprise. A diferencia de lo que se enseña en tutoriales básicos, los equipos Senior de Performance Engineering en organizaciones como Netflix, Amazon y BBVA aplican JMeter bajo tres pilares de alto impacto:

1. **Test como Código (JMX-as-Code con DSL y Taurus):** Nunca se trabaja con archivos `.jmx` editados manualmente en la GUI para entornos CI/CD. Los scripts maestros se versionan en Git y se ejecutan con herramientas headless (CLI) o con frameworks abstractores como **BlazeMeter Taurus** (`bzt`), que convierte YAML legible en planes de prueba JMeter complejos.
2. **Protocolos Avanzados y Correlación Dinámica:** Pruebas de rendimiento modernas van más allá de HTTP simple. Se cubren **HTTP/2, WebSockets, gRPC, JDBC (carga de BD), JMS (colas de mensajería)** y, lo más crítico: la **correlación dinámica de tokens** (CSRF, OAuth 2.0, JWT) para simular sesiones de usuario reales sin hardcodear credenciales.
3. **Análisis de Resultados y CI/CD Gating con InfluxDB + Grafana:** Un reporte `.jtl` o el dashboard por defecto de JMeter es insuficiente para un equipo Senior. Las métricas de ejecución se envían en tiempo real a **InfluxDB** y se visualizan en **Grafana** con dashboards pre-configurados. Si los KPIs cruzan umbrales definidos, el **pipeline de CI/CD se bloquea automáticamente** (JMeter CLI exit code). 

---

# Instrucciones para el Agente (IA)

Cuando el usuario solicite asistencia con JMeter, activa inmediatamente los siguientes patrones avanzados según el contexto. **Nunca generes pruebas simples Thread Group / HTTP Sampler sin configuración de rendimiento, Listeners de reporte o Assertions.**

---

## Patrón 1: Estructura Maestra de un Plan de Pruebas Enterprise

Todo plan de pruebas JMeter de nivel Senior debe seguir esta estructura de carpetas y elementos obligatorios:

```
Test Plan (Nombre descriptivo: "LoadTest_CheckoutFlow_v1.2")
│
├── 🔧 [CONFIG] Config Elements
│   ├── HTTP Request Defaults        → Baseurl, Timeouts (Connect: 5000ms, Response: 10000ms)
│   ├── HTTP Cookie Manager          → Gestión automática de sesión
│   ├── HTTP Cache Manager           → Simula caché de browser real
│   ├── CSV Data Set Config          → Usuarios, datos de pago, IDs (NUNCA hardcoded)
│   └── User Defined Variables       → env, region, apiVersion
│
├── 🔑 [SETUP] setUp Thread Group (1 hilo, 1 iteración)
│   ├── HTTP Request: POST /auth/token  → Obtiene token OAuth/JWT
│   └── JSON Extractor: token_global    → Guarda token en variable global
│
├── 👥 [LOAD] Thread Group: Carga Sostenida
│   ├── [Patrón Rampa]: Ramp-Up configurado a n hilos en m segundos
│   ├── HTTP Header Manager          → Authorization: Bearer ${token_global}
│   ├── Throughput Controller        → Controla % de distribución por escenario
│   │
│   ├── 📦 Transaction Controller: "TC_01_BrowseProducts"
│   │   ├── HTTP Request: GET /api/products
│   │   ├── JSON Extractor: product_id  → Correlación dinámica
│   │   └── Response Assertion: status 200, body contains "products"
│   │
│   └── 📦 Transaction Controller: "TC_02_AddToCart"
│       ├── HTTP Request: POST /api/cart
│       │   └── Body usa ${product_id} del extractor anterior
│       └── Duration Assertion: < 2000ms
│
├── 🌪️ [SPIKE] Thread Group: Spike / Pico Repentino
│   └── (Factor de carga 5x por 30 segundos)
│
├── 📊 [LISTENERS] Backend Listener → InfluxDB (tiempo real)
└── 🔻 [TEARDOWN] tearDown Thread Group → POST /auth/logout
```

**Regla de Oro del Agente:** Si el usuario no menciona `CSV Data Set`, `Extractors`, ni `Assertions`, debes agregarlos y explicar por qué son obligatorios antes de generar cualquier script.

---

## Patrón 2: Configuración de Carga Realista (No VU Lineales)

Un Senior nunca usa Ramp-Up lineal simple. Existen perfiles de carga que replican la realidad:

### A) Carga Sostenida con Rampa Gradual (¿Cuánto aguanta el sistema en régimen?)
```yaml
# Equivalente Taurus/YAML para JMeter (más legible que JMX)
execution:
  - concurrency:
      timeline:        # Escalada tipo "rampa S"
        0: 0
        60s: 50
        3m: 200
        5m: 200        # Plateau de carga pico
        6m: 0
    ramp-up: 60s
    hold-for: 4m
    scenario: checkout_flow

scenarios:
  checkout_flow:
    requests:
      - url: ${base_url}/api/products
        label: GET_Products
        assert-response-code: 200
        assert: [contains: "items"]
```

### B) Spike Test (¿Qué pasa cuando llega el Black Friday repentinamente?)
```yaml
execution:
  - concurrency:
      timeline:
        0: 0
        10s: 500      # Pico abrupto de 500 usuarios en 10 segundos
        30s: 500
        31s: 20       # Caída inmediata
    scenario: homepage_spike
```

### C) Soak Test / Endurance (¿Hay memory leaks tras 8 horas?)
```yaml
execution:
  - concurrency: 50
    hold-for: 8h       # Carga moderada sostenida por horas
    scenario: soak_flow
```

**Instrucción del Agente:** Cuando el usuario diga "prueba de estrés" o "carga", pregunta cuál perfil necesita: **Carga, Spike o Soak**, y genera el YAML/JMX correspondiente.

---

## Patrón 3: Correlación Dinámica de Sesiones OAuth 2.0 / JWT

Esta es la habilidad que distingue a un QA Senior de uno Junior. Los tokens expiran; hay que re-obtenerlos automáticamente.

```xml
<!-- Fragmento JMX: setUp Thread Group para OAuth 2.0 -->

<!-- HTTP Request Sampler: POST /oauth/token -->
<HTTPSamplerProxy>
  <stringProp name="HTTPSampler.path">/oauth/token</stringProp>
  <stringProp name="HTTPSampler.method">POST</stringProp>
  <!-- Body: grant_type=client_credentials&client_id=${client_id}&client_secret=${client_secret} -->
</HTTPSamplerProxy>

<!-- JSON Extractor: Captura access_token de la respuesta -->
<JSONPathExtractor>
  <stringProp name="JSONPathExtractor.names">ACCESS_TOKEN</stringProp>
  <stringProp name="JSONPathExtractor.jsonPathExprs">$.access_token</stringProp>
  <stringProp name="JSONPathExtractor.defaultValues">TOKEN_NOT_FOUND</stringProp>
</JSONPathExtractor>

<!-- BeanShell / JSR223 Assertion: Validar que el token no sea NOT_FOUND -->
<JSR223Assertion>
  <stringProp name="script">
    if (vars.get("ACCESS_TOKEN").equals("TOKEN_NOT_FOUND")) {
      AssertionResult.setFailureMessage("OAuth token extraction FAILED - Check credentials");
      AssertionResult.setFailure(true);
    }
    // Guardar como propiedad global accesible por todos los Thread Groups
    props.put("GLOBAL_TOKEN", vars.get("ACCESS_TOKEN"));
    log.info("✅ OAuth Token obtenido exitosamente");
  </stringProp>
</JSR223Assertion>
```

**Uso en Thread Groups de carga:** En el HTTP Header Manager de cada sampler:
```
Authorization: Bearer ${__P(GLOBAL_TOKEN,)}
```

---

## Patrón 4: CI/CD Integration y Pipeline Breaker (Jenkins / GitHub Actions)

### GitHub Actions - Pipeline de Performance:
```yaml
# .github/workflows/performance.yml
name: Performance Regression Gate

on:
  pull_request:
    branches: [main, develop]

jobs:
  jmeter-load-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup JMeter
        run: |
          wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.6.3.tgz
          tar -xzf apache-jmeter-5.6.3.tgz

      - name: Run JMeter Test (Headless CLI)
        run: |
          apache-jmeter-5.6.3/bin/jmeter \
            -n \
            -t tests/performance/checkout_load.jmx \
            -l results/results.jtl \
            -e -o results/html-report \
            -Jbase_url=${{ secrets.STAGING_URL }} \
            -Jclient_id=${{ secrets.API_CLIENT_ID }} \
            -Jclient_secret=${{ secrets.API_CLIENT_SECRET }} \
            -Jthreshold_error_rate=1 \
            -Jthreshold_p95_ms=250

      - name: Evaluate Performance Gate
        run: |
          # Parsear JTL y bloquear pipeline si SLAs fallan
          python3 scripts/evaluate_jtl.py --input results/results.jtl \
            --max-error-rate 1.0 \
            --max-p95 250

      - name: Upload HTML Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: jmeter-report
          path: results/html-report/
```

### Script Python evaluador de SLAs (`evaluate_jtl.py`):
```python
import pandas as pd
import sys
import argparse

def evaluate(jtl_path, max_error_rate, max_p95):
    df = pd.read_csv(jtl_path)
    df.columns = df.columns.str.strip()

    # Métricas clave
    total = len(df)
    errors = df[df['success'] == False].shape[0]
    error_rate = (errors / total) * 100
    p95 = df['elapsed'].quantile(0.95)

    print(f"📊 Total Requests   : {total}")
    print(f"❌ Error Rate       : {error_rate:.2f}% (Umbral: {max_error_rate}%)")
    print(f"⏱️  Latencia P95     : {p95:.0f}ms (Umbral: {max_p95}ms)")

    failed = False
    if error_rate > max_error_rate:
        print(f"🚨 PIPELINE BLOQUEADO: Error rate {error_rate:.2f}% supera {max_error_rate}%")
        failed = True
    if p95 > max_p95:
        print(f"🚨 PIPELINE BLOQUEADO: P95 {p95:.0f}ms supera {max_p95}ms")
        failed = True

    if not failed:
        print("✅ Todos los SLAs de rendimiento APROBADOS")
    
    sys.exit(1 if failed else 0)

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--input", required=True)
    parser.add_argument("--max-error-rate", type=float, required=True)
    parser.add_argument("--max-p95", type=float, required=True)
    args = parser.parse_args()
    evaluate(args.input, args.max_error_rate, args.max_p95)
```

---

## Patrón 5: Observabilidad en Tiempo Real (InfluxDB + Grafana)

### Configuración del Backend Listener en JMeter:
```
Backend Listener Implementation: org.apache.jmeter.visualizers.backend.influxdb.InfluxdbBackendListenerClient

Parámetros:
  influxdbMetricsSender  : org.apache.jmeter.visualizers.backend.influxdb.HttpMetricsSender
  influxdbUrl            : http://influxdb:8086/write?db=jmeter
  application            : checkout-service
  measurement            : jmeter
  summaryOnly            : false
  samplersRegex          : .*
  testTitle              : LoadTest_CheckoutFlow
  percentiles            : 90;95;99
  useRegexpForSamplersList: true
```

**Dashboard Grafana recomendado:** ID `1152` (Apache JMeter Dashboard) - importar directamente desde grafana.com.

**KPIs que el agente debe monitorear:**
| Métrica            | Umbral Aceptable | Umbral Crítico |
|--------------------|-----------------|----------------|
| P95 Latencia       | < 250ms         | > 500ms        |
| Tasa de Error HTTP | < 1%            | > 5%           |
| Throughput (RPS)   | > 100 req/s     | < 50 req/s     |
| P99 Latencia       | < 500ms         | > 1000ms       |
| Error de Red       | 0%              | > 0.1%         |

---

## Patrón 6: Root Cause Analysis (RCA) guiado por IA

Cuando el usuario pegue un log, reporte `.jtl`, output de consola o resultados de JMeter que fallaron, el agente NO debe simplemente decir "el test falló". Debe aplicar el siguiente árbol de diagnóstico:

```
¿Falló con alto P95 (latencia)?
├── ¿Data Received es bajo pero latencia alta?
│   └── → Cuello de botella en CPU/BD del servidor (TTFB alto)
│       → Acción: Revisar query plans, connection pool de BD
│
├── ¿Data Received es alto y latencia alta?
│   └── → Ancho de banda saturado (throughput de red)
│       → Acción: Revisar CDN, compresión gzip, payload size
│
└── ¿Latencia crece con el tiempo (soak)?
    └── → Memory Leak en la aplicación
        → Acción: Activar JVM heap dumps, revisar GC logs

¿Falló con alta tasa de errores HTTP 5xx?
├── ¿Error desde el inicio?
│   └── → Configuración incorrecta (auth, URL, datos de entrada)
│
└── ¿Error aparece tras N minutos de carga?
    └── → Resource Exhaustion: Connection Pool, File Descriptors, Threads
        → Acción: Revisar max-connections del server, reintentos automáticos

¿Falló con Connection Refused / Timeout de conexión?
└── → JMeter mismo está saturado (no el servidor)
    → Acción: Aumentar heap JMeter (-Xms4g -Xmx8g), usar modo distribuido
```

---

# Cómo Invocar esta Skill

El usuario debe escribir comandos de alto nivel como:

- *"Genera un plan de pruebas JMeter para el flujo de login → búsqueda → checkout de un e-commerce, con carga de 200 usuarios concurrentes, OAuth 2.0 y thresholds que bloqueen el pipeline si el P95 supera 300ms."*
- *"Tengo este resultado de JMeter donde el P95 llegó a 1200ms y la tasa de error fue 8%. Actúa como analista RCA y dime cuál es la causa raíz probable."*
- *"Convierte mi prueba JMeter a formato Taurus YAML para poder ejecutarla en GitHub Actions sin la GUI."*
- *"Crea un Spike Test para simular un flash sale con 0 a 800 usuarios en 15 segundos sobre el endpoint /api/flash-sale."*
- *"¿Cómo configuro JMeter Distribuido con 3 slaves para generar 10,000 usuarios concurrentes desde múltiples regiones geográficas?"*
- *"Configura el Backend Listener de JMeter para enviar métricas en tiempo real a InfluxDB y crea la query de Grafana para alertar si el P99 supera 500ms."*
