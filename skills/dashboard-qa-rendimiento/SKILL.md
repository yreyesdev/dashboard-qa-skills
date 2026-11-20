---
name: dashboard-qa-rendimiento
description: Mide rendimiento y carga de un dashboard: Web Vitals, payloads, consultas lentas y N+1, carga, estrés y resistencia, exportaciones pesadas y memoria, con trazabilidad a ISO/IEC 25010 y 27001.
---

# QA de rendimiento y carga

Mide cuánto tarda el dashboard, qué lo hace lento y cuánto aguanta antes de degradarse. Cada conclusión debe apoyarse en una medición repetible, no en impresiones.

## Condiciones obligatorias
1. **Entorno**: uno representativo de producción (tamaño de datos, infraestructura). Si es más pequeño, dilo en el informe y no extrapoles sin advertirlo.
2. **Autorización para carga**: las pruebas de carga y estrés solo se lanzan con permiso del responsable del entorno, en una ventana acordada y nunca contra producción sin un acuerdo explícito.
3. **Observabilidad**: acceso a métricas del servidor y de la base de datos (APM, logs de consultas lentas, CPU, memoria, conexiones). Sin esto puedes medir el síntoma, pero no la causa; dilo.
4. **Usuarios de prueba**: varias cuentas, para no concentrar la carga en una sola sesión o caché.

## Alcance
Incluye: rendimiento percibido en el navegador, red y payloads, tiempos de la API, base de datos, caché, carga concurrente, estrés, resistencia (soak), picos, exportaciones y trabajos pesados, y consumo de memoria del cliente.

Fuera de alcance: exactitud de los datos, seguridad y accesibilidad. Si encuentras algo de eso por casualidad, anótalo.

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números y nombres de características o controles con descripciones propias; nunca copies el texto de las normas.

- **ISO/IEC 25010:2023, modelo de calidad de producto**
  - *Eficiencia de desempeño* (característica principal de esta skill):
    - Comportamiento temporal: tiempos de respuesta, latencia y throughput (pasos 2, 3 y 4).
    - Utilización de recursos: CPU, memoria, red y conexiones (pasos 2, 3, 5 y 6).
    - Capacidad: límites máximos de usuarios, datos y operaciones (pasos 4, 5 y 7).
  - *Fiabilidad*: disponibilidad bajo carga, tolerancia a fallos en picos y recuperabilidad cuando baja la carga (paso 4).
  - *Flexibilidad / Escalabilidad*: comportamiento al crecer usuarios y volumen de datos (pasos 4 y 7).
- **ISO/IEC 25023**: si la organización la tiene, usa sus medidas de eficiencia de desempeño para expresar los resultados (por ejemplo, tiempo medio de respuesta o tasa de utilización de recursos).
- **ISO/IEC 27001:2022, Anexo A**:
  - 8.6, gestión de la capacidad: el informe aporta evidencia de que la capacidad está medida y proyectada (pasos 4 y 7).
  - 8.16, actividades de monitorización: existen métricas y alertas de rendimiento (condición 3).
  - 8.14, redundancia de las instalaciones de procesamiento: comportamiento del autoescalado o de la redundancia bajo carga, si aplica (paso 4).
  - 8.31, separación de entornos: las pruebas de carga no afectan a producción.

## Paso 1: Presupuestos de rendimiento
Acuerda los objetivos con el usuario. Si no hay, propón estos valores por defecto y márcalos como supuestos:
- **Navegador** (percentil 75 de usuarios): LCP < 2,5 s, INP < 200 ms, CLS < 0,1.
- **Dashboard útil**: datos principales visibles en < 3 s con caché fría y < 1,5 s con caché caliente.
- **API**: p95 < 500 ms en endpoints simples y < 2 s en agregaciones pesadas; tasa de error < 1 % bajo carga esperada.
- **Filtros**: respuesta visible en < 1 s.
- **Exportación**: se define por número de filas (por ejemplo, 100 000 filas en < 30 s o en un proceso asíncrono).
- **Carga esperada**: usuarios concurrentes en el pico y crecimiento en 12 a 24 meses.

