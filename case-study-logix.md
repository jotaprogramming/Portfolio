# LogiX: evolución de una plataforma crítica de distribución logística

> Este documento describe decisiones de ingeniería y su razonamiento. No detalla topología de producción, configuración de infraestructura, reglas completas de negocio ni estructuras de datos, por confidencialidad y seguridad.

---

## Contexto

Una operación de retail donde la distribución de mercancía es un proceso crítico: si un centro de distribución se detiene, las tiendas no reciben producto y la capacidad de venta se ve comprometida de forma directa.

El ERP corporativo (Siesa) actúa como fuente de verdad para inventario, compras y requisiciones. Cualquier sistema logístico tiene que reportarle, y su disponibilidad no está bajo control del equipo de desarrollo.

Lideré la reconstrucción del sistema de distribución y mantengo su ownership técnico desde entonces. Eso significa que las decisiones que describo aquí no son hipótesis: he visto sus consecuencias en producción a lo largo de más de un año de evolución.

El nombre del proyecto sería LogiX.

---

## El sistema heredado

El sistema anterior había sido construido para resolver una necesidad puntual, no una proyección del negocio. Acumulaba problemas de seguridad, mantenibilidad, confiabilidad de datos, productividad y experiencia de usuario, y era difícil de evolucionar.

Técnicamente, la lógica de negocio vivía repartida entre la capa de presentación y la base de datos, sin una capa de aplicación que la centralizara. Parte del stack estaba descontinuado.

### Los problemas concretos

**Confiabilidad de los datos.** El sistema y el ERP reportaban información distinta sobre los mismos hechos. No existía forma confiable de saber cuál tenía razón, lo que obligaba a verificaciones manuales y reprocesos.

**Bloqueo de la operación.** Los procesos de importación bloqueaban el acceso a la información durante varios minutos y debían ejecutarse varias veces al día, porque las requisiciones se anulaban y regeneraban durante la jornada. Cada ejecución detenía la operación.

**Despacho secuencial.** Sincronizar cada unidad de carga con el ERP requería dos operaciones por cada requisición contenida, ejecutadas en serie. A unos diez segundos por operación, una unidad con diez requisiciones implicaba veinte operaciones consecutivas: alrededor de tres minutos. Cuando el ERP se degradaba y cada operación superaba el minuto, ese tiempo escalaba hasta los veinte minutos por unidad.

El costo real para el negocio no era la lentitud del software. Era tener operarios detenidos esperando.

---

## Restricciones del proyecto

- **Tiempo.** La operación sufría a diario. No había margen para un proyecto largo.
- **Equipo.** Dos personas, ambas manteniendo otros sistemas en paralelo.
- **Dependencia del ERP.** Obligatoria, y sin control sobre su disponibilidad.
- **Gobernanza técnica.** Las decisiones de stack requerían aprobación fuera del equipo de desarrollo.
- **Riesgo de adopción.** Introducir tecnologías que el equipo no dominara aumentaba el riesgo de no entregar.

---

## Alternativas evaluadas

### Parchar el sistema existente

`Descartada`

Corregir los problemas de confiabilidad implicaba reescribir la lógica de negocio de todas formas, y esa lógica estaba dispersa entre presentación y base de datos, sin una capa donde intervenir de forma centralizada. El esfuerzo se acercaba al de reconstruir, con la diferencia de que el resultado seguiría siendo inmantenible.

### Arquitectura desacoplada con sincronización independiente

`Descartada`

Era la alternativa que yo consideraba más adecuada: que el sistema operara sobre su propio estado y que un componente independiente gestionara la sincronización con el ERP de forma asíncrona.

Implementarla correctamente exigía resolver persistencia de estados, reintentos, reconciliación de resultados ambiguos, idempotencia, control de concurrencia y recuperación ante indisponibilidad del sistema externo. Con dos desarrolladores a tiempo parcial y presión operativa inmediata, esa complejidad excedía lo que el proyecto podía absorber en el plazo disponible.

### Monolito Django acoplado al ERP

`Implementada`

Un sistema vertical que mantiene la dependencia del ERP pero rediseña los procesos y las integraciones que causaban los problemas.

Elegí Django porque era la tecnología que el equipo ya dominaba. Cualquier alternativa implicaba aprender mientras entregábamos. El stack inicial fue Python 3, Django, PostgreSQL, JavaScript, HTML/CSS y Bootstrap, con SQL Server para determinadas consultas e integraciones contra el ERP.

