# Trazabilidad normativa

Este documento relaciona cada skill con las normas que usa. Sirve para saber qué skill aporta evidencia sobre cada característica o control, y para detectar huecos de cobertura.

> Las referencias se hacen por número y nombre, con descripciones propias. Contrasta siempre con el texto oficial de cada norma, que tu organización debe adquirir. Los informes son evidencia técnica, no una certificación.

**Abreviaturas de las skills:** FUN funcional · DAT integridad de datos · EDGE casos límite · SEC seguridad y control de acceso · IA asistente de IA · EST estado y concurrencia · PERF rendimiento · A11Y accesibilidad · COMP compatibilidad e idiomas · MANT mantenibilidad.

**Leyenda:** ● cobertura principal · ○ cobertura parcial o secundaria.

## 1. ISO/IEC 25010:2023, modelo de calidad de producto

| Característica | Subcaracterística | FUN | DAT | EDGE | SEC | IA | EST | PERF | A11Y | COMP | MANT |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Adecuación funcional | Completitud funcional | ● | | | | | | | | | |
| | Corrección funcional | ● | ● | | | ● | ○ | | | ○ | |
| | Pertinencia funcional | ● | | | | | | | | | |
| Eficiencia de desempeño | Comportamiento temporal | | | | | | | ● | | | |
| | Utilización de recursos | | | | | | | ● | | | |
| | Capacidad | | | ○ | | | | ● | | | |
| Compatibilidad | Coexistencia | | | | | | ○ | | | ● | |
| | Interoperabilidad | | | | | | | | | ● | |
| Capacidad de interacción | Reconocibilidad de la adecuación | | | | | | | | ● | | |
| | Capacidad de aprendizaje | ○ | | | | | | | | | |
| | Operabilidad | | | | | | | | ● | | |
| | Protección frente a errores del usuario | ○ | | ● | | | ○ | | ● | | |
| | Compromiso del usuario | | | | | | | | | | |
| | Inclusividad | | | | | | | | ● | ● | |
| | Asistencia al usuario | | | ● | | | | | ● | | |
| | Autodescriptividad | ○ | | ● | | ○ | | | ● | ● | |
| Fiabilidad | Ausencia de fallos | | ○ | ● | | | ● | | | | |
| | Disponibilidad | | | | | | ● | ○ | | | |
| | Tolerancia a fallos | | | ● | | | ● | ○ | | | |
| | Recuperabilidad | | | ● | | | ● | ○ | | | |
| Seguridad | Confidencialidad | | | | ● | ● | ○ | | | | |
| | Integridad | | | | ● | ● | ● | | | | |
| | No repudio | | | | ● | | | | | | |
| | Responsabilidad | | | | ● | ● | | | | | |
| | Autenticidad | | | | ● | | | | | | |
| | Resistencia | | | | ● | ● | | | | | |
| Mantenibilidad | Modularidad | | | | | | | | | | ● |
| | Reusabilidad | | | | | | | | | | ● |
| | Analizabilidad | | | | | | | | | | ● |
| | Modificabilidad | | | | | | | | | | ● |
| | Capacidad de prueba | | | | | | | | | | ● |
| Flexibilidad | Adaptabilidad | | | ○ | | | | | | ● | |
| | Escalabilidad | | | | | | | ● | | | |
| | Instalabilidad | | | | | | | | | | |
| | Reemplazabilidad | | | | | | | | | | |
| Protección (safety) | Todas | | | | | | | | | | |

**Huecos conocidos:**
- *Compromiso del usuario* y *Capacidad de aprendizaje*: requieren pruebas con usuarios reales (usabilidad, encuestas), fuera del alcance de una auditoría técnica.
- *Instalabilidad* y *Reemplazabilidad*: poco relevantes para un dashboard web; si se distribuye como producto instalable, conviene añadir una skill.
- *Protección (safety)*: aplica a sistemas que pueden causar daño físico o ambiental. Para un dashboard suele no aplicar, salvo que controle equipos u operaciones físicas.

## 2. ISO/IEC 25012, modelo de calidad de datos

Cubierto por **DAT**:

| Característica | Paso de DAT |
|---|---|
| Exactitud | 2, 3, 9 |
| Completitud | 3, 8 |
| Consistencia | 7 |
| Credibilidad | 1 |
| Actualidad | 6 |
| Precisión | 5 |
| Trazabilidad | 1, 8 |
| Comprensibilidad | 1, 5 |

## 3. ISO/IEC 27001:2022, Anexo A