Cada presupuesto es un requisito medible de la subcaracterística correspondiente de ISO/IEC 25010; deja anotada esa relación.

## Paso 2: Rendimiento en el navegador
Para las pantallas principales, con caché fría y caliente, en un equipo de gama media simulado (CPU 4× más lenta) y en red Fast 4G y de escritorio:
- Lighthouse o el panel Performance: Web Vitals, tiempo hasta datos útiles y tareas largas (> 50 ms) que bloquean.
- **Cascada de peticiones** (pestaña Red): número total de peticiones, peticiones en serie que podrían ir en paralelo, peticiones duplicadas y peticiones que no se usan.
- **Payloads**: tamaño de cada respuesta. Es un problema común devolver decenas de miles de filas para mostrar diez o un gráfico de doce puntos; anótalo.
- **Recursos estáticos**: tamaño del JavaScript y del CSS, compresión (gzip o brotli), caché HTTP, fuentes e imágenes optimizadas, carga diferida de lo que no se ve.
- **Interacción**: tiempo al aplicar un filtro, cambiar de pestaña, ordenar una tabla grande o abrir un drill-down. Busca re-renderizados innecesarios (con el profiler del framework si está disponible).
- **Gráficos y tablas grandes**: miles de puntos o filas sin virtualización ni agregación.
- Repite cada medición al menos 3 veces y usa la mediana.

## Paso 3: API y servidor
Para cada endpoint que usa el dashboard:
- Tiempo p50, p95 y p99 con un solo usuario, con caché fría y caliente.
- Variación según los parámetros: rango de fechas de un día frente a dos años, sin filtros frente a filtros selectivos, organización pequeña frente a grande.
- **Paginación profunda**: página 1 frente a una página con offset muy alto (las consultas con `OFFSET` grande se degradan).
- **Consultas N+1**: cuenta las consultas a la base de datos por petición (en el APM o en los logs). Si crece con el número de filas, es N+1.
- **Consultas lentas**: revisa el log de consultas lentas y el plan de ejecución (`EXPLAIN`) de las más pesadas: escaneos completos, índices ausentes, ordenamientos en disco.
- **Caché**: tasa de aciertos; la caché se invalida correctamente y no se vacía entera con cada cambio.
- **Auto-refresh**: calcula la carga que genera. Por ejemplo, 500 usuarios con 6 widgets que se refrescan cada 30 s generan 100 peticiones por segundo solo en refrescos.

## Paso 4: Pruebas de carga
Usa una herramienta como k6, JMeter, Locust, Gatling o Artillery. Diseña un modelo de usuario realista: inicio de sesión, carga del dashboard, cambio de filtros, drill-down, exportación ocasional, con tiempos de reflexión entre acciones y una mezcla basada en el uso real si hay analítica.

Escenarios, en este orden y deteniéndote si el sistema se degrada gravemente:
1. **Línea base**: pocos usuarios, para tener referencia.
2. **Carga esperada**: el pico normal de usuarios concurrentes durante 15 a 30 minutos.
3. **Pico**: 2× o 3× la carga esperada.
4. **Estrés**: aumento gradual hasta encontrar el punto de quiebre. Anota el número de usuarios donde la latencia p95 supera el presupuesto y donde aparecen errores.
5. **Pico súbito (spike)**: un salto brusco, por ejemplo el lunes a primera hora o el cierre de mes.
6. **Resistencia (soak)**: la carga esperada durante 2 a 8 horas para detectar fugas de memoria, conexiones que no se liberan o colas que crecen.

Mide en cada escenario: throughput, latencia p50, p95 y p99, tasa de error por tipo, CPU, memoria, conexiones a base de datos y el uso del pool, longitud de las colas y comportamiento del autoescalado. Verifica que el sistema se degrada con elegancia (mensajes claros, sin pérdida de datos), que se recupera solo cuando baja la carga y que la monitorización generó las alertas esperadas (8.16).

