# R - Requirements: Spotty

## Requerimientos

| ID | Persona | Requerimiento |
|---|---|---|
| R1 | Professor, Teaching Assistant, Student, IT Administrator | Todo usuario se autentica con su cuenta UTEC a través del Login Service, y el Sign Up Service crea alumnos, profesores y jefes de práctica a partir de la matrícula de cada curso. Un estudiante solo ve sus propias entregas y un profesor o jefe de práctica solo los cursos asignados. |
| R2 | Professor | Como Profesor, crea una evaluación desde la desktop entregando solo el enunciado, la rúbrica (criterios, pesos y niveles), la fecha límite y una plantilla elegida del catálogo. No escribe tests, imágenes ni configuración de entorno. |
| R3 | Professor, Teaching Assistant, Minion | Al crear la evaluación, un minion compila una sola vez la rúbrica: los criterios objetivos se convierten en tests ocultos y los subjetivos quedan como instrucciones de evaluación. El Jefe de Práctica revisa y aprueba los tests antes de la fecha límite, y la corrección no empieza sin esa aprobación. Rubric Storage no es accesible para alumnos ni para su código. |
| R4 | Professor, IT Administrator | Profesores y jefes de práctica no pueden modificar workers, minions, sandboxes, imágenes ni límites de ejecución. Solo el Administrador TI publica plantillas en el Environment Catalog, que construye la imagen base de cada curso (3 a 5 GB) y la publica en el Image Registry. |
| R5 | Student | Como Estudiante, al entregar desde la web (desktop) o desde la app mobile antes de la fecha límite, el sistema registra la entrega con su timestamp de recepción y devuelve un comprobante en menos de 2 s (P99), aunque la corrección ocurra después. La puntualidad se decide por ese timestamp y nunca por la hora de corrección. |
| R6 | Student | Como Estudiante, ve desde la web o la app mobile el estado de cada entrega (En espera, En ejecución, En revisión, Corregido), recibe una push notification en la app al pasar a Corregido y, al final, ve el output de la corrección: nota por criterio de la rúbrica, resultado por test, stdout/stderr y causa del fallo. |
| R7 | Professor, IT Administrator | Cada entrega se ejecuta en un sandbox aislado dentro de un worker Spot, sin red ni credenciales, con los límites de memoria, CPU y timeout de su plantilla. Si el código excede un límite, solo ese sandbox termina y el resultado se registra como fallo del alumno (OOM o timeout) sin afectar otras ejecuciones del mismo worker. |
| R8 | Professor, Minion | El worker solo ejecuta y no califica. La nota la calcula el Minion Orchestrator en el plano permanente: los criterios objetivos con los tests ocultos y los subjetivos con los minions (LLM API externa). El código del alumno no puede leer los tests, abrir conexiones de red ni escribir su propio resultado, y su contenido viaja a los minions delimitado como dato, con respuesta estructurada por criterio, para que un texto escrito para manipular al modelo no altere la nota. |
| R9 | Professor, Student | El Strangler Router asigna cada entrega a un shard según su `curso_id`. Cada shard tiene su partición de cola, sus workers con la imagen de sus cursos y su partición de Submissions DB y Results DB. Los cursos con más de 300 matriculados tienen shard dedicado y los demás comparten shard con fair share en el Scheduler. Una ola de entregas de un curso de 600 alumnos no aumenta el tiempo de corrección de un curso de 30. |
| R10 | Student, Professor, Worker | La corrección es un pipeline de dos etapas: ejecución en el worker Spot y evaluación por minions desde la Evaluation Queue. Cuando AWS emite el aviso de interrupción de 2 minutos, el sistema deja de asignar jobs a ese worker, guarda checkpoint de los jobs de IA y reencola los que no terminaron su ejecución. Un job cuyo output ya llegó a la Evaluation Queue no depende del worker. Ninguna entrega se pierde, ninguna se califica dos veces y la interrupción no consume intentos del alumno. |
| R11 | IT Administrator | Cuando un tipo de instancia Spot no tiene capacidad en una zona, el sistema deja de pedirlo durante 15 minutos y solicita otros tipos y zonas equivalentes (y otra región en el caso de GPU) sin intervención manual. |
| R12 | Student, IT Administrator | Antes de cada fecha límite, el sistema escala los workers del shard del curso según sus matriculados y precarga la imagen base del curso en la AMI del shard, de modo que un job de Programación nunca espera la descarga de la imagen. |
| R13 | Minion, IT Administrator | Toda llamada a los minions pasa por un rate limiter y un circuit breaker. Si la LLM API falla o excede su límite, el job espera en la Evaluation Queue y se reintenta solo la evaluación, sin volver a ejecutar el sandbox y sin consumir intentos del alumno. |
| R14 | Professor | Como Profesor, todas las entregas de una evaluación quedan corregidas dentro de las 24 h posteriores a la fecha límite, ve en la desktop el avance y el ETA de la corrección y exporta las notas por criterio con sus outputs antes de que venzan las 48 h. |
| R15 | IT Administrator | Como Administrador TI, ve en un dashboard la profundidad de cada cola (incluida la Evaluation Queue), la antigüedad del job más antiguo, la tasa de interrupciones Spot y el costo por entrega, y recibe una alerta cuando el ETA de una evaluación supera 24 h o un job llega a la dead letter queue. |
| R16 | Student | Un estudiante tiene como máximo un job activo por evaluación y 10 entregas por hora, sumando web y app. Una reentrega reemplaza al job que aún está En espera y el exceso recibe HTTP 429. |
| R17 | Teaching Assistant | Como Jefe de Práctica, desde la desktop revisa el output, el puntaje por criterio y el timestamp de cualquier entrega de sus cursos y pide la recalificación de una evaluación. La recalificación entra al shard del curso con prioridad baja y reemplaza la nota anterior sin duplicarla. |
| R18 | Student, Professor | Los datos que leen alumnos y profesores (entregas, estados, notas, outputs, evaluaciones) viven solo en el plano permanente (on-demand y servicios administrados Multi-AZ). Los workers y sandboxes en Spot no guardan estado, por lo tanto la pérdida de todas las instancias Spot no impide entregar ni consultar estados, outputs o notas. |
| R19 | Professor, Teaching Assistant, Student | El Minion Orchestrator calcula una confianza entre 0 y 1 para cada nota, combinando la confianza que reporta el minion por criterio, señales de la ejecución en el worker (reintentos por infra, tests inestables, output truncado, límites casi alcanzados) y la consistencia entre tests y minions. Con confianza ≥ 0.85 la nota se publica sola. Con confianza < 0.85, o si la entrega cae en el muestreo aleatorio del 2 %, pasa al Manual Review Service y el Jefe de Práctica revisa lo que hicieron el worker y el minion antes de publicarla. Mientras tanto el alumno ve el estado En revisión, y la revisión cuenta dentro del SLO de 24 h. |
| R20 | Professor, Student, IT Administrator | La migración desde el sistema actual (laptop) es gradual y por curso, nunca de golpe. El Strangler Router decide por `curso_id` si una entrega va al Legacy Grader o al shard nuevo. Data Migration copia el histórico (entregas, notas y rúbricas de ciclos anteriores) a los almacenamientos nuevos a través de sus servicios, y mientras un curso corre en ambos sistemas sincroniza las notas del legacy. Un curso pasa al sistema nuevo solo después de dos evaluaciones en paralelo con 100 % de coincidencia de notas, y puede volver al legacy si aparece una falla. Alumnos y profesores ven su historial completo en el sistema nuevo desde el primer día. |

