---
name: dashboard-qa-mantenibilidad
description: Evalúa la mantenibilidad del código de un dashboard: modularidad, reutilización, complejidad, facilidad de cambio, pruebas automáticas, pipeline y dependencias, con trazabilidad a ISO/IEC 25010 y 27001.
---

# QA de mantenibilidad del código

Evalúa qué tan fácil y seguro es entender, cambiar y probar el código del dashboard. Un dashboard difícil de mantener acumula errores en cada cambio: un KPI nuevo rompe otro, un filtro se arregla en un widget y sigue roto en otro, y nadie se atreve a actualizar dependencias.

## Alcance
Incluye: estructura y modularidad, reutilización de componentes, complejidad y legibilidad, facilidad para hacer cambios típicos, pruebas automáticas, pipeline de integración y despliegue, dependencias y documentación técnica.

Fuera de alcance: probar el comportamiento del dashboard en ejecución (funcionalidad, rendimiento, seguridad dinámica, accesibilidad). Si al leer el código encuentras un defecto claro de esos temas, anótalo como hallazgo incidental.

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números y nombres de características o controles con descripciones propias; nunca copies el texto de las normas.

- **ISO/IEC 25010:2023**, característica *Mantenibilidad* (principal de esta skill):
  - Modularidad: los componentes son independientes y un cambio en uno afecta poco a los demás (paso 2).
  - Reusabilidad: las piezas comunes se reutilizan en vez de duplicarse (paso 3).
  - Analizabilidad: es fácil entender el código y diagnosticar fallos (paso 4).
  - Modificabilidad: se puede cambiar sin introducir defectos ni degradar la calidad (paso 5).
  - Capacidad de prueba: se pueden definir y ejecutar pruebas eficaces (paso 6).
- **ISO/IEC 25023**: si la organización la tiene, usa sus medidas de mantenibilidad para expresar los resultados.
- **ISO/IEC 27001:2022, Anexo A**:
  - 8.25, ciclo de vida de desarrollo seguro, y 8.28, codificación segura: prácticas y herramientas del paso 7.
  - 8.4, acceso al código fuente: permisos del repositorio y secretos (pasos 7 y 8).
  - 8.8, gestión de vulnerabilidades técnicas: dependencias vulnerables (paso 8).
  - 8.9, gestión de la configuración: configuración versionada y separada del código (paso 5).
  - 8.32, gestión de cambios: revisión de código, versionado y despliegues controlados (paso 7).
  - 5.37, procedimientos operativos documentados: documentación técnica y de operación (paso 4).

## Antes de empezar
Reúne lo siguiente. Si falta algo, pregúntalo una sola vez; si nadie responde, continúa con supuestos y anótalos.
1. **Acceso de solo lectura al código**: repositorio del frontend, del backend y de las consultas o transformaciones de datos del dashboard.
2. **Acceso al pipeline** (CI/CD) y, si existen, a los informes de análisis estático y de cobertura.
3. **Contexto del equipo**: tamaño, frecuencia de despliegues, cambios más habituales (nuevos KPIs, filtros, roles) y dolores conocidos.
4. **Estándares internos**: guías de estilo, convenciones o umbrales de calidad ya acordados. Si no existen, usa los valores por defecto de esta skill y márcalos como supuestos.

## Reglas de trabajo
- Solo lectura: no hagas commits, no abras pull requests y no cambies configuración del repositorio o del pipeline sin que te lo pidan.
- Ejecuta herramientas de análisis en una copia local; no ejecutes scripts del proyecto que modifiquen bases de datos o servicios.
- Si encuentras secretos (claves, contraseñas, tokens) en el código o en el historial, no los copies al informe: indica solo archivo, línea y tipo, y avisa de inmediato para que se roten.
- Las métricas automáticas orientan, pero no deciden solas: confirma cada hallazgo importante leyendo el código.
- Señala también lo que está bien hecho, para que no se pierda en una refactorización.
- Guarda salidas de linters, cobertura y análisis estático relevantes en `evidencia-mant/` junto al informe (sin secretos).

## Valores de referencia por defecto
Úsalos solo si el equipo no tiene los suyos, y márcalos como supuestos en el informe:
- Complejidad ciclomática por función: alerta por encima de 10, grave por encima de 20.
- Tamaño de función: alerta por encima de 50 líneas. Tamaño de archivo: alerta por encima de 400 líneas.
- Código duplicado: alerta por encima del 5 % del total.
- Cobertura de pruebas: al menos 80 % en la lógica de cálculo de KPIs y en las reglas de permisos. La cobertura global importa menos que cubrir lo crítico.
- Dependencias: ninguna con vulnerabilidades críticas o altas conocidas; ninguna fuera de soporte.

