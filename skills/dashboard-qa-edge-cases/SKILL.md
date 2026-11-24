---
name: dashboard-qa-edge-cases
description: Prueba casos límite de datos e interfaz en un dashboard: estados vacíos, volúmenes masivos, valores extremos, fechas límite, caracteres especiales y fallos de red, con trazabilidad a ISO/IEC 25010.
---

# QA de casos límite (datos e interfaz)

Busca el punto donde el dashboard deja de comportarse bien cuando los datos, las entradas o la red se alejan de lo normal. El objetivo es que el sistema falle de forma controlada: con un mensaje claro, sin pantalla en blanco y sin corromper datos.

## Alcance
Incluye: estados vacíos, valores extremos, volúmenes grandes, fechas límite, texto y caracteres especiales, entradas inválidas, archivos, fallos de red y del servidor, tamaños de pantalla extremos e interacciones bruscas.

Fuera de alcance: medición formal de rendimiento, explotación de vulnerabilidades o seguridad de exportaciones como inyección de fórmulas en CSV/Excel (ver `dashboard-qa-seguridad-acceso`), pruebas profundas de concurrencia/idempotencia en red (ver `dashboard-qa-estado-concurrencia`) y auditoría de accesibilidad. Si algo de eso aparece (o si la otra skill no se ejecuta), anótalo como hallazgo incidental.

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números y nombres de características o controles con descripciones propias; nunca copies el texto de las normas.

- **ISO/IEC 25010:2023, modelo de calidad de producto**
  - *Fiabilidad*:
    - Tolerancia a fallos: el sistema sigue funcionando cuando falla una parte, por ejemplo un widget o un endpoint (paso 8).
    - Recuperabilidad: se recupera tras un corte de red o un error del servidor (paso 8).
    - Ausencia de fallos: no falla con datos o entradas poco comunes pero válidas (pasos 2, 4 y 5).
  - *Capacidad de interacción*:
    - Protección frente a errores del usuario: validaciones y entradas inválidas (pasos 6 y 7).
    - Autodescriptividad y asistencia al usuario: estados vacíos y mensajes de error útiles (pasos 1 y 8).
  - *Eficiencia de desempeño / Capacidad*: comportamiento con volúmenes grandes, solo de forma observacional (paso 3).
  - *Flexibilidad / Adaptabilidad*: pantallas y zoom extremos (paso 9).
- **ISO/IEC 27001:2022, Anexo A**:
  - 8.26, requisitos de seguridad de las aplicaciones: validación de entradas en el servidor (pasos 5, 6 y 7), sin explotar vulnerabilidades.
  - 8.6, gestión de la capacidad: evidencia de los límites de volumen (paso 3).
  - 8.31, separación de entornos, y 8.33, información de prueba: reglas de trabajo.
- **ISO 8601**: referencia para el formato de fechas y horas en el paso 4.

## Antes de empezar
Reúne lo siguiente. Si falta algo, pregúntalo una sola vez; si nadie responde, continúa con supuestos y anótalos.
1. **Entorno**: URL de staging o QA. Nunca producción: estas pruebas crean datos raros y fuerzan errores.
2. **Usuarios de prueba**: incluye una cuenta nueva sin datos y otra con muchos datos.
3. **Capacidad de crear datos**: interfaz, API, scripts de semilla o acceso a la base de staging. Pide un dataset grande si existe.
4. **Límites documentados**: longitudes máximas, tamaños de archivo, filas por exportación, rangos de fecha permitidos.
5. **Navegador con herramientas de desarrollo**, para simular red lenta, desconexión y bloqueo de peticiones.

## Reglas de trabajo
- Trabaja solo en staging o QA (8.31) y usa únicamente datos ficticios (8.33).
- Prefija todos los datos que crees con `QA_AAAAMMDD_` (o `QA_AAAAMMDD_EDGE_` si corres en paralelo con otras skills) y deja un registro de cómo borrarlos.
- Guarda capturas y salidas relevantes en `evidencia-edge/` junto al informe (sin credenciales ni tokens).
- No cargues volúmenes masivos sin avisar al equipo del entorno: puedes afectar a otras personas que lo usan.
- Para cada caso, define antes de ejecutarlo qué sería un comportamiento aceptable; así evitas justificar después lo que veas.
- Comprueba también la consola del navegador y las respuestas de red: un error silencioso en consola es un hallazgo aunque la interfaz parezca bien.
- Lo que no pudiste probar va como "No verificado" con el motivo.

