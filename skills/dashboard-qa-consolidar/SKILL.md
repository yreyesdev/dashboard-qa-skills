---
name: dashboard-qa-consolidar
description: Orquesta la ejecución paralela o secuencial de las skills dashboard-qa-* y consolida sus informes en un único documento deduplicado por causa raíz, con trazabilidad a ISO/IEC 25010, 25012, 27001 y 29119.
---

# Orquestación y consolidación de QA

Coordina varias skills `dashboard-qa-*` sobre un mismo producto sin que se pisen entre sí, y une sus informes individuales en un único informe consolidado deduplicado por causa raíz.

## Alcance
Incluye: preparación del entorno para auditorías paralelas o secuenciales, asignación de prefijos aislados de datos de prueba, estandarización de carpetas de salida y evidencias, deduplicación de hallazgos repetidos en causas raíz (`R1`, `R2`, ...) y armado del informe consolidado usando `plantillas/informe-consolidado.md`.

Fuera de alcance: ejecutar las pruebas de cada skill por cuenta propia o modificar el código del producto evaluado.

## Referencias normativas
Esta skill consolida evidencia técnica producida por las demás skills; no certifica el cumplimiento de ninguna norma.

- **ISO/IEC/IEEE 29119** (proceso y documentación de pruebas): estructura del informe consolidado de finalización de pruebas.
- **ISO/IEC 25010:2023**, **ISO/IEC 25012**, **ISO/IEC 27001:2022 Anexo A** e **ISO/IEC 42001**: agregación de las tablas de trazabilidad individuales en una vista ejecutiva única.

## Fase 1: Antes de lanzar varias skills (orquestación)

Si se van a ejecutar varias skills `dashboard-qa-*` (en paralelo o en secuencia) sobre un mismo entorno:

1. **Carpeta de salida única**: define un directorio para la corrida, por ejemplo `docs/qa-<producto>-AAAAMMDD/`, con esta estructura:
   ```
   docs/qa-<producto>-AAAAMMDD/
   ├── informe-consolidado.md
   ├── funcional.md
   ├── integridad-datos.md
   ├── edge-cases.md
   ├── seguridad-acceso.md
   ├── asistente-ia.md          # si aplica
   ├── estado-concurrencia.md
   ├── rendimiento.md
   ├── accesibilidad.md
   ├── compatibilidad-i18n.md
   ├── mantenibilidad.md
   ├── evidencia-fun/
   ├── evidencia-dat/
   ├── evidencia-edge/
   ├── evidencia-sec/
   ├── evidencia-ia/            # si aplica
   ├── evidencia-est/
   ├── evidencia-perf/
   ├── evidencia-a11y/
   ├── evidencia-comp/
   └── evidencia-mant/
   ```
2. **Aislamiento de datos de prueba**: asigna a cada skill el prefijo `QA_AAAAMMDD_<PREFIJO>_` (por ejemplo `QA_20261124_FUN_`, `QA_20261124_EST_`) para que ninguna skill modifique o borre datos creados por otra.
3. **Aislamiento de carga (`PERF`)**: no ejecutes `dashboard-qa-rendimiento` (paso 4: carga, estrés o resistencia) al mismo tiempo que las demás skills sobre la misma instancia. La carga artificial provoca timeouts, falsos positivos y degrada la recolección de métricas en `FUN`, `DAT`, `EDGE`, `EST`, `A11Y` y `COMP`. Ejecuta `PERF` en una ventana separada o al final.
4. **Habilitación condicional**: omite las skills que no aplican al producto (por ejemplo `dashboard-qa-asistente-ia` si no hay asistente de IA) y déjalo explícito en el informe consolidado. En aplicaciones web de gestión sin gráficos analíticos, indica a las skills que marquen los pasos de gráficos/KPIs como "No aplica".
5. **Autorización explícita**: confirma autorización antes de incluir `SEC`, `IA` o el paso 4 de `PERF`.

## Fase 2: Consolidación de informes

Cuando todas las skills aplicables hayan entregado su informe y su carpeta `evidencia-<prefijo>/`:

### Paso 1: Verificar integridad de los insumos
- Comprueba que cada informe esperado existe y contiene sus secciones obligatorias (resumen, hallazgos con IDs `PREFIJO-NNN`, tabla de trazabilidad y datos de prueba creados).
- Verifica que no se filtraron credenciales, tokens completos ni datos personales reales en los textos ni en las carpetas `evidencia-*`.
- Si una skill no pudo completar parte de su alcance, conserva sus ítems como "No verificado" y documéntalo en la sección 7 del consolidado.

### Paso 2: Deduplicación por causa raíz
Varias skills suelen encontrar el mismo defecto técnico desde ángulos distintos (por ejemplo, un cálculo de fecha en UTC falla en `DAT`, `COMP`, `FUN` y `EDGE`).
1. Revisa todos los hallazgos y agrupa los que comparten exactamente la misma causa técnica subyacente bajo un identificador de causa raíz (`R1`, `R2`, ...).
2. Conserva siempre los IDs originales de cada skill y enlaza cada causa raíz con todos los IDs relacionados.
3. Asigna a la causa raíz la severidad más alta justificada por su impacto real combinado.
4. Calcula dos cifras claras en la tabla de severidades: **total bruto (con duplicados)** y **problemas únicos tras deduplicar**.

### Paso 3: Selección de los 5 hallazgos más importantes
Elige los 5 problemas (o causas raíz) de mayor impacto de negocio, seguridad o integridad de datos. Para cada uno redacta el "Por qué importa" en lenguaje claro para dirección y producto, sin jerga innecesaria.

### Paso 4: Agregación de trazabilidad normativa
Usa `plantillas/informe-consolidado.md` como base:
- **ISO/IEC 25010:2023**: resume el estado de cada una de las 9 características cruzando las tablas de trazabilidad de las skills que la evaluaron. Si una característica tiene hallazgos en una skill y pasó limpia en otra, su estado consolidado es `Con hallazgos`.
- **ISO/IEC 25012**: traslada la tabla de `DAT` con sus métricas observadas.
- **ISO/IEC 27001:2022, Anexo A**: consolida los controles probados por `SEC`, `IA`, `MANT`, `DAT`, `EST`, `PERF` y `FUN`, resolviendo contradicciones (si un control tiene hallazgos en una skill, queda `Con hallazgos` citando los IDs).

### Paso 5: Plan de acción y limpieza
1. Ordena el plan de acción por **impacto / severidad** y **esfuerzo** (empezando por bloqueantes de producción o pérdida/corrupción de datos).
2. Agrupa en una sola lista todos los datos de prueba `QA_AAAAMMDD_*` reportados por las skills con las instrucciones para limpiarlos en staging/QA.
3. Emite una recomendación explícita de aptitud (apto / apto con condiciones / no apto) coherente con los hallazgos críticos y altos.

## Informe final
El resultado debe seguir la estructura de `plantillas/informe-consolidado.md` e incluir enlaces relativos a cada informe individual y a sus carpetas `evidencia-<prefijo>/`.
