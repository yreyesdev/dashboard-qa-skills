---
name: dashboard-qa-funcional
description: Prueba funcional completa de un dashboard web: recorridos por rol, widgets, filtros, formularios, cálculos y reglas de negocio, exportaciones y navegación, con trazabilidad a ISO/IEC 25010.
---

# QA funcional de dashboards

Verifica que cada elemento del dashboard hace lo que debe según las reglas de negocio, empezando por los recorridos principales de cada rol. Entrega una matriz de cobertura y hallazgos reproducibles.

## Alcance
Incluye: recorridos principales (happy path) por rol, widgets y gráficos (si aplica al producto), filtros, formularios, cálculos y reglas de negocio visibles, acciones (crear, editar, borrar, aprobar), exportaciones, navegación y mensajes al usuario.

Fuera de alcance: auditoría de seguridad, pruebas de carga, accesibilidad en profundidad, reconciliación exhaustiva contra la base de datos (ver `dashboard-qa-integridad-datos`) y pruebas profundas de concurrencia/carreras de red (ver `dashboard-qa-estado-concurrencia`). Si encuentras algo grave de estos temas por casualidad, regístralo igual como incidental. Si la aplicación es transaccional y no tiene gráficos analíticos, marca los puntos de gráficos del paso 3 como "No aplica".

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números de cláusula, característica o control con descripciones propias; nunca copies el texto de las normas. Si la organización tiene las normas, contrasta con el texto oficial.

- **ISO/IEC 25010:2023, modelo de calidad de producto**
  - *Adecuación funcional* (característica principal de esta skill):
    - Completitud funcional: están todas las funciones especificadas (pasos 1, 2 y 7).
    - Corrección funcional: los resultados son correctos (pasos 3, 4, 5, 6 y 8).
    - Pertinencia funcional: las funciones facilitan las tareas del usuario (paso 2).
  - *Capacidad de interacción*, solo en lo funcional: protección frente a errores del usuario (validaciones, confirmaciones) y autodescriptividad (mensajes y feedback, paso 9).
- **ISO/IEC 27001:2022, Anexo A** (solo como reglas de trabajo y evidencia de aceptación):
  - 8.29, pruebas en desarrollo y aceptación: el informe sirve como evidencia de las pruebas de aceptación.
  - 8.31, separación de entornos: trabajar en staging o QA.
  - 8.33, información de prueba: usar datos ficticios o protegidos.
- **ISO/IEC/IEEE 29119** (proceso y documentación de pruebas): la estructura del informe final sigue la idea de un informe de finalización de pruebas.

## Antes de empezar
Reúne lo siguiente. Si falta algo, pregúntalo una sola vez; si nadie responde, continúa con supuestos razonables y anótalos en el informe.
1. **Entorno**: URL de staging o QA. Si parece producción (dominio real, datos reales de clientes), detente y pide confirmación explícita.
2. **Usuarios de prueba**: al menos una cuenta por rol.
3. **Accesos disponibles**: navegador, API (documentación u OpenAPI), base de datos de solo lectura, logs. Anota cuáles tienes, porque condicionan lo que puedes verificar.
4. **Alcance**: módulos incluidos y excluidos.
5. **Reglas de negocio**: especificaciones, historias de usuario, criterios de aceptación y definiciones de KPIs. Sin ellas, deduce el comportamiento esperado de la propia interfaz y marca esos casos como "esperado inferido".

## Reglas de trabajo
- Trabaja solo en el entorno acordado (ISO/IEC 27001, 8.31).
- Crea tus datos de prueba con un prefijo identificable (`QA_AAAAMMDD_`, o `QA_AAAAMMDD_FUN_` si corres en paralelo con otras skills), sin datos personales reales (8.33), y no modifiques ni borres datos que no creaste.
- Guarda capturas y archivos de respaldo en una subcarpeta `evidencia-fun/` junto al informe, sin credenciales ni tokens completos.
- Cada hallazgo debe ser reproducible: anota rol, filtros activos, datos usados y pasos exactos.
- Nunca afirmes que algo funciona sin haberlo ejecutado. Lo que no pudiste probar va como "No verificado" con el motivo.
- Ante un fallo intermitente, repite el caso varias veces y anota la frecuencia (por ejemplo, 2 de 5).

