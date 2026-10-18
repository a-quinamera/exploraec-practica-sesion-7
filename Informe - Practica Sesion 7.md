# Práctica — Sesión 7: Persistencia local con Hive

Verificación inicial: 10 de octubre de 2026. Revisión de la actividad y pruebas automatizadas: 18 de octubre de 2026.

## Resultado

Requisitos obligatorios del instructivo completados en Windows. Código editado en Antigravity y aplicación verificada en Brave. Proyecto en `exploraec-curso/exploraec`; abrir `Sesion-7.code-workspace` con Antigravity.

Aplicación de demostración: http://localhost:8186/
Backend reutilizado de la práctica 6: http://localhost:8000/

El anexo del PDF contiene instrucciones para Linux/macOS; se notificó y se adaptó la ejecución a Windows. La publicación de esta práctica en GitHub se realizó posteriormente, junto con las prácticas 2–8. En esta revisión no se realizó una nueva publicación ni una entrega en Moodle.

## Funcionalidad implementada

- Favoritos almacenados en Hive, recuperados al iniciar y mantenidos después de cerrar y volver a abrir la aplicación.
- Gastos consultados mediante GastosRepository: primero el servidor; cuando falla la red o hay un error 5xx, la última lista guardada.
- Caja personal `gastos_<id>` identificada mediante GET /usuarios/me, separada por usuario. Token únicamente en memoria.
- Aviso «Datos guardados — última sincronización» y botón Reintentar. El aviso también se muestra si la lista almacenada está vacía.
- Recuperación del servidor elimina el aviso y actualiza la copia local.
- Un 401 devuelve al formulario y no se oculta con datos almacenados.
- Cerrar sesión elimina la caja personal y vacía el estado del controlador; conserva favoritos de lugares públicos.
- Corrección de compatibilidad web: vaciar y cerrar la caja antes de Hive.deleteBoxFromDisk. La implementación inicial dejaba pendiente la eliminación de IndexedDB en Brave y bloqueaba el cierre de sesión.

## Verificación inicial y evidencias (10 de octubre)

| Comprobación | Resultado | Evidencia |
| --- | --- | --- |
| Diez pruebas del curso después de la corrección web | 10 aprobadas, incluyendo 401, fallo de red, 5xx, recuperación y eliminación de caja al salir | Evidencias/09-pruebas-finales-10-aprobadas.jpg; Resultado - pruebas finales.txt |
| Analizador Dart de Antigravity | 0 errores, 0 advertencias, 0 informaciones | Evidencias/05-analizador-sin-problemas.jpg |
| Backend detenido mediante Docker Desktop, recargar Gastos en Brave | Mantiene los tres gastos y muestra la fecha del respaldo | Evidencias/02-gastos-sin-backend.png |
| Backend reiniciado, pulsar Reintentar | Tres gastos desde el servidor; desaparece el aviso | Evidencias/03-servidor-recuperado.png |
| Cerrar y volver a abrir la aplicación | Parque El Ejido, Café Galletti y Mi lugar de prueba siguen en Favoritos | Evidencias/04-favoritos-tras-cerrar-y-abrir.png |
| Lugar creado localmente y aplicación reiniciada | Mi lugar de prueba desaparece de Inicio, pero conserva su representación en Favoritos | Evidencias/04-favoritos-tras-cerrar-y-abrir.png |
| Cerrar sesión en Brave | Regresa al formulario, sin quedar bloqueado | Evidencias/06-cierre-sesion-correcto.png |
| Entrar con una segunda cuenta local | Gastos (0), sin datos de la primera cuenta | Evidencias/07-segunda-cuenta-sin-gastos.png |
| Volver a la primera cuenta | Recupera sus tres gastos del servidor | Evidencias/08-primera-cuenta-restaurada.png |

Las pruebas son las diez suministradas por el curso; una corrección automática de estilo añadió llaves a un `if` del servidor simulado, sin cambiar las expectativas ni desactivar pruebas.

Limitación de la comprobación por comandos: `flutter analyze --no-pub` no pudo finalizar en el entorno de comandos de esta sesión porque Windows rechazó la resolución de `C:\flutter\bin\cache\flutter_tools.snapshot` (error 5). No se presenta como una ejecución CLI aprobada. El analizador del mismo SDK dentro de Antigravity sí verificó el proyecto sin problemas, y la aplicación compiló y ejecutó correctamente.

Se reutilizó la base de datos de la práctica 6 sin eliminar sus volúmenes. El backend estaba activo al terminar la verificación inicial; el 18 de octubre no estaba escuchando en localhost:8000. Las credenciales y tokens no se incluyen en las evidencias.

## Respuestas solicitadas por el instructivo