## Paso 1: Estados vacíos
- Usuario nuevo sin ningún dato: cada pantalla muestra un estado vacío útil (explicación y siguiente acción), no gráficos rotos ni ceros engañosos.
- Filtros sin resultados: un mensaje claro y la opción de limpiar los filtros.
- Un solo widget vacío mientras los demás tienen datos.
- Diferenciar "no hay datos" de "el valor es 0" y de "no se pudo cargar": son tres estados distintos y no deben verse iguales.
- Búsqueda sin coincidencias.
- Tablas vacías: los encabezados se mantienen y la paginación no muestra "página 1 de 0".
- Exportar sin datos: un archivo vacío con encabezados o un aviso, nunca un error.

## Paso 2: Valores extremos y especiales
Prueba en campos, cálculos y gráficos:
- Exactamente un registro (gráficos de línea con un solo punto, promedios de uno).
- Cero, negativos y la mezcla de positivos y negativos en el mismo gráfico.
- Valores muy grandes (1e12, el máximo del tipo de dato) y muy pequeños (0,0001).
- Muchos decimales; números que se muestran en notación científica sin querer.
- Nulos y campos opcionales vacíos en cada columna.
- Todos los valores iguales (gráficos con eje plano, escalas min = max).
- Un valor atípico enorme que aplasta el resto del gráfico.
- Porcentajes por encima del 100 % o negativos cuando el negocio los permite.

## Paso 3: Volumen
Con datasets de 10 000, 100 000 y, si es posible, 1 000 000 de registros:
- Paginación: primera página, última página, salto directo a una página lejana, cambiar el tamaño de página.
- Ordenar y filtrar sobre el total, no solo sobre la página cargada.
- Gráficos con miles de puntos: siguen siendo legibles (agregación, muestreo) y no bloquean el navegador.
- Leyendas y selectores con cientos de categorías: scroll, búsqueda, agrupación en "Otros".
- Desplegables de filtro con miles de opciones.
- Exportación del máximo permitido y por encima del máximo (mensaje claro, no un timeout mudo).
- Selección masiva ("seleccionar todo") sobre miles de registros.

Anota los umbrales aproximados donde la experiencia se degrada.

## Paso 4: Fechas límite
- 29 de febrero, 31 de diciembre → 1 de enero, fin de mes de 30 y 31 días.
- Rango invertido (inicio posterior al fin): se impide o se corrige con aviso.
- Rango de un solo día; rango de varios años; rango sin límite.
- Fechas futuras y muy antiguas (1900, 1970-01-01).
- Escritura manual en los selectores: formatos inválidos, día 31 en meses de 30, texto.
- Navegador en otra zona horaria (cámbiala en el sistema o con las herramientas de desarrollo): la fecha mostrada y la guardada siguen siendo coherentes.
- Horas límite: 00:00, 23:59:59, cambios de horario de verano en zonas que lo usan.

## Paso 5: Texto y caracteres especiales
Introduce estos valores en nombres, descripciones, comentarios, búsquedas y filtros, y verifica cómo se guardan, se muestran, se buscan y se exportan:
- Acentos y ñ: `Ñandú Peña Ángel`.
- Emoji, incluidos compuestos: `👨‍👩‍👧 🇻🇪`.
- Alfabetos no latinos y escritura de derecha a izquierda: `数据 بيانات данные`.
- Cadenas muy largas sin espacios (300 caracteres seguidos): no rompen el diseño.
- Solo espacios, espacios al inicio y al final, cadena vacía.
- Comillas y caracteres de escape: `O'Brien "test" \ / ; --`.
- Marcado inofensivo como texto: `<b>negrita</b>` debe verse literal, no en negrita.
- Saltos de línea y tabulaciones en campos de una línea.
- Contenido que empieza por `=`, `+`, `-` o `@`: verifica que no rompa el guardado o el renderizado; la evaluación de seguridad por inyección de fórmulas en CSV o Excel corresponde a `dashboard-qa-seguridad-acceso` (repórtalo como incidental solo si `SEC` no se ejecuta).
- Caracteres invisibles (espacio de ancho cero) en nombres que parecen duplicados.

