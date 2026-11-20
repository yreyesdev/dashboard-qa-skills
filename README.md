# Dashboard QA Skills

Colección de diez skills para auditar la calidad de un dashboard web de punta a punta. Cada skill es independiente: trae su propio alcance, preparación, pasos, escala de severidad, formato de hallazgo e informe final. Juntas cubren las nueve características del modelo de calidad de producto **ISO/IEC 25010:2023**, con trazabilidad a **ISO/IEC 25012** (calidad de datos), a los controles del Anexo A de **ISO/IEC 27001:2022** y, para el asistente de IA, a **ISO/IEC 42001**.

> Los informes que generan estas skills son **evidencia técnica**, no una certificación. ISO/IEC 27001 certifica el sistema de gestión de una organización, no un producto. Las skills citan normas solo por número y nombre, con descripciones propias; no reproducen su texto.

## Las skills

| Skill | Qué evalúa | Prefijo de hallazgos | Norma principal |
|---|---|---|---|
| [`dashboard-qa-funcional`](skills/dashboard-qa-funcional/SKILL.md) | Recorridos por rol, widgets, filtros, formularios, cálculos, exportaciones | `FUN` | ISO/IEC 25010, adecuación funcional |
| [`dashboard-qa-integridad-datos`](skills/dashboard-qa-integridad-datos/SKILL.md) | Que las cifras coincidan con la fuente de verdad | `DAT` | ISO/IEC 25012 |
| [`dashboard-qa-edge-cases`](skills/dashboard-qa-edge-cases/SKILL.md) | Estados vacíos, volumen, valores extremos, fechas, caracteres, red | `EDGE` | ISO/IEC 25010, fiabilidad |
| [`dashboard-qa-seguridad-acceso`](skills/dashboard-qa-seguridad-acceso/SKILL.md) | RBAC, IDOR, exportaciones, sesiones, OWASP Top 10, privacidad | `SEC` | ISO/IEC 27001, Anexo A |
| [`dashboard-qa-asistente-ia`](skills/dashboard-qa-asistente-ia/SKILL.md) | Inyección de prompts, aislamiento, acciones, fugas, veracidad | `IA` | ISO/IEC 27001 y 42001 |
| [`dashboard-qa-estado-concurrencia`](skills/dashboard-qa-estado-concurrencia/SKILL.md) | Sesión, pestañas, edición simultánea, tiempo real, carreras | `EST` | ISO/IEC 25010, fiabilidad |
| [`dashboard-qa-rendimiento`](skills/dashboard-qa-rendimiento/SKILL.md) | Web Vitals, API, base de datos, carga, estrés, resistencia | `PERF` | ISO/IEC 25010, eficiencia de desempeño |
| [`dashboard-qa-accesibilidad`](skills/dashboard-qa-accesibilidad/SKILL.md) | WCAG 2.2 AA: teclado, lectores de pantalla, gráficos, contraste | `A11Y` | WCAG 2.2 / ISO/IEC 40500 |
| [`dashboard-qa-compatibilidad-i18n`](skills/dashboard-qa-compatibilidad-i18n/SKILL.md) | Navegadores, dispositivos, traducciones, formatos regionales | `COMP` | ISO/IEC 25010, flexibilidad y compatibilidad |
| [`dashboard-qa-mantenibilidad`](skills/dashboard-qa-mantenibilidad/SKILL.md) | Código: modularidad, complejidad, pruebas, pipeline, dependencias | `MANT` | ISO/IEC 25010, mantenibilidad |

La correspondencia completa con las normas está en [`docs/trazabilidad-normativa.md`](docs/trazabilidad-normativa.md).

## Estructura del repositorio

```
dashboard-qa-skills/
├── .claude-plugin/
│   └── plugin.json              # Permite instalar el repositorio como plugin
├── skills/
│   └── <nombre-de-la-skill>/
│       └── SKILL.md             # Una carpeta por skill
├── docs/
│   └── trazabilidad-normativa.md
├── plantillas/
│   └── informe-consolidado.md   # Para unir los informes de varias skills
├── CHANGELOG.md
└── README.md
```

## Cómo usarlas

**Como skills sueltas en Claude:** sube cada carpeta de `skills/` (o su `SKILL.md`) desde la configuración de skills de Claude.

**Como plugin:** el repositorio incluye `.claude-plugin/plugin.json`, así que se puede instalar como plugin en los entornos que lo admiten y las diez skills quedan disponibles a la vez.

**Al pedir una auditoría**, nombra la skill y da el contexto que pide su sección "Antes de empezar". Por ejemplo:

> Usa `dashboard-qa-integridad-datos` sobre https://staging.ejemplo.com. Tienes acceso de solo lectura a la base de datos de staging. La zona horaria de la organización es America/Caracas.

## Orden sugerido

Si vas a ejecutar varias, este orden aprovecha lo que descubre cada una:

1. **Funcional**: construye el inventario de pantallas, widgets y roles que usan las demás.
2. **Integridad de datos**: confirma que las cifras son confiables antes de medir otra cosa.
3. **Seguridad y control de acceso** y, si existe, **asistente de IA**.
4. **Casos límite** y **estado y concurrencia**.
5. **Rendimiento**, en una ventana acordada con el equipo de infraestructura.
6. **Accesibilidad** y **compatibilidad e idiomas**.
7. **Mantenibilidad**, que mira el código y explica la causa de muchos de los hallazgos anteriores.

Para unir los resultados en un solo documento, usa [`plantillas/informe-consolidado.md`](plantillas/informe-consolidado.md).

## Reglas comunes a todas las skills

- Trabajar en staging o QA, nunca en producción sin autorización explícita.
- Usar solo datos ficticios, prefijados con `QA_AAAAMMDD_`, y documentar cómo limpiarlos.
- Las pruebas de seguridad y del asistente de IA exigen autorización previa del dueño del sistema.
- No afirmar nada que no se haya ejecutado: lo no probado se marca como "No verificado".
- No incluir credenciales, tokens ni datos personales reales en los informes.

## Contribuir

1. Crea una rama, edita el `SKILL.md` correspondiente y mantén su estructura de secciones.
2. Si cambias referencias normativas, actualiza también `docs/trazabilidad-normativa.md`.
3. Anota el cambio en `CHANGELOG.md`.
