# Diego Huamán - Student

## Rol
Estudiante de tercer año que lleva Programación II e Inteligencia Artificial en el mismo ciclo.
Entrega código y notebooks que entrenan modelos con GPU. Sube sus entregas desde la web en su
laptop, porque ahí escribe el código y ahí están los archivos, y consulta el estado y el output
desde la app mobile, normalmente en los minutos previos y posteriores a la fecha límite.

## Un día de trabajo
1. Inicia sesión en la web desde su laptop con su cuenta UTEC.
2. Termina su tarea de IA 15 minutos antes de la fecha límite y la sube desde la web.
3. Recibe un comprobante con la hora de recepción.
4. Desde la app mobile revisa el estado de su entrega: En espera, En ejecución, En revisión (un
   jefe de práctica está validando la nota), Corregido o Error (la corrección falló por la
   plataforma y TI ya fue alertado).
5. En IA, cada intento antes de la fecha límite le devuelve en minutos un smoke test en CPU, y
   solo su entrega final se ejecuta en GPU.
6. Recibe una push notification cuando la entrega pasa a Corregido.
7. Abre el output de la corrección: nota por criterio de la rúbrica, tests aprobados y fallidos,
   stdout/stderr y, si falló por falta de memoria o timeout, la causa.

## Necesidades
- Necesita que una entrega hecha antes de la fecha límite cuente como puntual aunque la
  corrección ocurra horas después, para no perder puntos por la congestión del sistema.
- Necesita ver el estado real de su entrega desde el celular, para no reenviar el mismo trabajo
  varias veces por desconfianza.
- Necesita que una interrupción de la infraestructura no le cueste un intento ni una nota de cero.
- Necesita saber si su código falló por su culpa (memoria, tiempo) o por la plataforma.
- Necesita entender por qué perdió puntos en cada criterio de la rúbrica.

## Pain points actuales
- En la última media hora el sistema actual deja de responder y no sabe si su entrega llegó
  (emocional).
- Una ejecución de IA de 40 minutos se pierde completa si la máquina se reinicia, y tiene que
  pedir al profesor que la vuelva a correr (logístico).
- No hay app mobile ni notificaciones. Tiene que preguntar al profesor por correo (logístico).

## Qué le da el sistema
- Entrega desde la web (desktop) con comprobante y timestamp de recepción en menos de 2 s.
- Estado de la entrega en la web o la app (En espera, En ejecución, En revisión, Corregido, Error) y
  push notification al pasar a Corregido.
- Una nota dudosa nunca se publica sin que un jefe de práctica la revise.
- Retry automático ante interrupciones Spot, sin consumir intentos del alumno.
- Feedback rápido de cada intento de IA con un smoke test en CPU, sin esperar GPU.
- Output final de la corrección: nota por criterio, resultado por test, stdout/stderr y causa del
  fallo.
