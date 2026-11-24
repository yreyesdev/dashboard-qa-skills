---
name: dashboard-qa-accesibilidad
description: Audita la accesibilidad de un dashboard según WCAG 2.2 AA: teclado, lectores de pantalla, gráficos, contraste, zoom, formularios y tiempos, con trazabilidad a ISO/IEC 25010 y normas de accesibilidad.
---

# QA de accesibilidad (WCAG 2.2 AA)

Comprueba que personas con discapacidad visual, motora, auditiva o cognitiva pueden usar el dashboard completo. Los dashboards tienen riesgos propios: gráficos que solo se entienden con la vista, colores como único código, datos que se actualizan solos y tablas complejas.

## Alcance
Incluye: revisión automática, navegación con teclado, lectores de pantalla, gráficos y visualizaciones (si aplica al producto), color y contraste, zoom y reflujo, formularios y errores, movimiento y límites de tiempo, contenido dinámico y documentos exportados. En aplicaciones transaccionales sin gráficos analíticos, marca el paso 4 como "No aplica".

Fuera de alcance: pruebas funcionales generales, rendimiento y seguridad. Si encuentras algo de eso por casualidad, anótalo.

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números y nombres de criterios o características con descripciones propias; nunca copies el texto de las normas.

- **WCAG 2.2, nivel AA** (W3C): referencia técnica principal; cada hallazgo se asocia a un criterio de conformidad.
- **ISO/IEC 40500**: es la versión ISO de WCAG 2.0. Si la organización exige "la norma ISO de accesibilidad", indica que WCAG 2.2 incluye y amplía esos criterios, y señala por separado los hallazgos en criterios que ya existían en 2.0.
- **ISO/IEC 25010:2023**, característica *Capacidad de interacción*:
  - Inclusividad: el dashboard es usable por personas con distintas capacidades (todos los pasos).
  - Operabilidad: se puede manejar con teclado, lector de pantalla y entradas alternativas (pasos 2, 3 y 6).
  - Protección frente a errores del usuario: formularios y confirmaciones (paso 7).
  - Autodescriptividad y asistencia al usuario: nombres, instrucciones y mensajes comprensibles (pasos 3 y 7).
  - Reconocibilidad de la adecuación: el usuario entiende qué muestra cada gráfico (paso 4).
- **ISO 9241-171** (ergonomía, accesibilidad del software): guía complementaria para recomendaciones de diseño.
- **EN 301 549**: si el dashboard se usa en el sector público europeo o en contrataciones que la exigen; sus requisitos web remiten a WCAG.
- **ISO/IEC 27001**: no aplica directamente a la accesibilidad, salvo 3.3.8 de WCAG (autenticación accesible), que conviene coordinar con 8.5 (autenticación segura).

## Antes de empezar
Reúne lo siguiente. Si falta algo, pregúntalo una sola vez; si nadie responde, continúa con supuestos y anótalos.
1. **Entorno** y **cuentas de prueba** por rol.
2. **Nivel objetivo**: WCAG 2.2 AA por defecto. Pregunta si aplica alguna norma o ley concreta (por ejemplo, EN 301 549, Sección 508, normativa local) o si hay usuarios con necesidades específicas conocidas.
3. **Pantallas prioritarias**: si son muchas, empieza por los recorridos principales y por una muestra representativa de cada tipo de componente.
4. **Herramientas**: un verificador automático (axe DevTools, Lighthouse, WAVE), un analizador de contraste y, si es posible, lectores de pantalla reales (NVDA con Firefox o Chrome en Windows, VoiceOver con Safari en macOS e iOS, TalkBack en Android).

## Reglas de trabajo
- Las herramientas automáticas detectan solo una parte de los problemas. Nunca declares una pantalla accesible solo porque pasó el análisis automático.
- Asocia cada hallazgo a un criterio WCAG concreto (número, nombre y nivel) y a una subcaracterística de ISO/IEC 25010.
- Describe el impacto en términos de la persona afectada ("un usuario de lector de pantalla no sabe qué filtro está activo"), no solo la regla técnica.
- Si no pudiste usar un lector de pantalla real, dilo y limita las conclusiones de esa parte a la inspección del código y del árbol de accesibilidad.
- Guarda salidas de herramientas automáticas, capturas y transcripciones relevantes en `evidencia-a11y/` junto al informe.

