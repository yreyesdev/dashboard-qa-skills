---
name: dashboard-qa-estado-concurrencia
description: Prueba estado, sesión y concurrencia en un dashboard: sesión que expira, varias pestañas, edición simultánea, tiempo real, condiciones de carrera y uso prolongado, con trazabilidad a ISO/IEC 25010 y 27001.
---

# QA de estado, sesión y concurrencia

Los fallos de esta categoría no aparecen al probar con un usuario, una pestaña y cinco minutos. Esta skill reproduce el uso real: varias pestañas, varias personas a la vez, sesiones largas, redes que se cortan y datos que cambian mientras alguien los mira.

## Alcance
Incluye: ciclo de vida de la sesión, varias pestañas y ventanas, edición concurrente entre usuarios, idempotencia de acciones (dueña de pruebas profundas de doble clic y reintentos frente a `FUN`/`EDGE`), actualizaciones en tiempo real, condiciones de carrera en la interfaz, estado en la URL y navegación, persistencia de preferencias, uso prolongado y trabajos asíncronos.

Fuera de alcance: pruebas de carga con muchos usuarios simulados y auditoría de seguridad de tokens. Si encuentras algo de eso por casualidad, anótalo.

## Referencias normativas
Esta skill produce evidencia alineada con las normas siguientes, pero no certifica el cumplimiento de ninguna. Cita solo números y nombres de características o controles con descripciones propias; nunca copies el texto de las normas.

- **ISO/IEC 25010:2023, modelo de calidad de producto**
  - *Fiabilidad*:
    - Ausencia de fallos: sin pérdidas de actualizaciones ni duplicados (pasos 3 y 4).
    - Disponibilidad: el dashboard sigue operativo durante el uso prolongado (paso 9).
    - Tolerancia a fallos y recuperabilidad: reconexión de tiempo real y recuperación tras suspensión o corte (pasos 5 y 9).
  - *Seguridad*:
    - Integridad: los datos no se corrompen por accesos simultáneos (paso 3).
    - Confidencialidad: no se mezclan datos entre usuarios por caché o pestañas (pasos 2 y 8).
  - *Adecuación funcional / Corrección funcional*: el resultado corresponde a la última acción del usuario (paso 6) y el estado compartido por URL es el correcto (paso 7).
  - *Capacidad de interacción / Protección frente a errores del usuario*: no se pierde lo escrito al expirar la sesión (paso 1).
  - *Compatibilidad / Coexistencia*: varias pestañas y versiones conviven sin interferir (pasos 2 y 8).
- **ISO/IEC 27001:2022, Anexo A**:
  - Paso 1, sesión: 8.5 autenticación segura (expiración y cierre de sesión), 5.18 derechos de acceso (revocación de permisos con efecto inmediato).
  - Pasos 2 y 8, mezcla de datos entre usuarios: 8.3 restricción del acceso a la información.
  - Paso 3, concurrencia: 8.15 registro de eventos (los conflictos y cambios quedan registrados).
  - Paso 8, despliegues: 8.32 gestión de cambios (la versión anterior en pestañas abiertas).
  - Todo el proceso: 8.31 separación de entornos, 8.33 información de prueba.

## Antes de empezar
Reúne lo siguiente. Si falta algo, pregúntalo una sola vez; si nadie responde, continúa con supuestos y anótalos.
1. **Entorno**: staging o QA (estas pruebas modifican datos).
2. **Cuentas**: al menos dos usuarios con permiso de edición sobre los mismos recursos, y una cuenta a la que se le puedan cambiar los permisos durante la prueba.
3. **Configuración relevante**: tiempo de expiración de la sesión y del token, intervalo de auto-refresh, mecanismo de tiempo real (WebSocket, SSE, polling).
4. **Comportamiento esperado ante conflictos**: bloqueo, aviso, fusión o "gana el último". Si no está definido, ese vacío ya es un hallazgo.
5. **Herramientas**: dos perfiles de navegador o un navegador normal más una ventana privada, y las herramientas de desarrollo para simular red y avanzar timers.

## Reglas de trabajo
- Trabaja en staging o QA (8.31) con datos ficticios (8.33), prefijados con `QA_AAAAMMDD_` (o `QA_AAAAMMDD_EST_` si corres en paralelo con otras skills).
- Guarda secuencias, capturas y trazas de red relevantes en `evidencia-est/` junto al informe (sin credenciales ni tokens).
- Para reducir la espera en pruebas de expiración, pide al equipo una configuración de sesión corta en staging, o borra o modifica cookies y tokens para simular la expiración, y anota qué método usaste.
- Anota hora exacta y orden de las acciones en cada prueba concurrente: sin la secuencia, el fallo no es reproducible.
- Repite varias veces las pruebas de carrera; anota la frecuencia.