Solo se listan los controles con evidencia técnica que aportan las skills. Los controles organizacionales, de personas y físicos quedan fuera.

| Control | Nombre | FUN | DAT | EDGE | SEC | IA | EST | PERF | A11Y | COMP | MANT |
|---|---|---|---|---|---|---|---|---|---|---|---|
| 5.15 | Control de acceso | | | | ● | ● | | | | | |
| 5.16 | Gestión de identidades | | | | ● | | | | | | |
| 5.17 | Información de autenticación | | | | ● | | | | | | |
| 5.18 | Derechos de acceso | | | | ● | | ● | | | | |
| 5.23 | Seguridad en servicios en la nube | | | | | | | | | ○ | |
| 5.33 | Protección de registros | | ○ | | | | | | | | |
| 5.34 | Privacidad y protección de datos personales | | | | ● | ● | | | | | |
| 5.37 | Procedimientos operativos documentados | | | | | | | | | | ○ |
| 8.2 | Derechos de acceso privilegiado | | | | ● | ● | | | | | |
| 8.3 | Restricción del acceso a la información | | ○ | | ● | ● | ● | | | | |
| 8.4 | Acceso al código fuente | | | | ○ | | | | | | ● |
| 8.5 | Autenticación segura | | | | ● | | ● | | ○ | | |
| 8.6 | Gestión de la capacidad | | | ○ | | | | ● | | | |
| 8.8 | Gestión de vulnerabilidades técnicas | | | | ● | | | | | | ● |
| 8.9 | Gestión de la configuración | | | | ● | | | | | | ● |
| 8.10 | Eliminación de información | | | | ● | | | | | | |
| 8.11 | Enmascaramiento de datos | | | | ● | ● | | | | | |
| 8.12 | Prevención de fuga de datos | | | | ● | ● | | | | | |
| 8.14 | Redundancia de instalaciones de procesamiento | | | | | | | ○ | | | |
| 8.15 | Registro de eventos | | | | ● | ● | ● | | | | |
| 8.16 | Actividades de monitorización | | ● | | | | | ● | | | |
| 8.24 | Uso de criptografía | | | | ● | | | | | | |
| 8.25 | Ciclo de vida de desarrollo seguro | | | | | | | | | | ● |
| 8.26 | Requisitos de seguridad de las aplicaciones | | | ● | ● | ● | | | | | |
| 8.28 | Codificación segura | | | | ● | ● | | | | | ● |
| 8.29 | Pruebas de seguridad en desarrollo y aceptación | ● | | | ● | ● | | | | | |
| 8.31 | Separación de entornos | ● | ● | ● | ● | ● | ● | ● | | ○ | |
| 8.32 | Gestión de cambios | | | | | | ● | | | | ● |
| 8.33 | Información de prueba | ● | ● | ● | ● | ● | ● | | | ○ | |

Las tablas de trazabilidad de los informes de SEC, IA y MANT pueden aportarse como evidencia técnica en una auditoría interna del sistema de gestión o en la revisión de la Declaración de Aplicabilidad.

## 4. Otras normas y referencias

| Norma o referencia | Uso | Skills |
|---|---|---|
| ISO/IEC 27002:2022 | Guía de implantación de los controles del Anexo A | SEC |
| ISO/IEC 27701 | Extensión de privacidad | SEC |
| ISO/IEC 42001 | Sistema de gestión de la IA | IA |
| ISO/IEC 23894 | Gestión del riesgo de IA | IA |
| ISO/IEC 25023 | Medidas de calidad de producto | PERF, MANT (y cualquier skill que quiera métricas formales) |
| ISO/IEC/IEEE 29119 | Proceso y documentación de pruebas | FUN, `dashboard-qa-consolidar` (y la plantilla de informe consolidado) |
| ISO/IEC 40500 | Versión ISO de WCAG 2.0 | A11Y |
| ISO 9241-171 | Accesibilidad del software | A11Y |
| EN 301 549 | Requisitos de accesibilidad TIC (Europa) | A11Y |
| ISO 8601 | Fechas y horas | DAT, EDGE, COMP |
| ISO 4217 | Códigos de moneda | DAT, COMP |
| ISO 639 / ISO 3166 | Códigos de idioma y país | COMP |
| OWASP Top 10, ASVS, CWE | Clasificación técnica de vulnerabilidades (no ISO) | SEC |
| OWASP Top 10 para LLM, MITRE ATLAS | Clasificación de riesgos de IA (no ISO) | IA |
| WCAG 2.2 (W3C) | Criterios de accesibilidad (no ISO) | A11Y |