### ¿Cómo registrar un gasto sin conexión y sincronizarlo después? (propuesta, 4 líneas)

La caché actual es de lectura: sin conexión, un nuevo «Almuerzo en el Mercado Central, 6.50» todavía no existe en el servidor.
Se necesitaría una cola local de operaciones por usuario, con identificador temporal, datos del gasto y estado pendiente.
Al recuperar la conexión, se enviarían los pendientes y se asociaría el identificador temporal con el devuelto por el servidor, evitando duplicados.
Si el servidor rechaza un importe o categoría, se conservaría el pendiente con su error para corregirlo, sin marcarlo como sincronizado.

### Gastos y favoritos: decisión de privacidad (una frase)

Los gastos son personales y su caja sin cifrar se elimina al cerrar sesión para protegerlos en un dispositivo compartido, mientras que los favoritos representan lugares públicos y se conservan como preferencia del dispositivo.

## Alcance opcional

La persistencia del idioma mediante una caja de ajustes es opcional en el PDF y no bloquea la sesión 8; no está implementada. La inspección mediante adb pertenece al anexo Android y no se ejecutó en esta práctica web sobre Windows. No se implementó una cola de creación sin conexión: se respondió el diseño solicitado, que corresponde a trabajo posterior.

## Material que debe revisar el estudiante

El video y las diapositivas se analizaron posteriormente, por solicitud del estudiante. La frase «compartiendo pantalla» describe la demostración en vivo del docente; no solicita grabar un video del estudiante. No se marcaron materiales como completados en Moodle.

- Archivo de la Sesión 7: https://cursos.cedia.org.ec/mod/resource/view.php?id=49576
- VIDEO SESIÓN 7: 07 DE OCTUBRE: https://cursos.cedia.org.ec/mod/page/view.php?id=49580&forceview=1

## Entrega en la plataforma

El instructivo adjunto señala que esta práctica se integra al proyecto final sin una nueva entrega. Sin embargo, al revisar Moodle el 10/10/2026 sí existe la tarea «Práctica Sesión 7 - Persistencia local con Hive», con botón Agregar entrega, estado «Todavía no se han realizado envíos» y cierre el 14/10/2026 a las 00:00. Esta diferencia debe tenerse en cuenta al preparar la publicación final.

Tarea: https://cursos.cedia.org.ec/mod/assign/view.php?id=49577

No se pulsó Agregar entrega ni Marcar como completado, y no se subieron archivos. El proyecto y las evidencias permanecen en esta carpeta. El estado de Moodle y la fecha indicados arriba son observaciones históricas del 10 de octubre, no una comprobación actual.

Fuente de requisitos: instructivo-practica-sesion-07.pdf.

## Revisión del 18 de octubre de 2026

Se comparó la implementación existente con el instructivo de la práctica y con la actividad de las diapositivas/video. No fue necesario clonar otro proyecto ni modificar el código de aplicación: los pasos obligatorios ya estaban implementados.

- Las diez pruebas suministradas por el curso volvieron a pasar (`flutter test --no-pub --reporter expanded`). Cubren registro, lectura, errores de validación, red/5xx con y sin caché, recuperación, 401 y borrado de la caja al salir.
- `dart analyze lib test`: `No issues found!`.
- Se utilizó una copia local del SDK Flutter para evitar el bloqueo del SDK instalado en C:\flutter. El comando envolvente `flutter analyze` encontró una limitación de esa copia parcial (directorio dev ausente); se ejecutó directamente el analizador oficial de Dart sobre lib y test, con resultado satisfactorio.
- El backend real no estaba escuchando en localhost:8000. Las verificaciones reales de parada/reinicio del backend y cambio de usuario que aparecen en la tabla son del 10 de octubre; no se presentan como repetidas hoy.
- Las respuestas de los ejercicios de la presentación y las explicaciones de los pasos 0 y 1 se encuentran en `Actividad - Sesion 7.md`.

La compilación web (`flutter build web --no-pub --dart-define=API_BASE_URL=http://localhost:8000`) finalizó correctamente. Se sirvió el resultado en http://localhost:8186/ y se verificó en Brave que Parque El Ejido, Café Galletti y Mi lugar de prueba se recuperan después de reiniciar la aplicación. Evidencia actual: `Evidencias/10-favoritos-verificados-2026-10-18.png`. La comprobación fue sobre los favoritos existentes, sin eliminarlos ni modificar sus datos.


## Actualización del repositorio

Esta actualización incorpora las respuestas de la actividad, este informe y la captura del 18 de octubre. Las evidencias anteriores citadas corresponden al archivo local de la práctica y no se agregan en este commit. El código fuente publicado coincide con el revisado; no se modificó en esta revisión.

