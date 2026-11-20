---
name: dashboard-qa-integridad-datos
description: Verifica que las cifras de un dashboard sean correctas: reconciliación con la fuente de verdad, agregaciones, zonas horarias, redondeo y frescura, con trazabilidad a ISO/IEC 25012 y 25010.
---

# QA de integridad y exactitud de datos

Un dashboard puede funcionar perfectamente y aun así mostrar cifras equivocadas. Esta skill comprueba que cada número coincide con la fuente de verdad, de forma independiente y con evidencia.

## Alcance
Incluye: definición de cada métrica, reconciliación con la fuente, agregaciones, cortes de tiempo y zonas horarias, redondeo y formato numérico, monedas, frescura de los datos, coherencia entre widgets, pantallas, API y exportaciones, y tratamiento de datos borrados, duplicados o tardíos.

Fuera de alcance: usabilidad, seguridad y rendimiento. Si una consulta tarda mucho o ves un dato de otro usuario, anótalo igual como hallazgo incidental.

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números y nombres de características o controles con descripciones propias; nunca copies el texto de las normas.

- **ISO/IEC 25012, modelo de calidad de datos** (norma principal de esta skill). Etiqueta cada hallazgo con una de estas características:
  - *Exactitud*: el valor representa correctamente la realidad (pasos 2, 3 y 9).
  - *Completitud*: no faltan registros ni atributos necesarios (pasos 3 y 8).
  - *Consistencia*: el mismo dato no se contradice entre lugares (paso 7).
  - *Credibilidad*: el usuario puede confiar en el origen del dato (paso 1, fuente documentada).
  - *Actualidad*: el dato tiene la antigüedad adecuada (paso 6).
  - *Precisión*: el nivel de detalle y los decimales son adecuados (paso 5).
  - *Trazabilidad*: se puede saber de dónde sale y quién cambió un dato (pasos 1 y 8).
  - *Comprensibilidad*: definiciones, unidades y símbolos claros (pasos 1 y 5).
- **ISO/IEC 25010:2023**: *Adecuación funcional / Corrección funcional* para los cálculos, y *Fiabilidad / Ausencia de fallos* para errores sistemáticos.
- **ISO/IEC 27001:2022, Anexo A**:
  - 8.31, separación de entornos, y 8.33, información de prueba: reglas de trabajo.
  - 8.3, restricción del acceso a la información: las cifras respetan el alcance de datos del rol.
  - 8.16, actividades de monitorización: evidencia de que los fallos de carga (ETL) se detectan.
  - 5.33, protección de registros: cuando el dashboard es fuente de informes oficiales que deben conservarse.
- **Normas de formato**: ISO 8601 para fechas y horas, e ISO 4217 para códigos de moneda.

## Antes de empezar
Reúne lo siguiente. Si falta algo, pregúntalo una sola vez; si nadie responde, continúa con supuestos y anótalos.
1. **Entorno**: URL de staging o QA. Si es producción, pide autorización explícita y usa solo lecturas.
2. **Fuente de verdad**: acceso de solo lectura a la base de datos, el data warehouse o una API independiente del dashboard. Sin ella, la reconciliación se limita a coherencia interna (pantalla vs. exportación vs. API del propio dashboard); dilo en el informe.
3. **Definiciones**: documentación de KPIs, consultas originales o código de las métricas, si existen.
4. **Contexto temporal**: zona horaria de la organización, del servidor y de la base de datos; horario del ETL o de la sincronización.
5. **Usuarios de prueba**: por rol, si las cifras cambian según el rol.
6. **Tolerancia aceptable**: por ejemplo, 0 para conteos y ±0,01 para importes. Si nadie la define, usa diferencia exacta para conteos y ±1 unidad del último decimal mostrado para valores redondeados, y anótalo.

## Reglas de trabajo
- Solo consultas de lectura en la fuente de verdad. Nunca `UPDATE`, `DELETE` ni DDL.
- Si creas datos de control, hazlo solo en staging (8.31), con el prefijo `QA_AAAAMMDD_` y sin datos personales reales (8.33), y documenta cómo limpiarlos.
- Calcula los valores esperados con código o SQL, nunca de cabeza. Guarda cada consulta usada como evidencia.
- Registra siempre los filtros, el periodo, la zona horaria y la hora exacta de cada comparación: sin eso, una diferencia no es reproducible.
- No supongas la causa de una diferencia sin evidencia: describe la diferencia y propone hipótesis marcadas como tales.