Una crítica válida al resultado es que el sistema no quedó reactivo, algo que otro stack habría facilitado. La prioridad, sin embargo, era la productividad de la operación, no la interfaz.

**Lo que se ganó:** una solución en producción dentro del plazo que la operación necesitaba.\
**Lo que se sacrificó:** independencia del ERP y, con ella, resiliencia ante su indisponibilidad.

---

## Decisiones de ingeniería de la primera versión

### Concurrencia en la sincronización

Ataqué el bloqueo procesando la importación de datos de forma concurrente en lugar de secuencial, reduciendo las ventanas en que la operación quedaba detenida.

El ERP aceptaba operaciones simultáneas, y ese comportamiento fue una premisa de diseño durante toda la construcción del sistema. Más adelante ese supuesto dejó de cumplirse.

### Idempotencia en operaciones críticas

Diseñé las operaciones de sincronización para poder reintentarse sin duplicar transacciones. Era indispensable: la solución debía tolerar indisponibilidad y tiempos de respuesta variables del ERP, y sin idempotencia cada interrupción introducía riesgo de duplicidad en los datos de inventario.

### Control de concurrencia sobre compromisos de mercancía

Durante el desarrollo identifiqué que el sistema anterior no distinguía entre dos conceptos que el ERP trata por separado al comprometer y despachar mercancía. Cuando una misma requisición se despachaba en cargas parciales, esa falta de distinción hacía que una carga absorbiera las cantidades de otra, dejando registros inconsistentes sin forma de rastrearlos.

Implementé un mecanismo de compromiso y bloqueo secuencial: se compromete una carga, se bloquean las siguientes mientras se procesa, y si la operación falla se libera el compromiso y se reintenta con la siguiente. Eso eliminó ese tipo de inconsistencia.

### Modelo de datos e integraciones

Diseñé el modelo de datos y las integraciones, condicionado por la necesidad de mantener correspondencia con las estructuras del ERP. Esa decisión tuvo consecuencias que aparecen más adelante en este documento.

### Despliegue

Desplegué sobre Ubuntu Server con Gunicorn y Apache2, como instalación directa sobre el sistema operativo. Para el alcance de ese momento —una operación principal, pocas instalaciones, infraestructura conocida— era la estrategia adecuada.

**Resultados de la primera entrega:** la importación de datos pasó de minutos a segundos; el despacho de una unidad de carga pasó de unos tres minutos a segundos, y desaparecieron los errores de inconsistencia entre cargas.

---

## De proyecto a producto

La primera entrega no cerró el proyecto. Mantuve el ownership técnico y durante los meses siguientes corregí incidentes, mejoré rendimiento, optimicé procesos, añadí funcionalidades y refactoricé componentes sobre un sistema que ya tenía usuarios reales.

Es la parte del trabajo que más me ha enseñado: las decisiones de la primera versión dejaron de ser hipótesis y empezaron a producir consecuencias observables.

---

## Cuando una dependencia externa cambia las reglas

El caso más instructivo de esa etapa no fue un error propio.

LogiX se diseñó y se construyó sobre un supuesto verificado: el ERP aceptaba operaciones simultáneas. La concurrencia en la sincronización era una de las razones por las que el sistema había recuperado la productividad de la operación.

Un día, sin aviso ni despliegue de nuestro lado, la integración empezó a fallar con respuestas con errores de *deadlock* y de documento ya existente —fallos característicos de un destino que ya no tolera operaciones concurrentes. No hubo cambios en LogiX ese día ni en los anteriores.

Escalamos el caso con el equipo del ERP para recuperar el comportamiento anterior. Su posición fue que no habían modificado nada y que el proceso seguía siendo el mismo. Sin acceso al sistema y sin acuerdo sobre qué había cambiado, no había forma de revertirlo.

**La única salida era adaptar LogiX.** Tuvimos que reducir la concurrencia contra el ERP y ajustar la operación a una restricción que no existía cuando el sistema fue diseñado, sacrificando parte del rendimiento que se había ganado.

Lo que aprendí de esto no fue sobre concurrencia. Fue sobre acoplamiento: **cuando un sistema depende del comportamiento de otro que no controlas, no dependes de su disponibilidad solamente, sino también de que sus reglas no cambien.** Un supuesto verificado en el momento del diseño puede dejar de ser cierto sin que nadie lo anuncie, y sin que exista un interlocutor dispuesto a reconocerlo.

