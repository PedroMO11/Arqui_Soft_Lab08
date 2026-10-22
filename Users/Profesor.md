# Martín Paredes - Profesor

## Rol
Profesor de Programación I, uno de los cursos con más demanda de la universidad (600 alumnos en
6 secciones con la misma fecha límite). No es especialista en infraestructura: no escribe
Dockerfiles, no configura entornos ni workers y no escribe unit tests. Usa el sistema desde una
desktop para publicar el enunciado y la rúbrica, seguir el avance de la corrección y exportar las
notas. Tiene como máximo 2 días desde la fecha límite para entregar las notas.

## Un día de trabajo
1. Inicia sesión desde la desktop.
2. Crea una evaluación con el enunciado, la rúbrica (criterios, pesos y descripción de cada
   nivel) y la fecha límite, y elige una plantilla del catálogo, por ejemplo "Python con unit
   tests".
3. El jefe de práctica revisa los tests que el sistema generó a partir de la rúbrica.
4. La noche de la fecha límite casi todas las entregas llegan en los últimos 30 minutos.
5. A la mañana siguiente revisa en el dashboard cuántas entregas están corregidas, cuántas siguen
   en espera y la hora estimada de término.
6. Exporta las notas con el puntaje por criterio y las publica antes de que venzan las 48 h.

## Necesidades
- Necesita que la rúbrica sea lo único que debe entregar, para no depender de conocimientos de
  infraestructura que no tiene.
- Necesita que todas las entregas estén corregidas dentro de las 24 h posteriores a la fecha
  límite, para tener al menos 24 h de revisión antes de publicar notas.
- Necesita ver el avance y el ETA de cada evaluación, para pedir una extensión a tiempo si hace
  falta.
- Necesita ver la nota desglosada por criterio de su rúbrica, para justificar cada nota ante un
  reclamo.
- Necesita que un alumno no pueda leer los tests ni manipular la corrección desde el código que
  entrega, para que la nota automática sea confiable.
- Necesita que su curso de 600 alumnos no espere detrás de otros cursos ni los bloquee.

## Pain points actuales
- Para usar el calificador actual tendría que preparar el entorno y los tests él mismo, cosa que
  no sabe hacer (físico).
- En las noches de entrega la corrección tarda horas y no hay forma de saber cuándo termina
  (logístico).
- Recibe reclamos de alumnos que entregaron a tiempo y no puede demostrar la hora real de entrega
  (emocional).

## Qué le da el sistema
- Evaluaciones creadas solo con enunciado, rúbrica, fecha límite y una plantilla del catálogo.
- Corrección completa en menos de 24 h, con avance y ETA por evaluación.
- Nota por criterio, timestamp de recepción y output de cada entrega, exportables.

## Qué no puede hacer
- Modificar workers, minions, sandboxes, imágenes o límites de ejecución. Esos los define el
  Administrador TI en el catálogo de plantillas.