## Paso 1: Diccionario de métricas
Para cada número visible, completa:

| Métrica | Dónde aparece | Definición en lenguaje de negocio | Fórmula | Fuente (tabla o endpoint) | Filtros implícitos (estado, tipo, excluir pruebas) | Granularidad | Zona horaria | Frescura esperada | Responsable del dato |

Los huecos de esta tabla ya son hallazgos de credibilidad, trazabilidad o comprensibilidad: una métrica sin definición acordada no se puede validar. Pregunta por las ambigüedades típicas: ¿"ventas" incluye impuestos, devoluciones o pedidos cancelados? ¿"usuarios activos" son únicos por día o por periodo?

## Paso 2: Reconciliación con la fuente
Para cada métrica prioritaria (empieza por las que se usan para decidir):
1. Fija un contexto concreto: periodo, filtros y rol.
2. Anota el valor mostrado y la hora.
3. Calcula el valor de forma independiente en la fuente, aplicando la definición del paso 1.
4. Compara con la tolerancia acordada.
5. Repite con al menos tres contextos: sin filtros, con un filtro típico y con un segmento pequeño (donde los errores se notan más).

Cuando haya diferencia, aísla la causa variando una cosa por vez: periodo, filtro, segmento o rango de fechas más corto, hasta encontrar el registro o el día que la explica.

## Paso 3: Agregaciones
- La suma de las partes es igual al total (por categoría, región o mes). Busca la categoría "Otros" o "Sin asignar": si falta, los segmentos no suman el total.
- Conteos: `COUNT` vs `COUNT DISTINCT`. Revisa el doble conteo por joins (un pedido con 3 líneas contado 3 veces).
- Promedios: los nulos excluidos o tratados como cero según la regla; promedios ponderados frente a promedios de promedios (error clásico al agregar porcentajes por región).
- Porcentajes y ratios: el denominador correcto, la división por cero mostrada como "—" o "N/A" y no como 0 ni como `Infinity`/`NaN`.
- Medianas y percentiles: calculados sobre los datos crudos, no sobre agregados.
- Valores acumulados (YTD, running totals): empiezan en la fecha correcta y no se reinician por error.
- Subtotales y totales en tablas paginadas: el total cubre todas las páginas, no solo la visible (o lo indica con claridad).

## Paso 4: Tiempo y zonas horarias
- Define qué es "hoy" para el negocio y compruébalo cerca de la medianoche en las zonas horarias del usuario, de la organización y de UTC. Un registro creado a las 23:30 locales puede caer en el día siguiente en UTC.
- Cortes de periodo: semana (lunes o domingo, semana ISO 8601), mes, trimestre y año fiscal si difiere del calendario.
- Cambios de horario (DST) en las zonas que lo usan: días de 23 o 25 horas.
- Fechas especiales: 29 de febrero, fin de mes (30 frente a 31), cambio de año.
- Comparativas (vs. periodo anterior, YoY): el periodo comparado tiene la misma longitud y está bien alineado. Por ejemplo, ¿cómo se compara el 29 de febrero? ¿El "mes anterior" de un 31 de marzo está bien calculado?
- Periodo en curso: se indica que está incompleto; las comparativas no enfrentan un mes parcial con uno completo sin advertirlo.
- Fechas de negocio frente a fechas técnicas: ¿se usa la fecha del pedido, la de pago o la de inserción en la base?
- En API y exportaciones, las fechas con hora deberían ir en formato ISO 8601 con zona horaria explícita.

## Paso 5: Redondeo, formato y moneda
- Redondeo al final del cálculo, no en pasos intermedios (lo contrario acumula error en totales).
- Porcentajes que deberían sumar 100 % y suman 99,9 % o 100,1 %: aceptable solo si está documentado.
- Número de decimales coherente en toda la interfaz para la misma métrica.
- Abreviaturas (1,2 M; 15 K) correctas y sin pérdida engañosa (1,04 M frente a 1,4 M).
- Moneda: código ISO 4217 y símbolo correctos; conversiones con el tipo de cambio de la fecha correcta (la del hecho o la del informe, según la regla) y la fuente del tipo de cambio indicada.
- Signos: devoluciones y créditos con el signo correcto; negativos que no se muestran como positivos.