Este episodio es la evidencia más directa a favor de la arquitectura que había propuesto al inicio. Una capa de sincronización independiente, con control de flujo explícito hacia el destino, habría permitido absorber ese cambio ajustando una configuración en lugar de modificar el comportamiento del sistema y de la operación.

---

## Cuando el alcance original llegó a su límite

LogiX se había construido pensando en una operación principal. Cuando fue necesario soportar un segundo centro de distribución, apareció un límite del diseño: **el sistema no era multitenant, ni había sido concebido para múltiples centros independientes.**

Había dos caminos. Convertir LogiX en multitenant implicaba una modificación profunda de un sistema que ya estaba en producción sosteniendo una operación crítica. Mantener instancias independientes evitaba esa reescritura pero introducía un problema distinto.

Elegí el modelo de instancias independientes. El nuevo centro recibió la suya.

### El problema que eso trasladaba al usuario

Si los puntos de venta tuvieran que trabajar directamente contra cada centro, los usuarios tendrían que saber qué centro usar, cambiar de aplicación, iniciar sesiones distintas y modificar su flujo según el destino. Eso convertía una decisión de arquitectura en carga cognitiva para el operario.

La solución fue extender el modelo: además de las instancias de los centros, configuré instancias asociadas a los puntos de venta, de modo que cada usuario trabajara sobre su propia instancia conservando su contexto operacional.

### Comunicación entre instalaciones

Para que esas instalaciones pudieran colaborar, incorporé Django REST Framework y LogiX empezó a exponer APIs REST.

El valor de esa decisión no fue adoptar una tecnología, sino **desacoplar la interacción del usuario de la ubicación física de la operación con la que trabaja**. El usuario sigue en su instancia; LogiX se encarga de la comunicación entre sistemas.

### Una precisión que importa

Esto **no fue una migración a microservicios**. Cada instancia de LogiX sigue siendo la misma aplicación monolítica que se construyó al principio. Lo que cambió fue la arquitectura del sistema completo y la estrategia de despliegue.

La descripción técnicamente correcta es **un monolito desplegado de forma distribuida**: un conjunto de instancias independientes de una misma aplicación, capaces de comunicarse entre sí.

---

## Evolución de la estrategia de despliegue

Con muchas más instalaciones, administrarlas directamente sobre el sistema operativo habría complicado el aislamiento, las dependencias, la configuración, los despliegues, el mantenimiento y cualquier migración futura.

Adopté Docker y Nginx **para las instancias nuevas**. Buscaba aislamiento, reproducibilidad, facilidad de despliegue y mantenimiento, y portabilidad entre servidores.

El cambio conceptual: antes, una instalación estaba fuertemente ligada al servidor donde se había configurado; después, una instancia podía empaquetarse y administrarse con relativa independencia del host que la ejecuta.

Eso permitió algo deliberado: **varias instancias podían compartir infraestructura al inicio, con la posibilidad de separarlas después si alguna lo necesitaba.** La contenerización no se adoptó porque cada instancia tuviera su propio servidor, sino precisamente para no tener que decidir eso desde el primer día.

### La transición fue progresiva

No reescribí la infraestructura existente. Durante un periodo coexistieron estrategias distintas: los centros podían seguir operando sobre Ubuntu Server con Apache2 mientras las instancias nuevas usaban Docker con Nginx.

La estrategia nueva se aplicó donde resolvía un problema real, no por uniformidad tecnológica.

### La decisión se validó después

Más adelante fue necesario mover la operación de uno de los centros a infraestructura dedicada. Como esa instancia ya estaba contenerizada, la migración fue considerablemente más simple que si la instalación hubiera estado configurada directamente sobre el sistema operativo.

Fue una decisión tomada pensando en flexibilidad futura que terminó produciendo un beneficio concreto cuando el crecimiento lo exigió.

---

## Adaptación a operaciones distintas

Incorporar un tercer centro con características operativas diferentes reveló un problema conceptual más interesante que cualquier requerimiento nuevo:

**Varias reglas que parecían formar parte general del dominio resultaron ser específicas de la forma de operar del primer centro.**

Mientras solo existió una operación, esa distinción era invisible. Un segundo contexto la hizo evidente y obligó a separar lo general de lo particular.

La alternativa fácil habría sido duplicar la base de código. En lugar de eso refactoricé los módulos afectados e introduje mecanismos de configuración, de modo que el mismo producto pudiera adaptar determinados comportamientos según el contexto operacional en el que se despliega.