## Paso 1: Sesión
- La sesión expira mientras el usuario llena un formulario largo: al enviar, ¿se pierde lo escrito? Lo esperado es avisar, reautenticar y conservar los datos, o al menos advertir antes de expirar.
- Aviso previo de expiración con opción de extender, si está especificado.
- Renovación silenciosa del token (refresh): ocurre sin interrumpir; varias peticiones simultáneas con el token caducado no provocan varias renovaciones en conflicto ni un cierre de sesión.
- Tras reautenticarse, el usuario vuelve a la misma pantalla y con el mismo estado.
- Cerrar sesión en una pestaña: las otras pestañas lo detectan y no siguen mostrando datos ni permitiendo acciones.
- Iniciar sesión con otro usuario en otra pestaña: no se mezclan los datos de ambos usuarios.
- Cambio de permisos en caliente: un administrador quita un permiso al usuario mientras este está conectado; el cambio se aplica a la siguiente acción (no a la siguiente sesión).
- Usuario desactivado o borrado durante la sesión.

## Paso 2: Varias pestañas
- Filtros y vistas distintos en dos pestañas: no se pisan, salvo que esté especificado que se comparten.
- Editar el mismo registro en dos pestañas del mismo usuario y guardar ambas.
- Cambiar de organización o espacio de trabajo en una pestaña y hacer una acción en otra que aún muestra la organización anterior: la acción no debe aplicarse en la organización equivocada.
- Preferencias guardadas en una pestaña (tema, columnas, diseño): cómo se reflejan en la otra.

## Paso 3: Edición concurrente entre usuarios
Con los usuarios A y B sobre el mismo recurso:
- **Actualización perdida**: A y B abren el registro, A guarda, B guarda después. ¿Se pierde el cambio de A sin aviso? Lo deseable es detectar el conflicto (versión o ETag) y avisar a B.
- **Campos distintos**: A cambia un campo y B otro; verifica si ambos cambios sobreviven.
- **Borrado durante la edición**: A borra mientras B edita; B recibe un mensaje claro al guardar.
- **Transiciones de estado**: A y B aprueban o rechazan la misma solicitud a la vez; solo una transición debe aplicarse.
- **Unicidad**: A y B crean a la vez un registro con el mismo nombre o código único.
- **Contadores y saldos**: dos operaciones simultáneas sobre la misma cantidad (stock, presupuesto, cupo) no producen un resultado incorrecto.
- **Asignaciones**: dos personas toman la misma tarea a la vez.
- **Registro**: cada cambio queda en el historial con su autor y su hora, incluido el que se rechazó por conflicto, si así está definido (8.15).

Para lograr simultaneidad real, prepara ambas acciones y lánzalas a la vez. Si tienes acceso a la API, envía las dos peticiones en paralelo con un script, que es mucho más fiable que hacer clic en dos ventanas.

## Paso 4: Idempotencia y reintentos
- Doble clic y clics rápidos en guardar, enviar, aprobar o pagar: una sola acción.
- Recargar la página justo después de enviar un formulario (y aceptar el reenvío del navegador).
- Red cortada justo después de enviar: al reintentar, no se duplica la operación.
- Botón atrás tras completar una acción y volver a enviar.

## Paso 5: Tiempo real y auto-refresh
- Un cambio de B aparece en la pantalla de A en el tiempo esperado.
- La actualización llega mientras A está filtrando, ordenando, paginando, con un modal abierto o a mitad de un drill-down: no debe perder su posición, su selección ni lo que escribe.
- Desconexión y reconexión del canal de tiempo real: los cambios ocurridos durante la desconexión se recuperan (sin huecos ni duplicados).
- Orden de los eventos: dos cambios rápidos sobre el mismo registro se aplican en el orden correcto.
- Notificaciones duplicadas o perdidas tras una reconexión.
- Pestaña en segundo plano: al volver, los datos se actualizan o se indica que están desactualizados.

## Paso 6: Condiciones de carrera en la interfaz
- Cambiar un filtro varias veces seguidas: la respuesta lenta de una selección anterior no debe sobrescribir el resultado de la última (respuesta obsoleta). Usa red lenta para forzarlo.
- Navegar a otra pantalla mientras la anterior aún carga: los datos de la anterior no aparecen en la nueva.
- Abrir y cerrar rápidamente modales o paneles laterales.
- Escribir en una búsqueda con autocompletado rápidamente: los resultados corresponden al texto final.