## Herramientas sugeridas
Usa las que encajen con el stack: SonarQube o SonarCloud, ESLint o el linter del lenguaje, `lizard` o `radon` (complejidad), `jscpd` (duplicación), `dependency-cruiser` o `madge` (dependencias entre módulos y ciclos), `cloc` o `tokei` (tamaño), `npm outdated`, `npm audit`, `pip-audit` u OWASP Dependency-Check (dependencias), y un escáner de secretos como `gitleaks`.

## Paso 1: Inventario
- Stack técnico: lenguajes, frameworks, librería de gráficos, gestor de estado, base de datos, herramientas de datos (ETL, dbt).
- Estructura del repositorio y de cada módulo, con su tamaño (líneas de código por módulo).
- Dónde vive cada responsabilidad: cálculo de KPIs (frontend, backend, base de datos o ETL), reglas de permisos, formatos, textos.
- Mapa de dependencias principales entre módulos.

Una señal temprana: si el mismo KPI se calcula en varios lugares (por ejemplo, en el frontend y en una consulta), es probable que tarde o temprano den cifras distintas.

## Paso 2: Modularidad
- Separación de capas: presentación, lógica de negocio y acceso a datos. Busca lógica de negocio (fórmulas, reglas de permisos) dentro de componentes visuales.
- Dependencias cíclicas entre módulos.
- Acoplamiento: módulos que importan detalles internos de otros; cambios pequeños que obligan a tocar muchos archivos.
- Widgets independientes: cada widget se puede añadir, quitar o modificar sin afectar a los demás.
- Estado global usado en exceso donde bastaría estado local.

## Paso 3: Reusabilidad
- Componentes compartidos para filtros, selectores de fecha, tablas, gráficos, tarjetas de KPI, estados vacíos y mensajes de error. Señala implementaciones duplicadas con comportamiento ligeramente distinto (fuente frecuente de inconsistencias).
- Funciones comunes de formato (números, monedas, fechas, zonas horarias) centralizadas en un solo lugar.
- Sistema de diseño o librería de componentes y su grado de uso real.
- Porcentaje de código duplicado y los bloques duplicados más grandes.

## Paso 4: Analizabilidad
- Complejidad ciclomática y cognitiva: lista las funciones que superan los umbrales, empezando por las de cálculo y permisos.
- Tamaño de funciones y archivos.
- Nombres claros y consistentes con la terminología del negocio.
- Tipado: uso de tipos estáticos, uso abusivo de tipos genéricos o sin tipo.
- Manejo de errores: errores silenciados, capturas genéricas que ocultan fallos.
- Observabilidad para diagnosticar: logs estructurados, identificadores de correlación, trazas, mensajes de error con contexto suficiente y sin datos sensibles.
- Documentación: README con cómo ejecutar el proyecto en local, decisiones de arquitectura (ADR), definiciones de KPIs cerca del código que los calcula, guía para añadir un widget o un KPI y procedimientos de operación (despliegue, recuperación).

## Paso 5: Modificabilidad
Haz ejercicios de cambio sobre el código, sin ejecutarlos, y cuenta qué habría que tocar:
- Añadir un KPI nuevo a un widget existente.
- Añadir un filtro global nuevo que afecte a todos los widgets.
- Añadir un rol nuevo con permisos intermedios.
- Cambiar un umbral de semáforo o un color corporativo.
- Añadir un idioma nuevo.

Para cada ejercicio anota: archivos y módulos afectados, si hay que tocar código o solo configuración, y riesgo de romper otra cosa.

Revisa además:
- Valores fijos en el código (umbrales, textos, colores, URL, IDs) que deberían ser configuración versionada y separada por entorno.
- Feature flags y su limpieza (flags viejos que nadie retira).
- Deuda técnica marcada (`TODO`, `FIXME`, `HACK`): cantidad, antigüedad y si alguna afecta a cálculos o permisos.
- Migraciones de base de datos versionadas y reversibles.