Ese trabajo incluyó también la generación de órdenes de compra y requisiciones desde LogiX, y su envío hacia el ERP.

---

## Mayor responsabilidad dentro del dominio

La adaptación anterior produjo un cambio de fondo en la relación entre LogiX y el ERP.

Originalmente LogiX consultaba, importaba y consumía información originada en el ERP. Con esta evolución empezó también a generar determinados documentos, originar operaciones y enviarlas hacia él.

LogiX no reemplaza al ERP: Siesa sigue siendo la fuente de verdad corporativa. Pero dejó de ser únicamente una capa operacional consumidora.

### Distribución basada en lo realmente recibido

El cambio funcional más significativo del producto surgió de una diferencia que la operación conocía pero el sistema no modelaba.

LogiX importaba del ERP requisiciones construidas sobre cantidades planificadas. La realidad podía diferir: un proveedor con una cantidad solicitada podía entregar menos. Entre lo solicitado, lo efectivamente recibido y lo realmente disponible para distribuir había diferencias, y resolverlas quedaba en manos del operario durante la preparación de pedidos.

Modifiqué ese enfoque para que LogiX generara directamente parte de las necesidades de distribución tomando como base la información de mercancía efectivamente recibida.

El efecto fue reducir decisiones manuales durante el picking y permitir que la distribución respondiera a la disponibilidad real en lugar de a una planificación previa.

### Menos interacción directa con el ERP

Más recientemente incorporé la capacidad de ejecutar desde LogiX el recibo de determinados documentos provenientes de los centros, un proceso que antes obligaba a los puntos de venta a trabajar directamente sobre el ERP.

Esto continúa una tendencia del producto: **LogiX ha ido encapsulando progresivamente la complejidad de interacción con el ERP**, funcionando cada vez más como la capa operacional especializada en procesos logísticos.

---

## Integraciones externas

Durante la evolución del producto integré además una plataforma B2B externa (CEN) para el intercambio de información logística, incluyendo avisos de despacho, mediante generación y transferencia de archivos por SFTP.

---

## Resultados

- Sustitución completa del sistema heredado, eliminando su deuda estructural y sus problemas de confiabilidad de datos.
- Reducción de los tiempos que detenían la operación: la importación de datos pasó de minutos a segundos y el despacho de una unidad de carga, de unos tres minutos a segundos.
- Eliminación de las inconsistencias entre cargas que obligaban a verificación manual.
- Expansión desde una operación principal hasta soportar procesos de distribución en **3 centros de distribución y 24 tiendas**.
- Evolución de una instalación única a un modelo de instancias independientes que se comunican mediante APIs REST.
- Evolución de la estrategia de despliegue hacia contenedores, con portabilidad demostrada al migrar una operación a infraestructura dedicada.
- Soporte de contextos operativos distintos sobre una misma base de código, mediante refactorización y configuración, sin duplicar el producto.
- Incorporación de integraciones con sistemas externos además del ERP.
- Reducción de tareas manuales y de la necesidad de que usuarios operativos interactúen directamente con el ERP.
- Capacidad de originar determinadas operaciones del dominio, no solo consumirlas.
- Continuidad de la operación ante un cambio no anunciado en el comportamiento del sistema externo, mediante adaptación del propio sistema.
- Sistema en producción y en evolución continua bajo mi ownership.

---

## Límites actuales y deuda conocida

**Dependencia de la disponibilidad del ERP.** Cuando el ERP no está disponible, LogiX no puede continuar. Es la consecuencia directa del trade-off aceptado al inicio.

**Dependencia del comportamiento del ERP.** Más allá de su disponibilidad, LogiX depende de que las reglas del sistema externo no cambien. Ya ocurrió una vez con la tolerancia a operaciones concurrentes, y la adaptación tuvo que hacerse del lado de LogiX y de la operación.

**Transacciones en estado ambiguo.** Cuando el ERP no responde dentro del tiempo de espera definido, LogiX interrumpe la transacción para no detener la operación, pero el ERP puede haberla completado igual. Esos casos requieren verificación y reenvío manual.

**Concurrencia acoplada al proceso de aplicación.** La concurrencia se ejecuta dentro del propio proceso que atiende a los usuarios, sin una capa que permita ajustar el flujo hacia el destino de forma independiente. Esa rigidez fue precisamente lo que encareció la adaptación cuando el ERP dejó de aceptar operaciones simultáneas.