## Parámetros

| ID | Parámetro | Valor |
|---|---|---|
| P1 | P99 de ejecución, Programación (unit tests, código) | 1 min |
| P2 | P99 de ejecución, Arquitectura (diagramas, código) | 10 min |
| P3 | P99 de ejecución, IA (modelos con GPU) | 40 min o más |
| P4 | Aviso de interrupción Spot | 2 min |
| P5 | Imagen base por curso | 3 a 5 GB |
| P6 | Plazo del profesor para entregar notas | 48 h desde la fecha límite |
| P7 | SLO de corrección | 100 % de entregas corregidas en 24 h |
| P8 | Patrón de llegada | ~80 % de las entregas en los últimos 30 min antes de la fecha límite |
| P9 | Pico estimado por fecha límite (supuesto) | 3,000 entregas de Programación, 600 de Arquitectura, 400 de IA |
| P10 | Límites de sandbox por plantilla | Programación 1 vCPU, 2 GB, 3 min. Arquitectura 2 vCPU, 4 GB, 20 min. IA 1 GPU, 4 vCPU, 16 GB, 3 h |
| P11 | Tipo de cómputo | Workers y sandboxes solo en instancias Spot. Plano permanente en on-demand y servicios administrados. Minions en LLM API externa |
| P12 | Canales de acceso | Profesor y Jefe de Práctica solo desde desktop (web). Estudiante desde desktop (web) o app mobile, para entregar y para consultar |
| P13 | Shard key | `curso_id`. Shard dedicado para cursos con más de 300 matriculados |
| P14 | Rate limit de la LLM API de los minions (supuesto) | 300 requests/min |
| P15 | Insumos del profesor por evaluación | Enunciado, rúbrica, fecha límite y plantilla del catálogo |
| P16 | Umbral de confianza para publicar sin revisión | 0.85. Muestreo aleatorio del 2 % de las notas sobre el umbral |
| P17 | Tasa esperada de revisión manual (supuesto) | ~5 % de las entregas, ~3 min de Jefe de Práctica por entrega |
| P18 | Sistema legacy | Laptop que hoy califica Arquitectura de Software, con el histórico de entregas y notas de ciclos anteriores |
