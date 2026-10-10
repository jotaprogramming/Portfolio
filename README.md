# Portfolio — Jerson Guerrero

A lo largo de mi carrera he liderado el diseño e integración de sistemas críticos de negocio: distribución logística, facturación electrónica, integración de ERP corporativo y plataformas de cara al cliente. El código de estos proyectos es propiedad de las empresas donde trabajé y no es público; este repositorio documenta el problema y el impacto de cada uno, sin detallar arquitectura interna ni qué tecnología se usó en qué sistema, por razones de confidencialidad y seguridad.

## Casos de estudio

- **[LogiX: evolución de una plataforma crítica de distribución logística](./case-study-logix.md)** — reconstrucción bajo restricciones, ownership prolongado en producción, evolución hacia despliegues distribuidos, y retrospectiva sobre las consecuencias de las decisiones iniciales.

## Resumen

| Proyecto | Problema | Rol | Escala |
|---|---|---|---|
| LogiX (distribución logística) | Sistema heredado que bloqueaba la operación, y su posterior crecimiento más allá del alcance original | Líder de proyecto y ownership continuo | 3 centros + 24 tiendas |
| Facturación electrónica | Migración de envío por lotes a tiempo real bajo exigencia normativa | Responsable técnico | Red de puntos de venta |
| Integración ERP | Datos inconsistentes entre sistemas internos | Diseño y desarrollo | 4 sistemas internos |
| Registro de clientes | Fricción y baja calidad de datos en la inscripción | Líder de proyecto | 24 tiendas |

## Sistemas en producción

- **Portal de clientes** (registro, fidelización, PQR): https://client.masxmenos.com.co/cliente
- **Portal de compras a proveedores** (integración con ERP Siesa): https://comprasproveedor.masxmenos.com.co/

---

## LogiX — Plataforma de Gestión Logística y Distribución

**El problema:** el sistema de distribución dejó de responder a una operación que exige alta productividad y velocidad logística. Su integración inicial con el ERP introdujo dificultades de sincronización, inconsistencias de datos y dependencias que lo hacían fallar en etapas críticas, comprometiendo la confiabilidad de la información. A esto se sumaba que el sistema había sido construido para resolver una necesidad puntual y no una proyección del negocio, lo que dificultaba su escalabilidad y su mantenimiento a largo plazo.

**Lo que hice:** lideré el proyecto en todo su ciclo de vida de desarrollo de software —análisis, diseño, desarrollo, pruebas y despliegue— y mantengo su ownership técnico desde entonces. Implementé mecanismos de concurrencia e idempotencia en los procesos de sincronización con el ERP. La arquitectura inicial fue monolítica, condicionada por restricciones de tiempo y capacidad del equipo frente a una propuesta desacoplada que habría sido preferible técnicamente.

**La evolución:** al crecer más allá de su alcance original, LogiX pasó de una instalación única a un modelo de instancias independientes que se comunican mediante APIs REST. Adopté contenedores progresivamente para las nuevas instancias, logrando aislamiento y facilitando una eventual migración a infraestructura dedicada, todavía pendiente. Refactoricé el producto para soportar operaciones con reglas distintas sobre una misma base de código y amplié su papel en el dominio: de consumir información del ERP a también originar determinadas operaciones y absorber procesos que antes obligaban a los usuarios a trabajar directamente sobre él.

**Resultado:** LogiX soporta hoy procesos de distribución en 3 centros de distribución y 24 tiendas, en producción y bajo evolución continua. En la primera entrega, la importación de datos pasó de unos cinco minutos a unos tres segundos y el despacho promedio por unidad de carga, de unos tres minutos a unos diez segundos. Son resultados históricos; posteriormente fue necesario reducir la concurrencia hacia el ERP.

📄 **[Leer el caso de estudio completo](./case-study-logix.md)** — restricciones, alternativas evaluadas, decisiones de ingeniería, límites actuales y qué habría hecho diferente.

---

## Facturación electrónica en tiempo real

