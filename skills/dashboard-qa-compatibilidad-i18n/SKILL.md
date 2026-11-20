---
name: dashboard-qa-compatibilidad-i18n
description: Prueba compatibilidad e internacionalización de un dashboard: navegadores, dispositivos, responsive, traducciones, formatos locales, RTL y exportaciones, con trazabilidad a ISO/IEC 25010, ISO 8601 e ISO 4217.
---

# QA de compatibilidad e internacionalización

Comprueba que el dashboard funciona y se ve bien en los navegadores y dispositivos que usan las personas reales, y en cada idioma y región que soporta.

## Alcance
Incluye: matriz de navegadores y sistemas operativos, dispositivos y diseño responsive, particularidades de cada navegador, impresión y descargas, traducciones, formatos regionales de números, monedas, fechas y horas, ordenamiento y búsqueda por idioma, escritura de derecha a izquierda y exportaciones según la región.

Fuera de alcance: lógica de negocio, seguridad, rendimiento y auditoría de accesibilidad. Si encuentras algo de eso por casualidad, anótalo.

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números y nombres de características con descripciones propias; nunca copies el texto de las normas.

- **ISO/IEC 25010:2023, modelo de calidad de producto**
  - *Flexibilidad / Adaptabilidad*: funciona en distintos navegadores, dispositivos, resoluciones e idiomas (pasos 1, 2, 3, 4 y 7).
  - *Compatibilidad*:
    - Interoperabilidad: los archivos exportados se abren e interpretan bien en otras herramientas, como Excel, en cada región (paso 8).
    - Coexistencia: convive con extensiones, bloqueadores y otras aplicaciones del navegador (paso 3).
  - *Capacidad de interacción*:
    - Inclusividad: usable por personas de distintos idiomas y regiones (pasos 4 a 7).
    - Autodescriptividad: textos traducidos y comprensibles (paso 4).
  - *Adecuación funcional / Corrección funcional*: los formatos regionales no cambian el valor de los datos (paso 5).
- **Normas ISO de formato y códigos**:
  - ISO 8601: fechas y horas en API, exportaciones e intercambio de datos (pasos 5 y 8).
  - ISO 4217: códigos de moneda y su número de decimales (paso 5).
  - ISO 639 e ISO 3166: códigos de idioma y país en la configuración regional (por ejemplo, `es-VE`).
- **ISO/IEC 27001:2022, Anexo A**: en general no aplica a esta skill. Como regla de trabajo, 8.31 separación de entornos y 8.33 información de prueba; y 5.23, seguridad en servicios en la nube, si se usa un servicio de pruebas en la nube con datos de la organización.

## Antes de empezar
Reúne lo siguiente. Si falta algo, pregúntalo una sola vez; si nadie responde, continúa con supuestos y anótalos.
1. **Entorno** y **cuentas de prueba**, idealmente configuradas en distintos idiomas y regiones.
2. **Datos de uso reales** (analítica): navegadores, versiones, sistemas, resoluciones e idiomas más usados. Si no hay datos, usa la matriz por defecto del paso 1 y anótalo.
3. **Soporte oficial**: navegadores, dispositivos e idiomas que el producto promete soportar.
4. **Configuración regional**: si el formato depende del idioma del navegador, de una preferencia del perfil o de la organización, y qué zona horaria se usa para mostrar fechas.
5. **Acceso a navegadores y dispositivos**: reales, emuladores o un servicio de pruebas en la nube. Indica en el informe qué fue real y qué emulado.

## Reglas de trabajo
- Prioriza la matriz por uso real: el 95 % del tráfico primero, los casos raros después.
- Lo emulado no sustituye a lo real en Safari de iOS ni en pantallas táctiles; si no tuviste acceso, márcalo como "No verificado en dispositivo real".
- Usa capturas comparativas de la misma pantalla en cada combinación para detectar diferencias visuales.
- Si usas un servicio de pruebas en la nube, hazlo con datos ficticios (8.33).

## Paso 1: Matriz de compatibilidad
Si no hay analítica, usa esta matriz por defecto:
- **Navegadores**: Chrome, Edge, Firefox y Safari en sus dos últimas versiones principales.
- **Sistemas**: Windows 10 y 11, macOS reciente, iOS y Android recientes.
- **Resoluciones**: 1366 × 768, 1920 × 1080, 2560 × 1440, pantalla 4K con escala del 150 %, tablet vertical y horizontal, y móvil (360 a 430 px de ancho).

