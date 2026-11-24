---
name: dashboard-qa-seguridad-acceso
description: Auditoría autorizada de seguridad y control de acceso de un dashboard: matriz RBAC, escalada de privilegios, IDOR, exportaciones, sesiones, OWASP Top 10 y privacidad, con trazabilidad a ISO/IEC 27001 y 25010.
---

# QA de seguridad y control de acceso (RBAC + OWASP)

Actúa como auditor de seguridad del propio dashboard del usuario: comprueba que cada persona solo puede ver y hacer lo que su rol permite, y que la aplicación resiste los fallos web más comunes. Es una prueba defensiva y autorizada, no un ataque.

## Condiciones obligatorias
No empieces sin confirmar todo esto. Si falta algo, detente y pregunta.
1. **Autorización**: el usuario confirma que es dueño del sistema o que tiene permiso escrito para probarlo, y cuál es el alcance (dominios, APIs, módulos).
2. **Entorno**: staging o QA. Producción solo con autorización explícita, y aun así sin pruebas destructivas.
3. **Cuentas de prueba**: al menos dos usuarios por rol y, si es multi-tenant, usuarios de al menos dos organizaciones distintas. Nunca uses cuentas de personas reales.
4. **Límites**: sin denegación de servicio, sin fuerza bruta masiva, sin ingeniería social, sin tocar sistemas de terceros y sin acceder a datos reales más allá de lo mínimo para demostrar un fallo.

Si encuentras una vulnerabilidad crítica (por ejemplo, acceso a datos de otra organización), detén esa línea de prueba, no extraigas más datos y avisa al usuario de inmediato antes de seguir.

## Alcance
Incluye: matriz de permisos, control de acceso vertical y horizontal, IDOR, multi-tenancy, exportaciones y descargas, autenticación, sesión y tokens, OWASP Top 10 aplicado al dashboard, fugas de información, privacidad y auditoría.

Fuera de alcance: pruebas de carga, auditoría de infraestructura y red, y seguridad del asistente de IA (si existe, se prueba aparte). Si encuentras algo de esto por casualidad, anótalo.

## Referencias normativas
Esta skill produce evidencia técnica para controles de las normas siguientes, pero no certifica el cumplimiento de ninguna. ISO/IEC 27001 certifica el sistema de gestión de la organización, no un producto: el informe sirve como evidencia de que ciertos controles funcionan en este dashboard. Cita solo números y nombres de controles con descripciones propias; nunca copies el texto de las normas.

- **ISO/IEC 27001:2022, Anexo A** (norma principal de esta skill), por paso:
  - Paso 1, matriz de permisos: 5.15 control de acceso, 5.18 derechos de acceso.
  - Paso 2, escalada vertical: 8.2 derechos de acceso privilegiado, 5.15.
  - Paso 3, IDOR y multi-tenant: 8.3 restricción del acceso a la información.
  - Paso 4, exportaciones: 8.3, 8.12 prevención de fuga de datos.
  - Paso 5, autenticación y sesión: 8.5 autenticación segura, 5.16 gestión de identidades, 5.17 información de autenticación.
  - Paso 6, OWASP: 8.26 requisitos de seguridad de aplicaciones, 8.28 codificación segura, 8.8 gestión de vulnerabilidades técnicas, 8.9 gestión de la configuración, 8.24 uso de criptografía.
  - Paso 7, fugas: 8.12, 8.4 acceso al código fuente (source maps y secretos publicados).
  - Paso 8, privacidad y auditoría: 5.34 privacidad y protección de datos personales, 8.11 enmascaramiento de datos, 8.15 registro de eventos, 8.10 eliminación de información.
  - Todo el proceso: 8.29 pruebas de seguridad en desarrollo y aceptación, 8.31 separación de entornos, 8.33 información de prueba.
- **ISO/IEC 27002:2022**: guía de implantación de los mismos controles; úsala para redactar las recomendaciones si la organización la tiene.
- **ISO/IEC 25010:2023**, característica *Seguridad*. Etiqueta cada hallazgo con una subcaracterística:
  - Confidencialidad: datos visibles solo para quien tiene permiso (pasos 1, 3, 4 y 7).
  - Integridad: nadie puede modificar lo que no debe (pasos 2, 3 y 6).
  - No repudio y responsabilidad: las acciones quedan registradas y atribuidas (paso 8).
  - Autenticidad: la identidad se verifica correctamente (paso 5).
  - Resistencia: el sistema resiste ataques comunes (paso 6).
- **ISO/IEC 27701**: extensión de privacidad, si la organización la aplica (paso 8).
- **Referencias técnicas complementarias** (no ISO): OWASP Top 10, OWASP ASVS y CWE para clasificar cada hallazgo.

## Herramientas
Navegador con herramientas de desarrollo (pestañas Red y Aplicación), `curl` o un cliente HTTP, y un proxy de intercepción (Burp Suite o OWASP ZAP) si el usuario lo tiene. Guarda cada petición y respuesta relevante en `evidencia-sec/` junto al informe, con los tokens y datos personales ocultados. Si creas datos de prueba en paralelo con otras skills, usa el prefijo `QA_AAAAMMDD_SEC_`.