## Paso 1: Inventario
Recorre el dashboard completo con el rol de mayor privilegio y construye esta tabla:

| Pantalla | Elemento | Tipo (widget / filtro / formulario / acción / export / enlace) | Roles que lo ven | Comportamiento esperado |

Repite con cada rol y anota diferencias de visibilidad respecto a lo especificado. Este inventario es la base de la matriz de cobertura y de la evaluación de completitud funcional.

## Paso 2: Recorridos principales por rol
Para cada rol, identifica de 3 a 7 tareas que justifican la existencia del dashboard (por ejemplo, "revisar ventas del mes y exportarlas" o "aprobar una solicitud pendiente"). Ejecútalas de principio a fin como un usuario real, sin atajos por URL. Verifica que cada paso da feedback claro y que el resultado final persiste: recarga la página y confírmalo. Anota pasos innecesarios o confusos como hallazgos de pertinencia funcional.

## Paso 3: Widgets y gráficos
Para cada widget:
- Carga con datos, muestra un estado de carga mientras espera y no se queda en un spinner infinito.
- Título, unidades, periodo y moneda visibles y correctos.
- Ejes, leyendas y etiquetas legibles; la leyenda coincide con las series.
- Los tooltips muestran el valor correcto del punto (compáralo con la tabla o la fuente si existe).
- Interacciones: ordenar, paginar, expandir, cambiar tipo de vista, ocultar series y drill-down. El drill-down abre el detalle del elemento correcto y conserva el contexto (filtros, periodo).
- Los indicadores comparativos (variación %, flechas, semáforos) usan la dirección y el color correctos. Por ejemplo, una subida de costos no debe verse en verde si la regla dice lo contrario.
- El refresco manual y el automático actualizan el dato y la marca "actualizado a las".

## Paso 4: Filtros
- Cada filtro por separado: el resultado cambia y es coherente con el valor elegido.
- Combinaciones: prueba pares de filtros relevantes (técnica pairwise si hay muchos). Verifica la lógica Y/O entre valores del mismo filtro y entre filtros distintos.
- Alcance: el filtro afecta a todos los widgets que debe afectar y a ninguno más. Anota los widgets que no lo respetan.
- Filtros dependientes (país → ciudad): las opciones hijas se actualizan y se limpia una selección que deja de ser válida.
- Rangos de fecha: cada preset (hoy, ayer, últimos 7 y 30 días, mes actual, mes anterior, año a la fecha) calcula el rango correcto. El rango personalizado incluye o excluye los extremos según la regla.
- Limpiar o restablecer vuelve exactamente al estado por defecto.
- Persistencia: según la especificación, los filtros se mantienen (o no) al recargar, al navegar y volver, y en la URL compartida.
- Búsqueda: coincidencias parciales, mayúsculas y minúsculas, acentos y resultados vacíos.
- Los filtros aplicados se muestran con claridad (chips, contador) y también se aplican a la exportación.

## Paso 5: Formularios
Para cada formulario:
- Campos obligatorios: enviar vacío muestra errores claros junto a cada campo.
- Validaciones de formato y rango (email, números, fechas, longitudes mínimas y máximas) en el cliente y también en el servidor. Si tienes acceso a la API, envía el mismo dato inválido directamente.
- Valores por defecto correctos; los campos condicionales aparecen y desaparecen según las reglas.
- Envío correcto: mensaje de éxito, el dato aparece donde corresponde y persiste al recargar.
- Edición: carga los valores actuales, guarda solo lo cambiado y no pisa campos ocultos.
- Cancelar o cerrar sin guardar descarta los cambios (y avisa de cambios sin guardar, si está especificado).
- Un doble clic en enviar no crea duplicados.
- Los errores del servidor se muestran sin perder lo que el usuario escribió.