Construye la tabla de navegador × sistema × resolución y ejecuta en cada combinación prioritaria los recorridos principales: iniciar sesión, cargar el dashboard, filtrar, drill-down, editar y exportar.

## Paso 2: Diseño responsive
- Cada punto de corte: nada se superpone, se corta ni queda fuera de la pantalla.
- Tablas anchas en móvil: desplazamiento horizontal claro, columnas prioritarias o vista de tarjetas.
- Gráficos que se redimensionan y siguen siendo legibles (etiquetas de ejes, leyendas).
- Menús y filtros en móvil: panel lateral o desplegable usable.
- Interacciones que dependen del hover (tooltips, menús) tienen equivalente táctil.
- Objetivos táctiles de tamaño suficiente.
- El teclado virtual no tapa el campo que se está escribiendo.
- Áreas seguras de pantallas con muesca.
- Cambio de orientación sin perder el estado.

## Paso 3: Particularidades de navegadores
- Selectores nativos de fecha y hora (Safari y Firefox los muestran distinto).
- Diferencias de CSS: flexbox y grid, alturas de viewport en móvil, barras de desplazamiento, fuentes y su suavizado.
- Safari: bloqueo de cookies de terceros (afecta a inicio de sesión único o contenido embebido en iframes), restricciones de almacenamiento, descargas de archivos en iOS.
- Modo privado o almacenamiento bloqueado: la aplicación no se rompe si `localStorage` no está disponible.
- Bloqueadores de anuncios y extensiones de privacidad: a veces bloquean endpoints o scripts con nombres como `analytics`, `tracking` o `ads`; comprueba que el dashboard sigue funcionando (coexistencia).
- Portapapeles (botón copiar), pantalla completa y notificaciones según el navegador.
- **Impresión**: imprimir o guardar como PDF desde el navegador produce un resultado legible (estilos de impresión, gráficos visibles, sin cortes de tabla absurdos).

## Paso 4: Traducciones
Repite en cada idioma soportado:
- No quedan claves sin traducir (`dashboard.filters.title`) ni textos en el idioma original mezclados.
- Textos que no se tradujeron porque están dentro de imágenes, gráficos, mensajes de error del servidor, emails o exportaciones.
- **Expansión del texto**: el alemán o el francés pueden ocupar entre un 30 % y un 40 % más; botones, pestañas, encabezados de tabla y etiquetas de gráficos no se cortan ni se desbordan.
- **Plurales**: "0 resultados", "1 resultado", "2 resultados", y las reglas de plural de cada idioma.
- Frases construidas concatenando trozos que no funcionan en otros idiomas por el orden de las palabras o el género.
- Consistencia terminológica: el mismo concepto se traduce siempre igual.
- **Pseudolocalización**, si el equipo puede activarla: un idioma falso con caracteres acentuados y textos más largos revela rápido los textos fijos y los cortes.

## Paso 5: Formatos regionales
Para cada región soportada (por ejemplo, es-VE, es-ES, en-US, pt-BR):
- **Números**: separador decimal y de miles (1.234,56 frente a 1,234.56) en la pantalla, en los ejes de gráficos y en los tooltips.
- **Entrada de números**: si el usuario escribe `1,5` o `1.5`, se interpreta según su región; `1.234` no se convierte por error en 1,234.
- **Monedas** (ISO 4217): símbolo y código, posición del símbolo, número de decimales según la moneda (por ejemplo, el yen no tiene decimales y el dinar de Baréin tiene tres), y no se confunden monedas con el mismo símbolo ($). Cuando hay varias monedas, se muestra el código de tres letras.
- **Fechas**: orden día, mes y año (03/04 es 3 de abril o 4 de marzo según la región); nombres de meses y días traducidos; primer día de la semana (lunes o domingo) en calendarios y en los cortes semanales. Internamente, en la API y en las exportaciones técnicas, las fechas van en ISO 8601.
- **Horas**: formato de 12 o 24 horas; zona horaria mostrada cuando importa.
- **Zonas horarias**: el dashboard muestra la hora en la zona definida (usuario, organización o UTC) y lo indica. Cambia la zona horaria del sistema y comprueba que las fechas mostradas y guardadas siguen siendo correctas.
- **Porcentajes y unidades**: espacio antes del símbolo de porcentaje según la región, unidades del sistema métrico o imperial si aplica.