## Paso 1: Revisión automática
Pasa el verificador automático en cada pantalla y en cada estado relevante (con un modal abierto, con errores de formulario, con un panel desplegado, en estado vacío). Agrupa los resultados por tipo de problema y no repitas el mismo error cien veces: un hallazgo por causa, con una lista de ubicaciones.

## Paso 2: Teclado
Desconecta el ratón y completa los recorridos principales solo con Tab, Shift+Tab, Enter, Espacio, flechas y Esc.
- Todo lo que se hace con el ratón se puede hacer con el teclado (2.1.1), incluidos filtros, selectores de fecha, desplegables con selección múltiple, ordenar columnas, drill-down de gráficos, menús contextuales y arrastrar y soltar (que necesita una alternativa, 2.5.7).
- No hay trampas de foco (2.1.2).
- El orden del foco sigue el orden lógico de lectura (2.4.3).
- El foco es siempre visible y con buen contraste (2.4.7, 1.4.11) y no queda tapado por cabeceras fijas, banners o paneles (2.4.11).
- Hay un enlace para saltar al contenido principal (2.4.1).
- **Modales**: el foco entra en el modal, queda contenido mientras está abierto, Esc lo cierra y el foco vuelve al elemento que lo abrió.
- **Tooltips**: aparecen también con el foco, se pueden descartar con Esc sin mover el foco y no desaparecen al pasar el puntero sobre ellos (1.4.13).
- Atajos de una sola tecla: se pueden desactivar o reasignar (2.1.4).

## Paso 3: Lector de pantalla
Recorre las pantallas principales con un lector de pantalla:
- Título de página descriptivo y que cambia al navegar (2.4.2); idioma de la página declarado (3.1.1).
- Regiones (landmarks) y jerarquía de encabezados coherente que permite moverse por el dashboard (1.3.1, 2.4.6).
- Botones con solo icono (filtrar, exportar, cerrar, menú) tienen nombre accesible (4.1.2).
- El nombre accesible incluye el texto visible (2.5.3).
- El estado de los controles se anuncia: filtro activo, pestaña seleccionada, panel expandido, orden de la columna (4.1.2).
- **Tablas de datos**: encabezados de fila y columna asociados, título o caption, y la paginación y el número de resultados anunciados.
- **Contenido dinámico** (4.1.3): al aplicar un filtro se anuncia el resultado ("128 resultados"); los mensajes de éxito y error se anuncian sin mover el foco; los estados de carga se comunican. El auto-refresh no debe generar anuncios constantes que hagan imposible usar la página.

## Paso 4: Gráficos y visualizaciones
Es el punto más débil de casi todos los dashboards.
- Cada gráfico tiene una alternativa textual: un título y un resumen de lo que muestra (tendencia, máximos, mínimos), además de acceso a los datos en forma de tabla (1.1.1, 1.3.1).
- Gráficos SVG con rol y nombre accesible adecuados; gráficos en canvas con una alternativa equivalente.
- La información no depende solo del color (1.4.1): series distinguibles también por patrones, formas de marcador, estilos de línea o etiquetas directas; semáforos rojo, amarillo y verde acompañados de iconos o texto.
- Contraste de 3:1 de las líneas, barras y sectores frente al fondo y, cuando es necesario para entenderlos, entre elementos adyacentes (1.4.11).
- Tooltips de datos accesibles con teclado y anunciados.
- Las interacciones del gráfico (zoom, ocultar series, drill-down) tienen alternativa con teclado.
- Simula los tipos de daltonismo (protanopía, deuteranopía, tritanopía) con las herramientas de desarrollo y comprueba que los gráficos se siguen entendiendo.

## Paso 5: Color y contraste
- Texto normal 4,5:1 y texto grande 3:1 (1.4.3), incluidos textos de ejes, leyendas, placeholders, textos deshabilitados que transmiten información, y textos sobre fondos de color o degradados.
- Componentes de interfaz y foco 3:1 (1.4.11): bordes de campos, interruptores, casillas, iconos con significado.
- Modo oscuro, si existe: repite las comprobaciones.
- Modo de alto contraste de Windows (forced colors): los elementos siguen siendo visibles y los iconos no desaparecen.

## Paso 6: Zoom, reflujo y espaciado
- Zoom de texto al 200 % sin pérdida de contenido ni de funciones (1.4.4).
- Reflujo a 320 px de ancho (equivalente a un zoom del 400 %) sin desplazamiento en dos direcciones, salvo en tablas y gráficos que lo necesitan por naturaleza (1.4.10).
- Espaciado de texto aumentado (interlineado, espacio entre letras, palabras y párrafos) sin cortes ni solapamientos (1.4.12).
- Funciona en orientación vertical y horizontal (1.3.4).
- Tamaño mínimo de los objetivos táctiles o de clic de 24 × 24 px, o con suficiente separación (2.5.8). Revisa iconos pequeños en tablas y gráficos.