## Paso 6: Entradas inválidas
- Superar y quedar justo en la longitud máxima de cada campo.
- Letras en campos numéricos; separadores decimales de otro idioma (`1,5` frente a `1.5`); separadores de miles.
- Pegar texto con formato desde Word o Excel.
- Autocompletado del navegador que rellena campos inesperados.
- Manipular un campo deshabilitado o de solo lectura con las herramientas de desarrollo y enviarlo: el servidor debe rechazarlo.

## Paso 7: Archivos (si el dashboard permite subirlos)
- Archivo vacío, archivo en el límite de tamaño y por encima del límite.
- Extensión incorrecta y extensión correcta con contenido de otro tipo.
- Nombres con espacios, acentos, emoji o muy largos.
- CSV con codificación distinta (Latin-1 frente a UTF-8), separador `;`, columnas faltantes o de más, filas mal formadas.
- Subir el mismo archivo dos veces.

## Paso 8: Red y servidor
Con las herramientas de desarrollo del navegador:
- Red lenta (3G) y muy lenta: los estados de carga aparecen y nada queda a medias sin explicación.
- Desconexión total durante la carga, durante un envío de formulario y durante una exportación; luego reconexión.
- Bloquear una sola petición (la de un widget): ese widget muestra un error y el resto del dashboard sigue funcionando.
- Simular respuestas 401, 403, 404, 429, 500 y 503 (con request overrides o un proxy si está disponible): cada una tiene un mensaje adecuado y la 401 lleva a iniciar sesión sin perder el trabajo, si es posible.
- Respuesta lenta que termina en timeout: hay mensaje y opción de reintentar.
- Respuesta malformada o incompleta (JSON truncado): sin pantalla en blanco.
- Reintentos: un reenvío automático no duplica la acción.

## Paso 9: Pantalla e interacción
- Ventanas muy estrechas, muy anchas (ultra-wide) y zoom del navegador al 50 % y al 200 %.
- Doble clic y clics repetidos en botones de acción, filtros y paginación.
- Cambiar filtros rápidamente varias veces seguidas: el resultado final corresponde a la última selección.
- Cerrar un modal o cambiar de pantalla mientras algo carga.
- Recargar la página a mitad de un flujo.

## Severidad
- **Crítica**: pérdida o corrupción de datos; pantalla en blanco o caída total ante un caso realista; acción duplicada con efecto económico.
- **Alta**: un caso razonablemente frecuente rompe una función importante; un error se muestra como dato válido (por ejemplo, 0 en lugar de "sin datos").
- **Media**: degradación con alternativa; mensaje de error confuso.
- **Baja**: detalle visual en un caso muy raro.

## Formato de cada hallazgo
```
ID: EDGE-001
Título:
Severidad: Crítica | Alta | Media | Baja
Categoría: Vacío | Extremos | Volumen | Fechas | Texto | Entrada | Archivos | Red | Pantalla
ISO/IEC 25010: <característica / subcaracterística, p. ej. Fiabilidad / Tolerancia a fallos>
ISO/IEC 27001 Anexo A: <control, o "No aplica">
Entorno y rol: <URL, navegador, rol, condiciones de red>
Dato o condición usada: <valor exacto, tamaño del dataset, error simulado>
Pasos:
  1. ...
Comportamiento aceptable esperado:
Resultado obtenido:
Errores en consola o red:
Evidencia:
Frecuencia:
```

## Informe final
1. **Resumen ejecutivo**: robustez general y los 3 hallazgos más importantes.
2. **Matriz de cobertura**: categorías de los pasos 1 a 9 por pantalla, marcadas como OK / Falla / No verificado.
3. **Umbrales observados**: volumen o tamaño a partir del cual la experiencia se degrada.
4. **Hallazgos** ordenados por severidad.
5. **Trazabilidad normativa**: por cada subcaracterística de ISO/IEC 25010 evaluada (tolerancia a fallos, recuperabilidad, ausencia de fallos, protección frente a errores del usuario, autodescriptividad, asistencia al usuario, adaptabilidad), indica Evaluada sin hallazgos / Con hallazgos (IDs) / No evaluada, con una métrica cuando sea posible (por ejemplo, fallos simulados con recuperación correcta / total). Añade los controles de ISO/IEC 27001 para los que el informe aporta evidencia.
6. **Datos de prueba creados** y cómo limpiarlos.
7. **Supuestos y limitaciones.** Recuerda que el informe es evidencia y no una certificación.
