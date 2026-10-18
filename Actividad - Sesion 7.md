# Actividad de la sesión 7 — Persistencia local con Hive

Revisión: 18 de octubre de 2026.
Proyecto utilizado: `exploraec-curso/exploraec`, dentro de la carpeta de la práctica 7. Se conserva la práctica 6 por separado; no es necesario crear otro clon.

## Qué se realizó

1. Conectar GastosController con GastosRepository: el servidor es la primera opción y la copia local sirve de respaldo frente a errores de red o 5xx.
2. Conservar la última lista de gastos en `gastos_<id>`, usando el identificador de GET /usuarios/me y guardando la fecha de sincronización.
3. Mostrar el aviso de datos guardados, su fecha y el botón Reintentar; al recuperar el servidor se reemplaza la copia y desaparece el aviso.
4. Persistir favoritos en una caja independiente y recuperar la lista reactiva de GetX al iniciar la aplicación.
5. Eliminar los gastos locales del usuario al cerrar sesión, manteniendo favoritos y sin guardar el JWT en Hive.

La caché es de lectura. Un error 401 exige volver a iniciar sesión; los errores 400, 403 y 404 no deben ocultarse con el respaldo.

## Paso 0: ¿qué guardar y qué no?

Guardaría una copia de los últimos gastos y los favoritos para conservar información útil sin red; no guardaría el JWT sin protección en Hive.

## Paso 1: explicar el código

- `main()` es asíncrono porque inicializa Hive y abre la caja de favoritos antes de construir la interfaz; la caja de gastos se abre después de identificar al usuario.
- `toMap()` transforma un objeto en campos primitivos guardables y `fromMap()` reconstruye el objeto con esos mismos campos.
- El repositorio consulta el servidor, guarda una respuesta exitosa y solo usa la copia local disponible si falla la red o el servidor devuelve un 5xx.
- `gastos_<id>` separa los datos personales de cada usuario y evita compartir una caja genérica entre cuentas.

## Ejercicio de la diapositiva 9: elegir almacenamiento

| Dato | Mecanismo y justificación |
| --- | --- |
| Tema claro u oscuro | Preferencias, porque es un valor pequeño de configuración. |
| Lista de Gastos del viaje | Hive, porque conserva una colección simple de objetos serializados para consultarla sin red. |
| Foto descargada de un recibo | Archivo en el almacenamiento de la aplicación, porque el contenido es binario. |
| Gastos de varios usuarios con totales por categoría y rango de fechas | SQLite, porque permite relaciones, filtros y agregaciones mediante consultas. |

## Ejercicio de la diapositiva 22: cómo reconocer datos guardados

Mostrar un aviso visible de que se está consultando el respaldo y la fecha de la última sincronización. Añadir Reintentar permite volver a consultar el servidor.

## Ejercicio de la diapositiva 29: gasto creado sin red (dos líneas)

Guardar «Almuerzo en el Mercado Central, 6.50» en una cola local por usuario, con un identificador temporal y estado «pendiente de sincronizar».
Al recuperar la red, enviarlo al servidor y sustituir el identificador por el definitivo; si se rechaza, conservarlo con su error para corregirlo.

Es una propuesta de diseño: la sesión 7 no exige implementar esta cola. La respuesta ampliada del instructivo está en el informe de la práctica.

## Qué persiste y qué no

| Información | Al reiniciar | Al cerrar sesión |
| --- | --- | --- |
| Favoritos | Se recuperan de Hive | Se conservan como preferencias del dispositivo |
| Copia de gastos | Permanece guardada, pero no permite iniciar sesión sin backend | Se elimina la caja del usuario |
| JWT | Se pierde porque está en memoria | Se olvida |
| Lugar creado manualmente en Inicio | Desaparece de la lista en memoria | Si se marcó como favorito, su representación permanece en Favoritos |

Los favoritos corresponden a lugares públicos; los gastos son personales y se eliminan al salir para proteger la privacidad en un equipo compartido.

## Ejecutar y comprobar en Windows con Antigravity

Abrir `Sesion-7.code-workspace` en Antigravity. En su terminal, desde la carpeta `exploraec-curso/exploraec`:

```powershell
flutter pub get
flutter test
flutter run -d web-server --web-port 8186 --dart-define=API_BASE_URL=http://localhost:8000
```

Abrir http://localhost:8186/ en Brave. El backend reutilizado de la sesión 6 debe estar activo para iniciar sesión y cargar gastos.

Para comprobar la caché real: iniciar sesión con el servidor encendido, cargar primero los gastos, detener el backend, recargar Gastos y comprobar el aviso con los datos guardados. Después, iniciar el backend y pulsar Reintentar. No apagar el backend antes del login: el token vive únicamente en memoria.

Para comprobar favoritos: marcar un lugar, cerrar la pestaña y volver a abrir la misma dirección y puerto; debe permanecer en Favoritos. Comprobar también que un segundo usuario no ve los gastos del primero después de cerrar sesión.

## Alcance y entrega

No se requiere grabar un video compartiendo pantalla: el texto de la diapositiva se refiere al trabajo en vivo del docente. La persistencia del idioma y la inspección de archivos mediante adb son opcionales y no se agregaron en esta revisión.

El instructivo indica integrar este trabajo al proyecto final. El video también menciona entregar el enlace de GitHub; las condiciones actuales de Moodle deben comprobarse en la tarea antes de enviar. Esta revisión no realiza una entrega en Moodle ni modifica el repositorio remoto.

Fuentes: `instructivo-practica-sesion-07.pdf`, `sesion-07.pdf` y video «Video sesión 7: 07 de Octubre» de Moodle, analizado a solicitud del estudiante.
