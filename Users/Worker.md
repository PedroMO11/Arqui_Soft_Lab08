# Worker - Spot Sandbox Executor

## Rol
Actor de sistema. Cada worker es una instancia Spot de un pool (Programación, Arquitectura o IA)
que ejecuta el Worker Agent. Vive en el plano Spot efímero: no guarda datos que alguien vaya a
leer después, y AWS puede reclamarlo en cualquier momento con 2 minutos de aviso. Ejecuta el
código del alumno dentro de sandboxes, pero no califica. La nota la calculan los minions a través
del Minion Orchestrator.

## Ciclo de trabajo
1. Arranca desde la AMI de su shard, que ya contiene las imágenes base de sus cursos.
2. Pide un job al Scheduler y recibe un lease (3 min en Programación, 20 min en Arquitectura,
   heartbeat cada 60 s en IA).
3. Descarga la entrega de Artifact Storage y los tests ocultos de Rubric Storage.
4. Ejecuta la entrega y los tests en un sandbox sin red, con los límites de memoria, CPU y
   timeout de la plantilla.
5. Publica el output de ejecución en la Evaluation Queue junto con sus señales de ejecución
   (reintentos por infra, tests que cambian de resultado al repetirse, output truncado, uso de
   memoria o tiempo sobre el 90 % del límite) y libera el sandbox. A partir de ahí el job ya no
   depende del worker. Esas señales bajan la confianza de la nota y pueden mandarla a revisión
   manual.
6. Si recibe el aviso de interrupción, deja de pedir jobs, guarda checkpoint de un job de IA y
   libera los jobs que no alcanzan a terminar.

## Necesidades
- Necesita recibir en cada job solo referencias (entrega, tests, imagen, límites), para no
  depender de estado local que se pierde al ser reclamado.
- Necesita que su imagen ya esté en disco al arrancar, para empezar a ejecutar sin esperar una
  descarga de 3 a 5 GB.
- Necesita distinguir un fallo del alumno de un fallo propio, para que el Scheduler solo
  reintente los segundos.
- Necesita credenciales de corta duración que nunca entran al sandbox.

## Riesgos que representa
- Es reclamado por AWS sin control del sistema.
- Ejecuta código no confiable en el mismo host, por lo tanto el aislamiento del sandbox es la
  única barrera entre el código del alumno y sus credenciales.

## Qué le da el sistema
- Lease, heartbeat y reencolado automático ante interrupción.
- AMI por shard con las imágenes de sus cursos.
- Checkpoint Store para reanudar jobs de IA en otro worker.