## Paso 7: Formularios y errores
- Cada campo tiene una etiqueta visible y asociada (1.3.1, 3.3.2); los placeholders no sustituyen a las etiquetas.
- Se indica el formato esperado (por ejemplo, de fecha).
- Los errores se identifican en texto, junto al campo, y se anuncian (3.3.1); se sugiere cómo corregirlos (3.3.3).
- Acciones importantes (borrar, enviar datos financieros o legales) se pueden revisar, confirmar o deshacer (3.3.4).
- Campos de datos personales con el atributo de autocompletado correcto (1.3.5).
- No se pide volver a introducir datos ya dados en el mismo proceso (3.3.7).
- El inicio de sesión no exige pruebas cognitivas como recordar o transcribir sin alternativa (3.3.8); permite pegar contraseñas y usar gestores.
- Cambiar un valor en un control no provoca un cambio de contexto inesperado (3.2.2).

## Paso 8: Movimiento y tiempo
- **Auto-refresh y contenido que se mueve o se actualiza solo**: se puede pausar, detener u ocultar (2.2.2).
- **Límite de tiempo de sesión**: se avisa antes y se puede extender (2.2.1).
- Animaciones: respetan la preferencia de reducir movimiento del sistema; nada parpadea más de 3 veces por segundo (2.3.1).
- Ayuda y contacto, si existen, están en el mismo lugar en todas las pantallas (3.2.6).

## Paso 9: Documentos exportados
- PDF exportados etiquetados (estructura, orden de lectura, texto alternativo de gráficos, idioma).
- Excel y CSV con fila de encabezados, sin celdas combinadas innecesarias y sin información transmitida solo por color.

## Severidad
- **Crítica**: impide completar un recorrido principal a un grupo de usuarios (por ejemplo, un filtro o un botón de guardar imposible de usar con teclado, o un modal que atrapa el foco).
- **Alta**: información esencial inaccesible (gráficos sin alternativa, estados solo por color en KPIs de decisión), errores de formulario no anunciados.
- **Media**: dificultad importante con alternativa, contraste insuficiente en elementos secundarios.
- **Baja**: problemas menores que no impiden la tarea.

## Formato de cada hallazgo
```
ID: A11Y-001
Título:
Severidad: Crítica | Alta | Media | Baja
Criterio WCAG: <número, nombre, nivel y versión en que se introdujo — p. ej. 1.4.1 Uso del color (A, WCAG 2.0)>
ISO/IEC 25010: Capacidad de interacción / <subcaracterística>
Ubicación: <pantalla, componente, estado; lista de ocurrencias si se repite>
Usuarios afectados: <lector de pantalla, solo teclado, baja visión, daltonismo, cognitivo, motor>
Cómo se detectó: <herramienta automática, teclado, NVDA + Chrome, etc.>
Pasos:
  1. ...
Resultado esperado:
Resultado obtenido:
Evidencia: <captura, transcripción del lector de pantalla, valores de contraste>
Recomendación: <corrección concreta>
```

## Informe final
1. **Resumen ejecutivo**: nivel de conformidad aproximado con WCAG 2.2 AA y las barreras que bloquean tareas.
2. **Tabla de criterios WCAG**: cumple, no cumple, no aplica o no verificado, con referencia a los hallazgos. Marca qué criterios forman parte de WCAG 2.0 (equivalente a ISO/IEC 40500) y cuáles se añadieron en 2.1 y 2.2.
3. **Hallazgos** ordenados por severidad.
4. **Trazabilidad normativa**: tabla de subcaracterísticas de capacidad de interacción de ISO/IEC 25010 (inclusividad, operabilidad, protección frente a errores del usuario, autodescriptividad, asistencia al usuario, reconocibilidad de la adecuación) con resultado Evaluada sin hallazgos / Con hallazgos (IDs) / No evaluada; y, si aplica, la correspondencia con EN 301 549. Recuerda que el informe es evidencia y no una declaración de conformidad formal.
5. **Herramientas, navegadores y lectores de pantalla usados**, y límites de la revisión.
6. **Recomendaciones priorizadas**, empezando por los componentes compartidos (filtros, gráficos, tablas, modales) porque corregirlos arregla muchas pantallas a la vez.