## Paso 6: Capacidad de prueba
- Existencia y tipo de pruebas: unitarias, de integración, de extremo a extremo y de regresión visual. Evalúa si la proporción es razonable.
- **Cobertura de lo crítico**: las fórmulas de KPIs, las reglas de permisos y los formatos regionales tienen pruebas con casos límite (cero, negativos, división por cero, fechas en el borde de periodo).
- Pruebas de permisos que verifiquen que cada rol ve solo lo suyo, no solo que el administrador ve todo.
- Datos de prueba: fixtures o datos semilla reproducibles y sin datos personales reales.
- El código permite aislar dependencias (inyección, mocks) sin trucos.
- El pipeline ejecuta las pruebas en cada cambio y bloquea la integración si fallan.
- Duración de la suite y pruebas intermitentes (flaky) conocidas.
- Facilidad para ejecutar el proyecto y las pruebas en local.

## Paso 7: Pipeline y proceso
- Linter, formateador y análisis estático integrados en el pipeline, con umbrales que bloquean.
- Análisis de seguridad del código (SAST) y de dependencias (SCA) automatizados (8.25, 8.28).
- Revisión de código obligatoria antes de integrar en la rama principal; ramas protegidas (8.32).
- Versionado, changelog y despliegues reproducibles, con posibilidad de volver a la versión anterior.
- Entornos separados (desarrollo, staging, producción) con configuración propia.
- Permisos del repositorio limitados a quien los necesita (8.4).

## Paso 8: Dependencias y secretos
- Dependencias desactualizadas: cuántas versiones mayores por detrás, especialmente framework, librería de gráficos y librerías de autenticación.
- Dependencias con vulnerabilidades conocidas, con su severidad (8.8).
- Dependencias abandonadas (sin mantenimiento) o fuera de soporte.
- Licencias incompatibles con el uso del producto.
- Herramienta de actualización automática (Dependabot, Renovate) configurada.
- Secretos en el código, en archivos de configuración o en el historial de git (8.4). Recuerda: solo ubicación y tipo en el informe.

## Severidad
- **Crítica**: secretos expuestos en el repositorio; dependencias con vulnerabilidades críticas explotables; lógica de cálculo o de permisos sin ninguna prueba y con historial de errores.
- **Alta**: un mismo KPI calculado en varios lugares sin fuente única; cambios típicos que requieren tocar muchos módulos con alto riesgo; pipeline que permite integrar código que no pasa las pruebas.
- **Media**: funciones o archivos que superan claramente los umbrales; duplicación relevante; documentación insuficiente para incorporar a una persona nueva.
- **Baja**: problemas de estilo o nombres sin impacto real en el mantenimiento.

## Formato de cada hallazgo
```
ID: MANT-001
Título:
Severidad: Crítica | Alta | Media | Baja
Categoría: Modularidad | Reusabilidad | Analizabilidad | Modificabilidad | Pruebas | Pipeline | Dependencias | Secretos | Documentación
ISO/IEC 25010: Mantenibilidad / <subcaracterística>
ISO/IEC 27001 Anexo A: <control, o "No aplica">
Ubicación: <repositorio, archivo, función o módulo; lista si se repite>
Evidencia: <métrica, fragmento breve de código o salida de la herramienta, sin secretos>
Impacto: <qué riesgo o costo genera en el mantenimiento>
Recomendación: <cambio concreto y esfuerzo estimado: bajo / medio / alto>
```

## Informe final
1. **Resumen ejecutivo**: estado de mantenibilidad, riesgos principales y fortalezas.
2. **Inventario técnico** del paso 1.
3. **Métricas clave** frente a los umbrales: complejidad, tamaño, duplicación, cobertura de lo crítico, dependencias desactualizadas y vulnerables.
4. **Ejercicios de cambio** del paso 5, con archivos afectados y riesgo.
5. **Hallazgos** ordenados por severidad.
6. **Trazabilidad normativa**: tabla de subcaracterísticas de mantenibilidad de ISO/IEC 25010 (modularidad, reusabilidad, analizabilidad, modificabilidad, capacidad de prueba) con resultado Evaluada sin hallazgos / Con hallazgos (IDs) / No evaluada y la métrica correspondiente; más los controles de ISO/IEC 27001 para los que el informe aporta evidencia. Recuerda que el informe es evidencia y no una certificación.
7. **Supuestos y limitaciones**: repositorios revisados, herramientas usadas y umbrales supuestos.
8. **Plan de mejora priorizado** por impacto y esfuerzo, empezando por lo que reduce errores en cifras y permisos.