## Paso 6: Cálculos y reglas de negocio
Es el paso de mayor valor y la evidencia principal de corrección funcional. Para cada métrica o regla:
1. Escribe la regla en una línea. Por ejemplo: "Margen = (Ingreso − Costo) / Ingreso, en %, redondeado a 1 decimal".
2. Diseña casos con resultado conocido: valor típico, cero, negativo, el límite exacto de cada umbral, justo por encima y justo por debajo (análisis de valores límite) y división por cero.
3. Calcula tú mismo el resultado esperado (con código si hay más de un par de operaciones) y compáralo con lo que muestra el dashboard.
4. Para reglas con ramas (tramos de comisión, semáforos, estados de aprobación), usa una tabla de decisión que cubra cada combinación.
5. Revisa las reglas que dependen del rol o del usuario (por ejemplo, un gerente ve los totales de su equipo).

Incluye la tabla de casos en el informe: entrada, esperado, obtenido y resultado.

## Paso 7: Acciones y flujos
- CRUD completo de cada entidad que el dashboard permita gestionar.
- Flujos de varios pasos (asistentes, aprobaciones): avanzar, retroceder, abandonar a mitad y retomar.
- Transiciones de estado: solo están disponibles las permitidas (por ejemplo, no se puede aprobar algo ya rechazado, si la regla lo dicta).
- Acciones masivas: seleccionar todo, selección parcial y selección a través de varias páginas.
- Efectos secundarios: notificaciones, emails, registro de actividad y actualización de contadores.
- Confirmación en acciones destructivas y opción de deshacer, si está especificada.

## Paso 8: Exportaciones e informes
- Cada formato (CSV, Excel, PDF, imagen) se descarga, abre sin errores y tiene un nombre de archivo útil.
- Contenido: respeta filtros, orden y columnas visibles; los encabezados son legibles y los totales coinciden con la pantalla.
- Tipos de dato correctos en Excel (números como números, fechas como fechas).
- Si hay informes programados o enviados por email, llegan con el contenido y el periodo correctos.

## Paso 9: Navegación y feedback
- Todos los enlaces, menús, breadcrumbs y pestañas llevan donde deben, sin enlaces rotos (404).
- El botón atrás del navegador se comporta de forma razonable.
- Hay estados de carga, éxito, error y vacío, con textos comprensibles (sin mensajes técnicos crudos).
- Textos, etiquetas y ayudas usan la terminología del negocio.

## Severidad
- **Crítica**: un recorrido principal no se puede completar; un cálculo muestra un valor incorrecto que se usa para decidir; pérdida de datos.
- **Alta**: función importante rota sin alternativa; un filtro devuelve datos equivocados sin que el usuario lo note.
- **Media**: fallo con alternativa razonable, o que afecta a un caso poco frecuente.
- **Baja**: texto, formato o detalle visual sin impacto en el uso.

## Formato de cada hallazgo
```
ID: FUN-001
Título: <qué falla, en una línea>
Severidad: Crítica | Alta | Media | Baja
Área: <pantalla / widget / filtro / formulario>
ISO/IEC 25010: <característica / subcaracterística, p. ej. Adecuación funcional / Corrección funcional>
ISO/IEC 27001 Anexo A: <control, o "No aplica">
Entorno y rol: <URL, navegador, rol>
Precondiciones: <datos, filtros activos>
Pasos:
  1. ...
Resultado esperado: <y su fuente: especificación o "inferido">
Resultado obtenido:
Evidencia: <captura, respuesta de API, archivo exportado>
Frecuencia: <siempre / x de N>
Recomendación: <opcional>
```

## Informe final
1. **Resumen ejecutivo** (3 a 5 líneas): estado general, número de hallazgos por severidad y los 3 más importantes.
2. **Matriz de cobertura**: elementos del inventario por rol, marcados como Probado OK / Falla / No verificado / Bloqueado.
3. **Tabla de casos de cálculo** del paso 6.
4. **Hallazgos** ordenados por severidad.
5. **Trazabilidad normativa**: por cada subcaracterística de ISO/IEC 25010 evaluada (completitud, corrección y pertinencia funcional; protección frente a errores del usuario; autodescriptividad), indica Evaluada sin hallazgos / Con hallazgos (IDs) / No evaluada. Añade una métrica simple por subcaracterística cuando sea posible, por ejemplo: funciones especificadas presentes / total, o casos de cálculo correctos / total.
6. **Supuestos y limitaciones**: qué se infirió, qué no se pudo probar y por qué. Recuerda que el informe es evidencia y no una certificación.
7. **Siguientes pasos** recomendados.