## Paso 6: Ordenamiento y búsqueda por idioma
- Orden alfabético correcto según el idioma: la ñ va después de la n en español; las vocales acentuadas no se van al final de la lista.
- La búsqueda encuentra resultados sin importar los acentos y las mayúsculas (`Peña` con `pena`, `José` con `jose`), si así está especificado.
- Nombres y datos en alfabetos no latinos se muestran con fuentes que tienen los caracteres (sin cuadros vacíos).

## Paso 7: Derecha a izquierda (si se soporta árabe, hebreo, etc.)
- La interfaz se invierte correctamente: menús, iconos direccionales, barras de progreso, paginación.
- Los gráficos y los números mantienen la dirección que corresponde.
- Mezcla de texto en ambas direcciones (un nombre en árabe con un número o una palabra en inglés) se muestra bien.

## Paso 8: Exportaciones según la región (interoperabilidad)
- **CSV**: separador adecuado (Excel en configuración española espera `;` porque `,` es el separador decimal), codificación UTF-8 con BOM para que Excel muestre bien los acentos, y saltos de línea dentro de las celdas.
- **Excel**: números y fechas guardados como tipos reales, no como texto; el formato visible respeta la región.
- **PDF**: fuentes que incluyen todos los caracteres del idioma; textos traducidos.
- **Formatos técnicos** (JSON, CSV para integraciones): fechas en ISO 8601 con zona horaria y monedas con código ISO 4217, independientemente del idioma del usuario.
- Nombres de archivo con caracteres no ASCII descargan bien en todos los navegadores.

## Paso 9: Comunicaciones
Emails, notificaciones e informes programados llegan en el idioma, el formato regional y la zona horaria del destinatario, no en los de quien los creó o los del servidor.

## Severidad
- **Crítica**: un recorrido principal no funciona en un navegador o dispositivo con una porción relevante de usuarios; un formato regional cambia el valor de una cifra (por ejemplo, 1.234 leído como 1,234) o de una fecha.
- **Alta**: pantallas inutilizables en móvil o en un idioma soportado; exportaciones ilegibles en Excel para una región.
- **Media**: textos sin traducir o cortados en elementos visibles; diferencias visuales notorias entre navegadores.
- **Baja**: detalles visuales menores en combinaciones poco usadas.

## Formato de cada hallazgo
```
ID: COMP-001
Título:
Severidad: Crítica | Alta | Media | Baja
Categoría: Navegador | Responsive | Dispositivo | Impresión / descargas | Traducción | Formato regional | Ordenamiento / búsqueda | RTL | Exportación | Comunicaciones
ISO/IEC 25010: <característica / subcaracterística, p. ej. Flexibilidad / Adaptabilidad>
Otra norma ISO: <ISO 8601, ISO 4217, ISO 639 / 3166, o "No aplica">
Combinación: <navegador y versión, sistema, dispositivo, resolución, idioma y región, zona horaria; real o emulado>
Pasos:
  1. ...
Resultado esperado:
Resultado obtenido:
Evidencia: <capturas comparativas>
Otras combinaciones afectadas o no afectadas:
Recomendación:
```

## Informe final
1. **Resumen ejecutivo**: porcentaje de usuarios cubiertos por combinaciones probadas sin fallos graves, y los problemas principales.
2. **Matriz de compatibilidad** con el resultado de cada combinación (OK / Falla / No verificado / Emulado).
3. **Matriz de idiomas y regiones** con el resultado por categoría (traducción, formatos, exportación).
4. **Hallazgos** ordenados por severidad.
5. **Trazabilidad normativa**: tabla de subcaracterísticas de ISO/IEC 25010 (adaptabilidad, interoperabilidad, coexistencia, inclusividad, autodescriptividad, corrección funcional) con resultado Evaluada sin hallazgos / Con hallazgos (IDs) / No evaluada, y conformidad observada con ISO 8601 e ISO 4217 en API y exportaciones. Recuerda que el informe es evidencia y no una certificación.
6. **Herramientas, dispositivos reales frente a emulados, y limitaciones.**
7. **Recomendaciones**: por ejemplo, centralizar el formato regional en una sola librería, añadir pseudolocalización al proceso de desarrollo, o pruebas de regresión visual en varios navegadores.