**El problema:** heredé una integración de facturación electrónica incompleta, basada en envíos por lotes que debían adaptarse a la validación en tiempo real exigida por la normativa. El sistema de punto de venta almacenaba datos fiscales incompletos y trataba de manera diferente ventas, devoluciones y descuentos; esas inconsistencias provocaban errores de cálculo. Además, faltaba incorporar a la tirilla de venta el código QR y el CUFE proporcionados por el proveedor tecnológico, sin afectar el tiempo de atención en caja.

**Lo que hice:** asumí la responsabilidad técnica de la integración. Investigué la trazabilidad de los datos del punto de venta y desarrollé mecanismos de recálculo mediante PL/pgSQL para reconstruir valores fiscales a partir de registros incompletos o inconsistentes. Adapté el flujo de envío para procesar los comprobantes en tiempo real, dentro de un límite operativo de cinco segundos por factura. Integré el webservice del proveedor tecnológico de facturación electrónica, que realiza la comunicación directa con la DIAN y devuelve el CUFE ya generado y el QR codificado en base64. Mi trabajo incluyó obtener esos datos, generar y publicar la imagen del QR para su incorporación a la impresión mediante el sistema legado; no consistió en generar el CUFE ni en integrar directamente con la DIAN.

**Resultado:** la integración pasó del envío por lotes a la validación en tiempo real a través del proveedor tecnológico, con tratamiento de inconsistencias fiscales y entrega del QR/CUFE necesarios para la tirilla de venta.

---

## Integración ERP corporativo (Siesa)

**El problema:** los sistemas internos de gestión comercial, compras, ventas y distribución operaban con información desactualizada o inconsistente respecto al ERP corporativo, generando conflictos de datos entre sistemas, reprocesos manuales, y afectando la confiabilidad de la información usada en la operación diaria.

**Lo que hice:** integré el ERP corporativo con cada uno de estos sistemas, adaptando cada integración al sistema de destino. Diseñé el flujo de sincronización de datos maestros y transaccionales entre sistemas, estableciendo una única fuente de verdad para evitar conflictos de información. Sostuve estas integraciones en producción, atendiendo su soporte y evolución continua.

**Resultado:** el ERP pasó de ser una fuente de datos aislada y desactualizada para los sistemas internos, a la fuente única de verdad que sostiene los flujos críticos de inventario, proveedores y clientes.

---

## Modernización del registro de clientes

**El problema:** el proceso de inscripción a los programas de fidelización y facturación electrónica presentaba un formulario extenso, con captura de datos innecesarios y sin estandarización, lo que generaba fricción en el registro, mala calidad de datos, y duplicidad de puntos de acceso por tienda.

**Lo que hice:** lideré el rediseño del proceso de inscripción, definiendo qué datos eran realmente necesarios según la normativa fiscal vigente, y diseñando una arquitectura desacoplada entre la capa pública de cara al cliente y el sistema interno de consolidación de datos. Implementé un flujo de captura guiado paso a paso, y agregué verificación automática de identidad mediante escaneo de cédula.

**Resultado:** el proceso de inscripción pasó de un formulario extenso y con datos inconsistentes, a un flujo simplificado, estandarizado y verificado automáticamente.

---

## Tecnologías

Estas son las tecnologías en las que me he apoyado para resolver los problemas descritos. No detallo cuál corresponde a cada sistema, por razones de confidencialidad.

Python · Django · Django REST Framework · Nuxt (SSR) · JavaScript · TypeScript · Vue.js · React · Bootstrap · Tailwind CSS · PostgreSQL · SQL Server · PL/pgSQL · PL/Python · Prisma ORM · APIs REST / Web Services (JSON, HTTP/S) · Docker · n8n · Linux (Ubuntu Server) · Nginx · Apache · Git

---

## Proyectos personales / académicos

- [bolsaempleo2023](https://github.com/jotaprogramming/bolsaempleo2023) — Proyecto de grado: bolsa de empleo para egresados, desarrollado en Django.
- [planservices](https://github.com/jotaprogramming/planservices) — Mi primer proyecto real, construido en Python/Flask.