## Paso 6: Frescura (actualidad)
- La marca "actualizado a las" existe y coincide con la última carga real.
- Prueba de latencia (solo en staging): crea o modifica un registro de control y mide cuánto tarda en aparecer en cada widget. Compáralo con la frescura esperada.
- Caché: dos usuarios o dos pestañas ven el mismo valor; tras refrescar se ve el dato nuevo.
- Fallo de ETL: si la carga falla, el dashboard avisa en lugar de mostrar datos viejos como actuales, y el fallo queda registrado o genera una alerta (27001, 8.16).

## Paso 7: Coherencia cruzada (consistencia)
Un mismo dato debe coincidir en todos los lugares donde aparece:
- El mismo KPI en distintos widgets y pantallas.
- El agregado frente a la suma de su drill-down.
- La pantalla frente a la respuesta de la API (inspecciona la red en el navegador).
- La pantalla frente a la exportación (CSV, Excel, PDF) con los mismos filtros.
- Las notificaciones o emails de resumen frente al dashboard.
- El dashboard frente a otros informes oficiales de la organización, si existen.

## Paso 8: Calidad de los datos subyacentes
- Registros de prueba o internos excluidos de las cifras de negocio.
- Registros con borrado lógico (soft delete) excluidos o incluidos según la regla.
- Duplicados en la fuente que inflan los conteos.
- Datos tardíos o cambios retroactivos: ¿se recalculan los periodos pasados? ¿Cambian cifras ya "cerradas" sin aviso ni registro de quién y cuándo (trazabilidad)?
- Nulos y valores por defecto (fecha 1970-01-01, importe 0) que distorsionan promedios.
- Registros huérfanos (sin categoría, sin región) que desaparecen de los desgloses.

## Paso 9: Dataset de control (si puedes crear datos en staging)
Es la prueba más concluyente. Crea un conjunto pequeño con resultados conocidos de antemano: por ejemplo, 10 registros que cubran cero, negativos, nulos, el límite de cada umbral, fechas en los bordes de periodo y un duplicado intencional. Calcula el resultado esperado de cada widget con ese dataset aislado (fíltralo por el prefijo QA si es posible) y verifica la coincidencia exacta.

## Severidad
- **Crítica**: una cifra usada para decidir (ingresos, KPIs de dirección, cumplimiento regulatorio) es incorrecta; la exportación oficial difiere de la fuente.
- **Alta**: error sistemático en un segmento, periodo o zona horaria; datos desactualizados mostrados como actuales.
- **Media**: inconsistencia de redondeo o formato que puede confundir; diferencia en un caso límite poco frecuente.
- **Baja**: diferencias dentro de la tolerancia pero mal explicadas; falta de documentación.

## Formato de cada hallazgo
```
ID: DAT-001
Título: <qué cifra difiere y dónde>
Severidad: Crítica | Alta | Media | Baja
Métrica: <nombre y ubicación>
ISO/IEC 25012: <característica, p. ej. Exactitud>
ISO/IEC 25010: <característica / subcaracterística, si aplica>
ISO/IEC 27001 Anexo A: <control, o "No aplica">
Contexto: <periodo, filtros, rol, zona horaria, hora de la comprobación>
Valor mostrado:
Valor esperado: <y cómo se obtuvo>
Diferencia: <absoluta y %>
Consulta o cálculo de referencia: <SQL o código>
Aislamiento: <qué variaste para localizar la diferencia>
Hipótesis de causa: <marcada como hipótesis>
Evidencia:
```

## Informe final
1. **Resumen ejecutivo**: cuántas métricas se reconciliaron, cuántas coinciden y cuáles no son confiables hoy.
2. **Diccionario de métricas** completado, con los huecos de definición señalados.
3. **Tabla de reconciliación**: métrica, contexto, valor mostrado, esperado, diferencia, resultado.
4. **Hallazgos** ordenados por severidad.
5. **Trazabilidad normativa**: tabla de características de ISO/IEC 25012 (exactitud, completitud, consistencia, credibilidad, actualidad, precisión, trazabilidad, comprensibilidad) con resultado Evaluada sin hallazgos / Con hallazgos (IDs) / No evaluada, y una métrica cuando sea posible (por ejemplo, métricas reconciliadas sin diferencia / total, o latencia medida frente a la esperada). Indica también los controles de ISO/IEC 27001 para los que el informe aporta evidencia.
6. **Supuestos y limitaciones**: fuente usada, tolerancias y lo que no se pudo verificar. Recuerda que el informe es evidencia y no una certificación.
7. **Recomendaciones**: por ejemplo, documentar definiciones o añadir pruebas automáticas de reconciliación.
