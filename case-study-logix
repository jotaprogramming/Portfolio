# Caso de estudio: reconstrucción de un sistema de distribución logística

> Por razones de confidencialidad, este documento no detalla la arquitectura interna del sistema ni qué tecnologías se usaron en cada componente. El foco está en el problema, las decisiones de ingeniería y sus consecuencias.

## Contexto

Una operación de retail con varios centros de distribución que abastecen a una red de tiendas. La distribución de mercancía es un proceso crítico: si el centro de distribución se detiene, las tiendas no reciben producto, y la capacidad de venta de la empresa se ve comprometida directamente.

El sistema de distribución existente había sido integrado con el ERP corporativo, que actúa como fuente única de verdad para inventario, compras y requisiciones. Esa integración se había construido para resolver una necesidad puntual y no una proyección del negocio.

## El problema

El sistema presentaba tres capas de problemas, en orden de gravedad:

**1. Confiabilidad de los datos.** El sistema y el ERP reportaban información distinta sobre los mismos hechos. No existía forma confiable de saber cuál de los dos tenía la información correcta, lo que obligaba a verificaciones manuales y reprocesos.

**2. Bloqueo de la operación.** Los procesos de importación de datos bloqueaban el acceso a la información durante varios minutos. Como estos procesos debían ejecutarse varias veces al día —la información en el ERP cambiaba durante la jornada—, la operación se detenía cada vez. El costo real para el negocio no era la lentitud del sistema en sí, sino tener operarios sin poder trabajar mientras esperaban.

**3. Deuda técnica estructural.** La lógica de negocio estaba distribuida entre la capa de presentación y la base de datos, sin una capa de aplicación que la centralizara. Parte del stack estaba descontinuado. Esto hacía que cualquier cambio fuera costoso y riesgoso, y que el sistema fuera difícil de escalar hacia nuevos centros de distribución.

## Restricciones

Cualquier solución tenía que convivir con estas condiciones, que no eran negociables:

- **Tiempo.** La operación estaba sufriendo a diario. No había margen para un proyecto largo.
- **Equipo.** Dos personas, con tiempo parcial (ambos manteníamos otros sistemas en paralelo).
- **Dependencia del ERP.** El ERP es la fuente de verdad corporativa. Cualquier sistema debe reportarle, y su disponibilidad no está bajo nuestro control.
- **Aprobación jerárquica.** Las decisiones de stack requerían aprobación de una jefatura conservadora frente a tecnologías nuevas.

## Alternativas evaluadas

### Parchar el sistema existente
**Descartada.** El acoplamiento era tal que corregir los problemas de confiabilidad implicaba reescribir la lógica de negocio de todas formas. Además, la lógica vivía repartida entre presentación y base de datos, sin una capa donde intervenir de forma centralizada. El esfuerzo de parchar se acercaba al de reconstruir, con la diferencia de que el resultado seguía siendo inmantenible.

### Sistema desacoplado por módulos, independiente del ERP
**Descartada por restricciones, no por criterio técnico.** Era la solución correcta: un sistema que operara de forma autónoma y un componente sincronizador dedicado a la comunicación con el ERP. Se descartó porque el diseño y desarrollo del sincronizador representaba una complejidad significativa que el tiempo disponible no permitía absorber.

### Monolito acoplado al ERP
**Elegida.** Un sistema vertical que mantiene la dependencia del ERP, pero rediseña los procesos y las integraciones que causaban los problemas. Se construyó sobre Django.

**El razonamiento:** el objetivo prioritario era devolverle productividad a la operación, no construir la arquitectura ideal. Django era la tecnología que el equipo ya dominaba, lo que permitía entregar dentro del plazo disponible. Cualquier alternativa implicaba que un equipo de dos personas aprendiera tecnologías nuevas en el mismo tiempo en que debía entregar, aumentando el costo y el riesgo del proyecto.

Una crítica válida al resultado es que el sistema no quedó reactivo, cosa que otro stack habría facilitado. La respuesta es que la necesidad prioritaria era la productividad de la operación, no la interfaz.

**El trade-off aceptado, explícitamente:** se sacrificó independencia del ERP —y con ella, resiliencia ante sus caídas— a cambio de tiempo de entrega. Se documentó como deuda técnica conocida, no como un descuido.

## Decisiones de diseño

Dentro de las restricciones de arquitectura y stack, estas fueron las decisiones de ingeniería:

### Concurrencia en los procesos de sincronización
El problema de bloqueo se atacó procesando la importación de datos de forma concurrente en lugar de secuencial. Esto redujo las ventanas en que la operación quedaba detenida.

### Idempotencia en las operaciones críticas
Las operaciones de sincronización se diseñaron para poder reintentarse sin duplicar transacciones. Esto era indispensable: la comunicación con el ERP falla con frecuencia, y sin idempotencia cada fallo de red introducía riesgo de duplicidad en los datos de inventario.

