# Jorge Salas - IT Administrator

## Rol
Responsable de la infraestructura cloud de la universidad y del budget de cómputo. Aprobó el uso
exclusivo de instancias Spot de AWS para sandboxes y workers. Mantiene el catálogo de plantillas
de entorno, del que salen las imágenes base de 3 a 5 GB de cada curso, porque los profesores no
configuran entornos. Responde por la disponibilidad de la plataforma en las noches de entrega y
por el costo mensual frente a la dirección académica.

## Un día de trabajo
1. Publica o actualiza plantillas en el Environment Catalog (por ejemplo Python con unit tests,
   Java, diagramas de arquitectura, PyTorch con GPU) con sus límites de memoria, CPU y timeout.
2. Revisa el calendario de fechas límite de la semana para anticipar los picos.
3. Durante cada pico monitorea la profundidad de cola, la antigüedad del job más antiguo y la tasa
   de interrupciones Spot por tipo de carga.
4. Recibe una alerta si el ETA de una evaluación supera el SLO de 24 h, por ejemplo cuando AWS
   no tiene capacidad Spot de GPU o la LLM API de los minions limita las llamadas.
5. Revisa los jobs en la dead letter queue y decide si se reprocesan.
6. Reporta a la dirección el costo por entrega corregida y el cumplimiento de SLO.

## Necesidades
- Necesita que el sistema use solo instancias Spot para los workers, para cumplir el budget
  aprobado.
- Necesita que los profesores elijan entornos de un catálogo cerrado, para que ningún curso
  introduzca una imagen sin límites ni una configuración insegura.
- Necesita que la falta de capacidad de un tipo de instancia no detenga la corrección, para no
  intervenir manualmente cada noche de entrega.
- Necesita un dashboard con cola, interrupciones, ETA y costo por tipo de carga, para detectar un
  incumplimiento de SLO antes que los profesores.
- Necesita que el código no confiable de los alumnos no pueda acceder a la red interna ni a
  credenciales de AWS.

## Pain points actuales
- Un solo servidor local (una laptop) es un SPOF y no escala al resto de cursos (físico).
- Cada curso nuevo implicaría armar su entorno a mano con el profesor (logístico).
- No tiene métricas: se entera de los problemas por correos de profesores (logístico).
- Las instancias de GPU on-demand exceden el budget aprobado (financiero).

## Qué le da el sistema
- Environment Catalog con Image Builder: una plantilla aprobada genera la imagen de cada curso.
- Workers 100 % Spot con diversificación de tipos, zonas y regiones.
- Dashboard y alertas de cola, ETA, interrupciones y costo.
- Sandboxes aislados sin red ni credenciales, con límites definidos en la plantilla.