## Paso 7: Estado en la URL y navegación
- Copiar la URL de una vista (filtros, periodo, pestaña, registro abierto) y abrirla en otra sesión: reproduce la misma vista.
- Abrir esa URL con un usuario que tiene otros permisos: ve solo lo que le corresponde, con un mensaje claro si no tiene acceso.
- Recargar mantiene el estado esperado.
- Atrás y adelante del navegador recorren los estados de forma lógica; no se rompe ni sale del dashboard inesperadamente.
- Parámetros inválidos o manipulados en la URL (filtro inexistente, fecha imposible, ID borrado): mensaje claro, sin pantalla en blanco.
- Enlaces antiguos guardados en favoritos tras cambios de rutas.

## Paso 8: Persistencia y caché
- Preferencias del usuario (columnas, orden, diseño, tema): se guardan, se mantienen al cerrar sesión y entre dispositivos según lo especificado, y nunca se mezclan entre usuarios del mismo equipo.
- Tras un despliegue nuevo, una pestaña con la versión anterior abierta: avisa de que hay una nueva versión o sigue funcionando sin errores (8.32).
- Service workers o caché sin conexión, si existen: no sirven datos de otro usuario ni datos muy viejos sin indicarlo.

## Paso 9: Uso prolongado
- Dashboard abierto varias horas con auto-refresh: memoria del navegador estable (comprueba en el administrador de tareas o en el perfil de memoria), sin acumulación de peticiones ni lentitud progresiva.
- Equipo suspendido y reanudado: la sesión, el canal en tiempo real y los datos se recuperan.
- Cambio de red (wifi a datos móviles, VPN).

## Paso 10: Trabajos asíncronos
- Exportaciones o informes largos: estado visible, notificación al terminar, opción de cancelar.
- Dos trabajos simultáneos del mismo usuario y de usuarios distintos: cada uno recibe su resultado.
- Cerrar la pestaña durante el trabajo y volver: el resultado sigue disponible.
- Un trabajo que falla: se informa con claridad y se puede reintentar.

## Severidad
- **Crítica**: pérdida silenciosa de cambios; acción duplicada con efecto económico o legal; acción aplicada en la organización o el usuario equivocado; datos de otro usuario visibles por caché o mezcla de sesiones.
- **Alta**: pérdida de lo escrito al expirar la sesión; respuestas obsoletas que muestran datos de otro filtro sin que se note; permisos revocados que siguen funcionando.
- **Media**: el estado no se conserva en la URL o al recargar; tiempo real con retrasos o duplicados visibles.
- **Baja**: incomodidades de navegación sin impacto en datos.

## Formato de cada hallazgo
```
ID: EST-001
Título:
Severidad: Crítica | Alta | Media | Baja
Categoría: Sesión | Pestañas | Concurrencia | Idempotencia | Tiempo real | Carrera UI | URL / navegación | Persistencia | Uso prolongado | Asíncrono
ISO/IEC 25010: <característica / subcaracterística, p. ej. Fiabilidad / Ausencia de fallos>
ISO/IEC 27001 Anexo A: <control, o "No aplica">
Actores: <usuarios, pestañas, navegadores>
Secuencia con tiempos:
  t0 — A abre el registro 123
  t1 — B abre el registro 123
  ...
Resultado esperado:
Resultado obtenido:
Frecuencia: <x de N>
Evidencia:
Recomendación:
```

## Informe final
1. **Resumen ejecutivo** con los riesgos de pérdida o duplicación de datos al principio.
2. **Matriz de cobertura** por categoría (pasos 1 a 10): OK / Falla / No verificado.
3. **Política de conflictos observada** frente a la esperada.
4. **Hallazgos** ordenados por severidad, con su secuencia.
5. **Trazabilidad normativa**: tabla de subcaracterísticas de ISO/IEC 25010 y controles de ISO/IEC 27001 evaluados, con resultado Evaluado sin hallazgos / Con hallazgos (IDs) / No evaluado, y una métrica cuando sea posible (por ejemplo, escenarios concurrentes sin pérdida de datos / total). Recuerda que el informe es evidencia y no una certificación.
6. **Supuestos, métodos de simulación usados y limitaciones.**
7. **Datos de prueba creados** y cómo limpiarlos.
