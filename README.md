# Arqui_Soft_Lab08

## Spotty — Plataforma de calificación automática sobre AWS Spot

**Caso de Estudio #11 · Arquitectura de Software · UTEC 2026-II**
**Team:** _completar_ · **Integrantes:** _completar_

Hoy el curso de Arquitectura de Software califica las entregas de código de sus alumnos en un servidor local (una laptop), con un sandbox por alumno y workers (minions) que corrigen código y diagramas. Se quiere extender el sistema a todos los cursos usando únicamente instancias Spot de AWS para los sandboxes y workers.

---

## 1. Actores del sistema

| Actor | Tipo | Canal | Descripción |
|---|---|---|---|
| **Alumno** | Usuario principal | App mobile | Entrega sus trabajos (código, diagramas, modelos) y consulta el estado y la nota de cada entrega. |
| **Profesor** | Usuario principal | Web desktop | Crea las tareas de su curso cargando la rúbrica y la forma de evaluar la entrega del alumno, revisa el avance de la calificación y publica las notas como máximo 2 días después del deadline. **No necesita conocimientos técnicos ni configura infraestructura.** |
| **Administrador de TI** | Usuario de soporte | Web desktop / dashboards | Prepara la imagen base de cada curso y define los recursos por tipo de curso (CPU, memoria, GPU, tiempo máximo). Gestiona el presupuesto, la capacidad Spot y la salud de la plataforma. |
| **Worker / Minion** | Sistema interno | — | Ejecuta el código no confiable del alumno dentro de un sandbox aislado y genera el resultado de la evaluación. |
| **AWS EC2 Spot** | Sistema externo | API de AWS | Provee las instancias para los workers. Puede reclamarlas con 2 minutos de aviso o quedarse sin capacidad del tipo pedido (sobre todo GPU). |
| **Sistema académico de la universidad** | Sistema externo | API | Recibe las notas publicadas por el profesor. |
| **Servicio de notificaciones push** | Sistema externo | FCM / APNs | Entrega las notificaciones de estado a la app mobile del alumno. |
| **Proveedor LLM** | Sistema externo | API | Convierte la rúbrica del profesor en una evaluación ejecutable y ayuda a los workers a calificar criterios abiertos (diagramas, calidad de código). |

---

## 2. Requerimientos funcionales

| ID | Actor | Requerimiento |
|---|---|---|
| **RF-01** | Alumno | El sistema debe permitir al alumno subir una entrega (código, diagramas o notebooks/modelos) para una tarea abierta, rechazándola si el deadline ya pasó o si superó los intentos máximos. |
| **RF-02** | Alumno | El sistema debe permitir al alumno consultar desde la app mobile el estado de cada entrega (`EN COLA`, `EJECUTANDO`, `REINTENTANDO`, `CALIFICADO`, `ERROR`), junto con su nota y feedback, y notificarle cada cambio por push. |
| **RF-03** | Profesor | El sistema debe permitir al profesor crear una tarea mediante un formulario guiado, indicando solo el deadline, su **rúbrica** (criterios y puntajes) y la **forma de evaluar** la entrega (por ejemplo, casos de prueba con entrada y salida esperada, una solución de referencia o una lista de verificación para diagramas). Un LLM transforma la rúbrica en una evaluación ejecutable que el profesor aprueba con una vista previa. Los recursos y la imagen base se asignan automáticamente según el tipo de curso. |
| **RF-04** | Sistema | El sistema debe ejecutar y calificar cada entrega en un sandbox aislado con la imagen base del curso (con tests y, para criterios abiertos, apoyo de un LLM), y reintentarla automáticamente si AWS reclama la instancia, sin que el alumno vuelva a entregar. |
| **RF-05** | Profesor | El sistema debe mostrar al profesor el avance de calificación de cada tarea (calificadas, en cola, con error, tiempo estimado) y permitirle publicar las notas en el sistema académico. |

---

## 3. Requerimientos no funcionales

| ID | Atributo | Requerimiento | Métrica |
|---|---|---|---|
| **RNF-01** | Performance | Cada entrega debe calificarse dentro del P99 de su tipo de curso: **Programación ≤ 1 min**, **Arquitectura ≤ 10 min**, **IA ≤ 40 min** de ejecución. | P99 desde la entrega hasta el resultado, por tipo de curso |
| **RNF-02** | Escalabilidad | El sistema debe absorber el pico de entregas de los **últimos 30 minutos antes del deadline** escalando los workers según la cola, sin que los cursos con más alumnos degraden a los cursos pequeños. | P99 por curso durante la ventana de deadline |
| **RNF-03** | Reliability | Ninguna entrega debe perderse por una interrupción Spot (aviso de 2 minutos) o por falta de capacidad, y el 100 % de las notas deben estar listas antes de **deadline + 48 horas**. | `% jobs con nota / jobs recibidos = 100 %` · `% notas listas a las 48 h = 100 %` |
| **RNF-04** | Seguridad | El código del alumno es **no confiable**: debe ejecutarse sin acceso a la red, a credenciales ni a otros sandboxes, con límites duros de CPU, **memoria** y tiempo, de modo que un código ineficiente solo afecte su propia ejecución. | 100 % de la suite de código malicioso bloqueada |
| **RNF-05** | Costo | Los workers y sandboxes deben ejecutarse **únicamente en instancias Spot** de AWS, usando GPU solo para las ejecuciones que la necesiten. | % de cómputo de workers en Spot = 100 % |

---

## 4. Restricciones y supuestos

**Restricciones del enunciado**

- Solo instancias **Spot** para sandboxes y workers. AWS puede reclamarlas con **2 minutos de aviso** o quedarse sin capacidad, y las de GPU son las más escasas.
- Cada curso tiene su propia **imagen base de 3 a 5 GB**.
- El uso **no es constante**: casi todas las entregas llegan en los últimos 30 minutos antes del deadline.
- Algunos alumnos entregan código ineficiente o que consume mucha memoria.
- Los profesores tienen como máximo **2 días** para entregar las notas.

**Supuestos (pendientes de confirmar con el profesor)**

- ~10 000 alumnos y ~300 secciones con entregables. Peor caso: varias tareas con el mismo deadline.
- El plazo de 2 días se cuenta desde el deadline de la tarea.
- La restricción Spot solo aplica a workers y sandboxes; el API, la base de datos y las colas pueden usar servicios gestionados.
- El sandbox no necesita acceso a internet: las dependencias vienen dentro de la imagen base del curso.
- Los deadlines se conocen con anticipación.
- El profesor no configura infraestructura: TI prepara la imagen base de cada curso y la plataforma asigna los recursos según el tipo de curso (Programación, Arquitectura o IA).