### Control de concurrencia sobre compromisos de mercancía
Durante el desarrollo se identificó que el sistema anterior no distinguía entre dos conceptos que el ERP trata por separado al comprometer y despachar mercancía. Cuando una misma solicitud se despachaba en cargas parciales, esa falta de distinción hacía que una carga absorbiera las cantidades de otra, dejando registros inconsistentes sin forma de rastrearlos.

Se implementó un mecanismo de compromiso y bloqueo secuencial: se compromete una carga, se bloquean las siguientes mientras se procesa, y si la operación falla se libera el compromiso y se reintenta con la siguiente. Esto eliminó ese tipo de inconsistencia.

### Modelado de datos e integraciones
El modelado de la base de datos y el diseño de las integraciones fueron decisiones propias, condicionadas por la necesidad de mantener correspondencia con las estructuras del ERP.

### Funcionalidades modeladas pero no implementadas
Se diseñaron y modelaron funcionalidades de ajuste y auditoría del proceso de preparación de pedidos. No se implementaron: el tiempo disponible se priorizó para los problemas de productividad. Quedaron documentadas para una fase posterior.

## Resultado

El sistema pasó de resolver una integración compleja e inestable a sostener toda la operación de distribución en tres centros y la red de tiendas, absorbiendo además casos de uso que no formaban parte del alcance inicial.

## Qué habría hecho diferente

No sostengo que lo siguiente sea la solución objetivamente correcta. Son las decisiones que yo habría tomado con el criterio que tengo hoy, y el razonamiento detrás de cada una.

### Operación local primero, sincronización en segundo plano

**Qué haría:** que el sistema opere de forma autónoma sobre los datos ya importados, con un componente sincronizador dedicado que gestione la comunicación con el ERP en segundo plano, con reintentos automáticos y control de estado.

**Por qué:** el criterio que aplicaría es que **la disponibilidad de un sistema no debería depender de otro sistema sobre el que no se tiene control.** La operación de distribución tiene que poder continuar aunque el ERP no responda; la sincronización de esos datos es importante, pero no tiene por qué ser inmediata.

Hoy ocurre algo que ilustra el costo de no haberlo hecho así: cuando el ERP no responde dentro del tiempo de espera definido, la transacción se interrumpe del lado del sistema —esperar más detendría la operación—, pero el ERP puede haberla completado igual. Eso deja transacciones en estado ambiguo que requieren verificación y reenvío manual. Un sincronizador con control de estado podría reconciliar esos casos sin intervención humana.

**El contra-argumento, que reconozco:** esta arquitectura implica manejar consistencia eventual y resolver conflictos de sincronización, lo cual introduce su propia complejidad. Para un equipo pequeño, esa complejidad es real y no gratuita.

### Control de flujo en lugar de concurrencia directa

**Qué haría:** diseñar el paralelismo considerando explícitamente la capacidad de procesamiento del sistema destino, usando encolamiento y control de flujo en lugar de lanzar operaciones concurrentes contra él.

**Por qué:** el criterio sería que **la concurrencia se diseña para el sistema más lento de la cadena, no para el propio.** En la práctica, el ERP no estaba preparado para recibir operaciones simultáneas: ante varias solicitudes concurrentes procesa una y rechaza el resto, lo que genera reenvíos manuales. La concurrencia del lado propio no produjo el beneficio esperado porque el cuello de botella estaba del otro lado.

También movería el paralelismo fuera del proceso de aplicación. La concurrencia hoy vive dentro del mismo proceso de Python, donde el GIL limita el paralelismo real de operaciones intensivas en CPU — un patrón de workers o procesos separados aprovecharía mejor los recursos disponibles.

### Identidad propia en el modelo de datos

**Qué haría:** que cada entidad del sistema tenga su propio identificador, y que el identificador del sistema externo sea un atributo más del registro.

**Por qué:** el criterio es que **la identidad de una entidad pertenece al sistema que la modela, no a la fuente desde la que llegó.** Hoy una tabla central usa como clave primaria el identificador que asigna el ERP. Cuando el sistema empezó a generar sus propios registros, no podía asignarles identificadores sin riesgo de colisión con los del ERP, lo que obligó a crear estructuras paralelas y un flujo indirecto: enviar el registro al ERP y después reimportarlo para obtener su identificador.

Con identidad propia, una sola estructura unificada habría podido contener registros de ambos orígenes sin distinción.

---

## Reflexión

La decisión de construir un monolito acoplado fue razonable dadas las restricciones: la operación necesitaba una solución en el plazo disponible, y una arquitectura mejor entregada tarde no habría resuelto el problema del negocio.

Pero fue una decisión con fecha de vencimiento, y esa fecha ya pasó. Los límites que hoy tiene el sistema son exactamente los que se aceptaron conscientemente al inicio. La diferencia entre deuda técnica y mal diseño es saber cuál se está contrayendo y por qué.