## Paso 1: Matriz de permisos
Construye la matriz esperada (según la especificación o, si no existe, preguntando al usuario):

| Recurso o función | Acción (ver / crear / editar / borrar / exportar / compartir / administrar) | Rol A | Rol B | Rol C | Sin sesión |

Luego llena la matriz real probando cada celda, no solo en la interfaz sino también llamando directamente a la API. Cada diferencia entre la matriz esperada y la real es un hallazgo. Si no existe una matriz documentada, eso ya es un hallazgo frente al control 5.15.

## Paso 2: Control de acceso vertical (escalada de privilegios)
- Con un rol básico, abre directamente las URL de pantallas de administración.
- Llama a los endpoints de administración con el token de un rol básico, copiando las peticiones que hace un administrador.
- Elementos ocultos o deshabilitados en la interfaz: el endpoint correspondiente también debe rechazar la acción (403), no solo esconder el botón.
- Asignación masiva (mass assignment): añade campos como `role`, `isAdmin`, `permissions`, `orgId` o `ownerId` al cuerpo de las peticiones de perfil o de edición y comprueba que el servidor los ignora.
- Cambia parámetros de la petición que indican el rol o el alcance (`?scope=all`, `?admin=true`, cabeceras personalizadas).
- Métodos HTTP alternativos: si `GET` está protegido, prueba `POST`, `PUT`, `PATCH`, `DELETE` y `HEAD` sobre la misma ruta.

## Paso 3: Control de acceso horizontal e IDOR
Con el usuario A, captura los identificadores de sus recursos; con el usuario B (mismo rol), intenta acceder a ellos:
- IDs en la URL, en parámetros de consulta, en el cuerpo y en cabeceras.
- IDs secuenciales (prueba ±1) y también UUID obtenidos de otras respuestas, listados o exportaciones.
- Recursos anidados: `/orgs/{suOrg}/reports/{reportDeOtraOrg}`.
- Operaciones de lectura, edición, borrado, exportación y compartición.
- Multi-tenant: cambia `orgId` o `tenantId` en peticiones y en el token; cambia de organización en una pestaña y reutiliza peticiones de otra.
- GraphQL, si existe: consultas a objetos por ID, introspección habilitada y campos de otros usuarios en relaciones anidadas.
- Filtros como vía de fuga: un filtro por un ID ajeno no debe devolver datos ajenos ni conteos que revelen su existencia.

## Paso 4: Exportaciones y descargas
- La exportación respeta los permisos de filas y columnas del rol (no incluye campos ocultos en pantalla, como costos o datos personales).
- Enlaces de descarga: ¿son predecibles? ¿Caducan? ¿Funcionan sin sesión o con la sesión de otro usuario?
- Exportaciones asíncronas: el ID del trabajo de otro usuario no permite descargar su archivo.
- Inyección de fórmulas: un valor guardado como `=HYPERLINK("https://ejemplo.invalid","clic")` o `=1+1` no se ejecuta al abrir la exportación en Excel (debe ir escapado).
- Metadatos del archivo (autor, rutas internas) y PDF con capas o texto oculto.
- Límites: un usuario no puede exportar la base completa saltándose la paginación o los límites por rol.

## Paso 5: Autenticación, sesión y tokens
- Inicio de sesión: mensajes que no revelan si el usuario existe; límite de intentos o bloqueo temporal (prueba con pocos intentos, sin fuerza bruta real); recuperación de contraseña sin enumeración de usuarios y con tokens de un solo uso y caducidad.
- MFA, si existe: no se puede saltar yendo directamente a una URL posterior al login.
- Cookies: atributos `HttpOnly`, `Secure` y `SameSite` adecuados.
- JWT u otros tokens:
  - Modificar el contenido (rol, ID de usuario, expiración) sin volver a firmar: el servidor debe rechazarlo.
  - Algoritmo `none` o cambio de algoritmo: debe rechazarse.
  - Token caducado: debe rechazarse.
  - Información sensible dentro del token (es legible por cualquiera).
- Cierre de sesión: invalida la sesión en el servidor. Reutiliza un token o una cookie capturados antes del logout: debe fallar.
- Cambio de contraseña o de rol: invalida o actualiza las sesiones activas.
- Fijación de sesión: el identificador de sesión cambia tras iniciar sesión.
- Tiempo de inactividad: la sesión caduca según la política.
- Almacenamiento en el navegador: tokens o datos sensibles en `localStorage`, `sessionStorage` o en la caché.

