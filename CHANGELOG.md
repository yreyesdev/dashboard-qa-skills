# Changelog

## 1.1.0 — 2026-11-24

Mejoras de orquestación, fronteras entre skills y consolidación:

- Nueva skill `dashboard-qa-consolidar` para orquestar ejecuciones paralelas/secuenciales, aislar datos de prueba (`QA_AAAAMMDD_<PREFIJO>_`), proteger la ventana de carga de `PERF`, estandarizar carpetas `evidencia-<prefijo>/` y deduplicar hallazgos por causa raíz.
- Actualizada `plantillas/informe-consolidado.md` con la sección `2.1 Causas raíz (deduplicación)` y conteo bruto frente a problemas únicos.
- Aclaradas las fronteras entre skills con solapamiento frecuente (inyección de fórmulas en CSV entre `SEC` y `EDGE`, doble clic/carreras entre `FUN`/`EDGE`/`EST`, fechas y zonas horarias entre `DAT`/`COMP`/`EDGE`).
- Estandarizada la regla de evidencias y prefijos aislados en todas las skills, además de contemplar tanto dashboards analíticos como aplicaciones web de gestión.

## 1.0.0 — 2026-11-18

Primera versión.

- Diez skills independientes: funcional, integridad de datos, casos límite, seguridad y control de acceso, asistente de IA, estado y concurrencia, rendimiento, accesibilidad, compatibilidad e idiomas, y mantenibilidad.
- Cada skill incluye referencias normativas, formato de hallazgo con campos ISO y tabla de trazabilidad en el informe final.
- Documento de trazabilidad normativa con ISO/IEC 25010:2023, ISO/IEC 25012, ISO/IEC 27001:2022 Anexo A y otras normas.
- Plantilla de informe consolidado.
- Manifiesto de plugin en `.claude-plugin/plugin.json`.
