# Luis Torres - Teaching Assistant

## Rol
Jefe de práctica de Programación II y Arquitectura de Software. Es el enlace técnico del
profesor: valida que los tests generados a partir de la rúbrica midan lo que el profesor quiere,
revisa outputs, atiende reclamos y pide recalificaciones. Trabaja desde una desktop y tampoco
administra la infraestructura.

## Un día de trabajo
1. Inicia sesión en la desktop con su cuenta UTEC.
2. Revisa los tests ocultos que el sistema generó a partir de la rúbrica del profesor, ajusta los
   que no correspondan y los aprueba. Sin su aprobación la evaluación no se corrige.
3. Después de la fecha límite atiende la cola de revisión manual: las entregas cuya nota salió con
   confianza menor a 0.85 y una muestra aleatoria del 2 % de las demás. Para cada una ve lo que
   hizo el worker (tests, output, reintentos, límites alcanzados) y lo que respondió el minion
   (puntaje y evidencia por criterio), y confirma o corrige la nota.
4. Atiende el reclamo de un alumno comparando el timestamp de recepción con la fecha límite.
5. Si un test estaba mal, lo corrige y pide la recalificación de toda la evaluación.

## Necesidades
- Necesita ver los tests generados antes de la fecha límite, para que la nota no dependa de un
  test mal interpretado de la rúbrica.
- Necesita ver el output completo de cada entrega y el puntaje por criterio con su justificación,
  para resolver un reclamo sin volver a ejecutar el código a mano.
- Necesita revisar solo las notas dudosas y no las 600 de un curso, para terminar la revisión
  dentro de las 24 h del SLO.
- Necesita ver por qué una nota quedó con baja confianza (qué señal la bajó), para revisar rápido.
- Necesita recalificar una evaluación completa sin que esa ola de jobs retrase la corrección de
  una fecha límite que está en curso.
- Necesita que la recalificación reemplace la nota anterior y no cree una segunda.

## Pain points actuales
- Hoy recalificar significa volver a correr todo en la laptop durante horas (logístico).
- No tiene evidencia de la hora de recepción para responder reclamos de puntualidad (emocional).

## Qué le da el sistema
- Tests generados desde la rúbrica, editables y con aprobación previa a la corrección.
- Cola de revisión manual con las notas de confianza menor a 0.85, cada una con la señal que bajó
  la confianza, el output del worker y la evidencia del minion.
- Output, puntaje por criterio y timestamp de recepción por entrega desde la desktop.
- Recalificación con prioridad baja, enrutada al shard del curso, con upsert idempotente de la
  nota.