Esqueleto mínimo de k6, para adaptar:
```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
export const options = {
  stages: [
    { duration: '5m', target: 50 },
    { duration: '20m', target: 50 },
    { duration: '5m', target: 0 },
  ],
  thresholds: { http_req_duration: ['p(95)<2000'], http_req_failed: ['rate<0.01'] },
};
export default function () {
  const res = http.get(`${__ENV.BASE_URL}/api/dashboard/summary?range=30d`, {
    headers: { Authorization: `Bearer ${__ENV.TOKEN}` },
  });
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(Math.random() * 5 + 3);
}
```

## Paso 5: Exportaciones y trabajos pesados
- Exporta el máximo permitido: tiempo, uso de memoria del servidor y si el archivo se genera por streaming o se carga entero en memoria.
- Varias exportaciones grandes simultáneas: no tumban el servidor ni degradan al resto de usuarios.
- Timeouts de proxy o balanceador (por ejemplo, 60 s) que cortan exportaciones síncronas largas.
- Informes programados que se ejecutan todos a la misma hora.

## Paso 6: Memoria del cliente
- Deja el dashboard abierto con auto-refresh durante 1 hora y toma capturas de memoria (heap snapshots) al principio y al final.
- Cambia filtros 50 veces seguidas y compara la memoria: si crece sin volver a bajar, hay una fuga (escuchas de eventos, gráficos que no se destruyen, suscripciones abiertas).

## Paso 7: Volumen de datos futuro
Si es posible, carga en staging el volumen proyectado a 12 o 24 meses y repite las mediciones clave de los pasos 2 y 3. Muchos dashboards son rápidos hoy y lentos al año siguiente. Esta proyección es la evidencia principal para la gestión de la capacidad (8.6).

## Severidad
- **Crítica**: el sistema cae o pierde datos bajo la carga esperada; un usuario o una exportación pueden degradar el servicio para todos.
- **Alta**: no se cumple el presupuesto en recorridos principales bajo carga esperada; fuga de memoria en resistencia; el punto de quiebre está por debajo del pico previsto.
- **Media**: incumplimiento en pantallas secundarias; payloads o peticiones claramente excesivos; consultas sin índice que todavía no duelen pero escalarán mal.
- **Baja**: optimizaciones recomendables con poco impacto actual.

## Formato de cada hallazgo
```
ID: PERF-001
Título:
Severidad: Crítica | Alta | Media | Baja
Categoría: Navegador | Red / payload | API | Base de datos | Caché | Carga | Estrés | Resistencia | Exportación | Memoria
ISO/IEC 25010: <característica / subcaracterística, p. ej. Eficiencia de desempeño / Comportamiento temporal>
ISO/IEC 27001 Anexo A: <control, o "No aplica">
Contexto: <pantalla o endpoint, parámetros, volumen de datos, usuarios concurrentes>
Métrica: <valor medido (mediana de N) frente al presupuesto>
Condiciones: <caché fría o caliente, red, CPU simulada>
Evidencia: <traza, captura de Lighthouse, plan de ejecución, gráfico de la prueba de carga>
Causa probable: <con la evidencia que la sostiene, marcada como hipótesis si no está confirmada>
Recomendación: <por ejemplo, índice, paginación en servidor, agregación previa, caché, virtualización>
```

## Informe final
1. **Resumen ejecutivo**: ¿aguanta la carga esperada y la de 12 meses? ¿Dónde está el punto de quiebre?
2. **Tabla de métricas frente a presupuesto** por pantalla y por endpoint.
3. **Resultados de cada escenario de carga**, con gráficos de latencia, throughput y errores en el tiempo.
4. **Hallazgos** ordenados por severidad, con su causa probable.
5. **Trazabilidad normativa**: tabla de subcaracterísticas de ISO/IEC 25010 (comportamiento temporal, utilización de recursos, capacidad, disponibilidad, escalabilidad) con el valor medido frente al presupuesto y el resultado Cumple / No cumple (IDs) / No evaluada; más los controles de ISO/IEC 27001 para los que el informe aporta evidencia. Recuerda que el informe es evidencia y no una certificación.
6. **Entorno, herramientas, scripts y limitaciones** (incluida la diferencia con producción).
7. **Recomendaciones priorizadas** por impacto y esfuerzo.
