# Informe consolidado de calidad del dashboard

> Plantilla para unir los informes de varias skills en un solo documento. Copia este archivo, reemplaza lo que está entre `< >` y borra las secciones de skills que no se ejecutaron.

## 1. Datos generales

| Campo | Valor |
|---|---|
| Producto | <nombre del dashboard y versión> |
| Entorno evaluado | <URL de staging o QA> |
| Periodo de evaluación | <fechas> |
| Responsable de la evaluación | <nombre o equipo> |
| Autorización para pruebas de seguridad | <quién la dio y cuándo, o "No aplica"> |
| Skills ejecutadas | <lista> |

## 2. Resumen ejecutivo

<3 a 6 líneas para dirección: estado general, riesgos principales y decisión recomendada (por ejemplo, apto para producción, apto con condiciones o no apto).>

### Hallazgos por severidad

| Skill | Crítica | Alta | Media | Baja | Total |
|---|---|---|---|---|---|
| Funcional (FUN) | | | | | |
| Integridad de datos (DAT) | | | | | |
| Casos límite (EDGE) | | | | | |
| Seguridad y control de acceso (SEC) | | | | | |
| Asistente de IA (IA) | | | | | |
| Estado y concurrencia (EST) | | | | | |
| Rendimiento (PERF) | | | | | |
| Accesibilidad (A11Y) | | | | | |
| Compatibilidad e idiomas (COMP) | | | | | |
| Mantenibilidad (MANT) | | | | | |
| **Total (con duplicados)** | | | | | |

> Si varias skills detectaron el mismo problema por caminos distintos, indica aquí el total de **problemas únicos tras deduplicar por causa raíz** (por ejemplo: `<N> problemas únicos tras agrupar en <M> causas raíz`).

### 2.1 Causas raíz (deduplicación)

| # | Causa raíz | Severidad consolidada | IDs relacionados |
|---|---|---|---|
| R1 | `<descripción técnica breve de la causa común>` | | |

### Los 5 hallazgos más importantes

| ID | Título | Severidad | Por qué importa |
|---|---|---|---|
| | | | |

## 3. Calidad del producto según ISO/IEC 25010:2023

Resume el resultado por característica a partir de las tablas de trazabilidad de cada skill.

| Característica | Estado | Hallazgos (IDs) | Skills que la evaluaron |
|---|---|---|---|
| Adecuación funcional | <Sin hallazgos / Con hallazgos / No evaluada> | | FUN, DAT, IA |
| Eficiencia de desempeño | | | PERF |
| Compatibilidad | | | COMP, EST |
| Capacidad de interacción | | | A11Y, EDGE, COMP |
| Fiabilidad | | | EDGE, EST, PERF |
| Seguridad | | | SEC, IA, EST |
| Mantenibilidad | | | MANT |
| Flexibilidad | | | COMP, PERF |
| Protección (safety) | <normalmente "No aplica"> | | — |

## 4. Calidad de los datos según ISO/IEC 25012

<Copia la tabla de trazabilidad del informe de integridad de datos.>

## 5. Evidencia para ISO/IEC 27001:2022, Anexo A

| Control | Resultado | Hallazgos (IDs) | Skill |
|---|---|---|---|
| | <Evidencia de funcionamiento correcto / Con hallazgos / No evaluado> | | |

> Esta tabla es evidencia técnica para la auditoría interna del sistema de gestión. No equivale a una certificación ni a una evaluación completa de los controles.

## 6. Hallazgos detallados

<Pega aquí los hallazgos de cada skill en su formato original, agrupados por skill y ordenados por severidad, o enlaza a los informes individuales (`funcional.md`, `integridad-datos.md`, etc.) y a sus carpetas `evidencia-<prefijo>/`. Si un mismo problema aparece en varias skills, déjalo en una sola (o en la sección 2.1) y cita los demás IDs.>

## 7. Supuestos, limitaciones y lo que no se evaluó

<Une las secciones de supuestos y limitaciones de cada informe. Incluye las características de ISO/IEC 25010 que quedaron sin evaluar y por qué.>

## 8. Plan de acción

| Prioridad | Acción | Hallazgos que resuelve | Responsable | Fecha objetivo |
|---|---|---|---|---|
| 1 | | | | |

## 9. Datos de prueba creados

<Lista de datos con prefijo `QA_` creados por todas las skills y cómo limpiarlos.>