**Acoplamiento en el modelo de datos.** Una estructura central usa como identificador propio el que asigna el ERP. Cuando LogiX empezó a originar sus propios registros, eso obligó a diseñar estructuras paralelas y a un flujo indirecto en lugar de una estructura unificada.

**Funcionalidades modeladas y no implementadas.** Diseñé funcionalidades de ajuste y auditoría del proceso de preparación de pedidos que quedaron documentadas pero fuera de alcance por tiempo.

---

## Qué habría hecho diferente

No sostengo que lo siguiente sea objetivamente correcto. Son las decisiones que tomaría hoy, con el criterio que me dio mantener este sistema en producción.

### Operación local primero, sincronización en segundo plano

**Qué haría:** que el sistema opere de forma autónoma sobre sus propios datos, con un componente sincronizador que gestione la comunicación con el ERP en segundo plano, con reintentos y control de estado.

**Por qué:** la disponibilidad de un sistema no debería depender de otro sobre el que no se tiene control. La distribución tiene que poder continuar aunque el ERP no responda; la sincronización importa, pero no tiene por qué ser inmediata.

La evidencia acumulada respalda el criterio: las transacciones en estado ambiguo que hoy requieren intervención manual son exactamente el tipo de caso que un sincronizador con control de estado podría reconciliar automáticamente.

**El contra-argumento, que reconozco:** implica consistencia eventual y resolución de conflictos, complejidad real y no gratuita para un equipo pequeño. Esa fue precisamente la razón de descartarla al inicio, y seguiría siendo un factor.

### Ejecución desacoplada del trabajo de sincronización

**Qué haría:** mover las operaciones de sincronización a workers con una cola y control explícito de concurrencia, fuera del proceso que atiende a los usuarios.

**Por qué:** el trabajo que depende de un sistema externo no debería ejecutarse en el mismo espacio que atiende a los usuarios. Hoy la concurrencia vive dentro del proceso de aplicación mediante hilos que hacen llamadas síncronas de larga duración: si el ERP responde lento, el sistema completo se degrada.

Una cola aporta además algo que este caso demostró con claridad: **el nivel de concurrencia hacia un destino externo debería ser un parámetro, no una característica del código.** Cuando el ERP dejó de aceptar operaciones simultáneas, adaptarse significó modificar el comportamiento del sistema. Con control de flujo explícito, habría significado cambiar un valor.

Como nota secundaria: el cuello de botella es de I/O, no de CPU, así que el problema principal es el acoplamiento de la ejecución y no el GIL de Python.

### Identidad propia en el modelo de datos

**Qué haría:** que cada entidad tenga su propio identificador y que el del sistema externo sea un atributo más.

**Por qué:** la identidad de una entidad pertenece al sistema que la modela, no a la fuente desde la que llegó.

Esta es la decisión cuyo costo he podido medir mejor. Usar el identificador del ERP como clave primaria daba correspondencia directa y simplificó la primera versión. Pero cuando LogiX empezó a originar sus propios registros, no podía asignarles identificadores sin riesgo de colisión, lo que obligó a diseñar estructuras paralelas y a enviar el registro al ERP para después reimportarlo y obtener su identificador. Una estructura unificada habría contenido registros de ambos orígenes sin distinción.

### Lo que no cambiaría

**La contenerización.** Añadió complejidad operacional en su momento, pero el aislamiento y la portabilidad que buscaba se materializaron cuando hubo que migrar una operación a infraestructura dedicada.

**El monolito.** Con dos personas, tiempo parcial y presión operativa, un monolito sobre tecnología conocida fue lo que permitió entregar y sostener el sistema. El crecimiento posterior no se resolvió reescribiéndolo, sino cambiando cómo se despliega.

---

## Reflexión

La arquitectura inicial resolvió correctamente el problema que existía bajo las restricciones de ese momento. LogiX terminó creciendo mucho más allá del alcance para el que fue concebido.

Varias de sus limitaciones actuales son consecuencias conocidas de decisiones que fueron razonables en su contexto. Otras decisiones, tomadas pensando en flexibilidad futura, terminaron produciendo beneficios que en su momento eran solo una apuesta. Y algunas restricciones no vinieron de decisiones propias en absoluto, sino de un sistema externo que cambió sus reglas sin aviso.

Haber mantenido el ownership durante toda esa evolución me permitió ver las dos caras de esas decisiones: el beneficio inmediato que hicieron posible y el costo —o el beneficio adicional— que apareció cuando el sistema tuvo que crecer.
