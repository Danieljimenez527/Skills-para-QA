---
name: k6 Performance, Chaos Engineering & AI Observability
description: Skill de nivel Senior para implementar pruebas de rendimiento Shift-Left, inyección de fallos (Chaos Engineering) via xk6-disruptor y perfiles de carga dinámicos con k6.
---

# Contexto Tecnológico (Estado del Arte 2026)
Las pruebas de rendimiento modernas han dejado de ser un evento "post-desarrollo". A nivel global, la herramienta líder **Grafana k6** se implementa bajo 3 pilares clave para ingenieros QA Senior y SREs (Site Reliability Engineers):
1. **Shift-Left y Presupuestos (Thresholds):** Ejecutar k6 en cada Pull Request (CI/CD). No basta con "ver si la app responde bajo estrés"; se definen estrictos *Presupuestos de Rendimiento* en código. Si una API nueva supera los SLAs globales (ej. p95 > 200ms), el pipeline la bloquea automáticamente.
2. **Chaos Engineering (`xk6-disruptor`):** Integración nativa de inyección de caos. Permite evaluar simultáneamente la carga y la **resiliencia**. (*¿Qué pasa si mientras hay 1000 usuarios comprando, el pod de Kubernetes de la BD sufre 500ms de latencia?*)
3. **Observabilidad y RCA guiado por IA:** Correlación directa de métricas de k6 con herramientas como Prometheus. La IA (o este agente) juega un rol fundamental analizando los dumps de resultados y descifrando los cuellos de botella (CPU, Memory Leaks, Red) en el backend.

# Instrucciones para el Agente (IA) a ejecutar

Cuando el usuario requiera asistencia con k6, debes aplicar inmediatamente estos patrones avanzados:

## 1. Patrón: CI/CD Performance Budgets (Thresholds Estrictos)
Nunca generes un script de k6 "Hola Mundo" (solo con virtual users `VUs` vacíos). Siempre debes proteger el código con el bloque `thresholds`.

**Plantilla base requerida:**
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  // Simulación de tráfico en forma de campana (Realista)
  stages: [
    { duration: '1m', target: 50 }, // Ramp-up
    { duration: '3m', target: 50 }, // Peak Load
    { duration: '1m', target: 0 },  // Ramp-down
  ],
  // El verdadero valor Senior: Pipeline Breakers
  thresholds: {
    // El 95% de las llamadas debe ser menor a 250ms
    http_req_duration: ['p(95)<250'], 
    // La tasa de errores HTTP 500/400 debe ser menor al 1%
    http_req_failed: ['rate<0.01'],   
    // Evaluar latencia de validación TLS (Opcional en entornos seguros)
    http_req_tls_handshaking: ['p(99)<100']
  },
};

export default function () {
  const res = http.get('https://api.empresa.com/users');
  check(res, { 'El status fue 200': (r) => r.status == 200 });
  // Empular tiempo de pensamiento del usuario
  sleep(Math.random() * 2 + 1); 
}
```

## 2. Patrón: Chaos Engineering Integrado
Si el escenario del usuario involucra alta disponibilidad, inyectale *xk6-disruptor*.
```javascript
import { PodDisruptor } from 'k6/x/disruptor';

// Esto perturba la infraestructura MIENTRAS ocurre la prueba de carga
export function setup() {
  const disruptor = new PodDisruptor({
    namespace: 'prod-namespace',
    select: { labels: { app: 'payments-microservice' } },
  });
  // Inyectar 50ms de lag simulando daño de red o reinicio de Pods
  disruptor.injectHTTPFaults({ delay: '50ms' }, '30s');
}
```

## 3. Asistencia Analítica (Generación y RCA por IA)
1. **Mapeo de Tráfico:** Si el usuario te da analíticas de ayer ("Tuvimos 20,000 logins distribuidos en 1 hora, pero un pico de 5,000 en 3 minutos"), tú (la IA) debes hacer la matemática y generar el bloque de `scenarios` exacto para recrearlo.
2. **Root Cause Analysis (RCA):** Si el usuario te pega los resultados de una consola de k6 que falló, tu deber no es decirle "El test no pasó". Tu deber es cruzar el `http_req_duration` frente al `data_received` e `iteration_duration` e inferir si el cuello de botella fue la Base de Datos (Time To First Byte muy alto), el ancho de banda, o un Memory Leak de la aplicación.

# Cómo invocar esta Skill
El usuario deberá escribir comandos de alto nivel:
- *"Genera un script de k6 para el checkout con thresholds que rompan el pipeline si pasamos de 300ms, y genera una carga tipo Spike (Pico repentino)."*
- *"Usa k6-disruptor. Recrea una latencia del 20% en el backend de facturación mientras 100 usuarios intentan descargar facturas."*
- *"Tengo este reporte de salida de k6 donde fallaron los thresholds, actívate como analista RCA y dime dónde crees que se ahogó el servidor."*