## Paso 6: OWASP Top 10 aplicado al dashboard
- **Inyección**: en filtros, búsqueda, parámetros de orden (`sort`, `orderBy`) y paginación, prueba entradas que revelen un manejo inseguro (una comilla simple, `' OR '1'='1`, nombres de columna inexistentes). Busca errores de base de datos en la respuesta o cambios anómalos en los resultados. Detente al confirmar el fallo; no extraigas datos.
- **XSS**: guarda en nombres de widgets, etiquetas, comentarios y descripciones un payload inofensivo y reconocible, como `<img src=x onerror=console.log('QA-XSS-001')>`, y revisa si se ejecuta para otro usuario (almacenado). Prueba también en parámetros de la URL que se reflejan en la página. Revisa los tooltips de gráficos y las celdas de tablas, que a menudo renderizan HTML.
- **CSRF**: las acciones que cambian estado exigen un token o dependen de cookies `SameSite` adecuadas; una petición desde otro origen no debe completarse.
- **SSRF**: en funciones que reciben una URL (webhooks, importar desde URL, imágenes remotas), intenta direcciones internas (`http://127.0.0.1`, `http://169.254.169.254`) y observa si el servidor las alcanza. Solo observación, sin explotar.
- **Configuración** (8.9): cabeceras de seguridad (`Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `frame-ancestors`), CORS demasiado permisivo (origen arbitrario reflejado con credenciales), modo debug, trazas de error, documentación de API expuesta públicamente y listado de directorios.
- **Componentes vulnerables** (8.8): versiones de librerías visibles en cabeceras o en el JavaScript; contrástalas con avisos de seguridad conocidos.
- **Criptografía** (8.24): todo el tráfico en HTTPS, sin contenido mixto y con HSTS.
- **Límite de peticiones**: endpoints costosos (exportar, buscar, generar informes) tienen límites para evitar abuso.

## Paso 7: Fugas de información
- Respuestas de API con más campos de los que se muestran (sobre-exposición): datos personales, costos internos, IDs de otras organizaciones.
- Mensajes de error con trazas, rutas, nombres de tablas o versiones.
- Source maps publicados, comentarios en el JavaScript, claves de API o secretos dentro del bundle.
- Datos sensibles en la URL (quedan en historial, logs y cabecera Referer).
- Caché del navegador o de un proxy que guarda páginas con datos sensibles (`Cache-Control`).
- Conteos o mensajes que revelan la existencia de recursos ajenos ("no tienes permiso" frente a "no existe").

## Paso 8: Privacidad y auditoría
- Datos personales enmascarados según el rol (documento de identidad, teléfono, email) (8.11).
- Datos personales ausentes de logs del navegador, de la URL y de las herramientas de analítica (5.34).
- Registro de auditoría (8.15): queda constancia de quién vio, exportó, editó o borró datos sensibles, con fecha, hora y usuario, y el registro no lo puede alterar un usuario normal.
- Borrado y retención (8.10): lo borrado deja de aparecer en búsquedas, exportaciones y cachés según la política.

## Severidad
Usa una escala alineada con CVSS e indica impacto y facilidad de explotación:
- **Crítica**: acceso a datos de otra organización o de otros usuarios a escala, escalada a administrador, ejecución de código o inyección confirmada.
- **Alta**: IDOR sobre datos sensibles, XSS almacenado, salto de autenticación o de MFA, exportación con datos fuera de permiso.
- **Media**: CSRF en acciones de bajo impacto, sesiones que no se invalidan, CORS permisivo sin credenciales, enumeración de usuarios.
- **Baja**: cabeceras ausentes sin impacto directo, divulgación menor de versiones.

## Formato de cada hallazgo
```
ID: SEC-001
Título:
Severidad: Crítica | Alta | Media | Baja
Categoría: RBAC vertical | IDOR / horizontal | Multi-tenant | Exportación | Autenticación / sesión | Inyección | XSS | CSRF | SSRF | Configuración | Fuga de información | Privacidad
ISO/IEC 27001 Anexo A: <control o controles afectados, p. ej. 8.3 Restricción del acceso a la información>
ISO/IEC 25010: Seguridad / <subcaracterística>
OWASP / CWE: <si aplica>
Entorno, cuenta atacante y cuenta víctima: <roles y organizaciones, sin credenciales>
Precondiciones:
Pasos de reproducción:
  1. ...
Petición y respuesta relevantes: <con tokens y datos personales ocultados>
Impacto: <qué podría hacer un atacante real>
Resultado esperado:
Recomendación de corrección:
```

## Informe final
1. **Resumen ejecutivo** para dirección: nivel de riesgo general y hallazgos críticos y altos en lenguaje de negocio.
2. **Matriz de permisos** esperada frente a real, con las diferencias resaltadas.
3. **Hallazgos** ordenados por severidad, con su reproducción técnica.
4. **Controles verificados que funcionan bien** (también es información útil).
5. **Trazabilidad normativa**:
   - Tabla de controles de ISO/IEC 27001 Anexo A probados, con resultado: Evidencia de funcionamiento correcto / Con hallazgos (IDs) / No evaluado. Esta tabla puede aportarse como evidencia técnica en una auditoría interna del sistema de gestión o en la revisión de la Declaración de Aplicabilidad.
   - Tabla de subcaracterísticas de seguridad de ISO/IEC 25010 con el mismo formato.
6. **Alcance, autorización y limitaciones**: qué no se probó y por qué. Recuerda que el informe es evidencia y no una certificación.
7. **Recomendaciones priorizadas** y sugerencia de volver a probar tras las correcciones.

No incluyas credenciales, tokens completos ni datos personales reales en el informe.
