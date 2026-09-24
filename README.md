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

**La evolución:** al crecer más allá de su alcance original, LogiX pasó de una instalación única a un modelo de instancias independientes que se comunican mediante APIs REST, y de instalaciones directas sobre el sistema operativo a despliegues contenerizados. Refactoricé el producto para soportar operaciones con reglas distintas sobre una misma base de código, y amplié su papel en el dominio: de consumir información del ERP a también originar determinadas operaciones y absorber procesos que antes obligaban a los usuarios a trabajar directamente sobre él.

**Resultado:** LogiX soporta hoy procesos de distribución en 3 centros de distribución y 24 tiendas, en producción y bajo evolución continua.

📄 **[Leer el caso de estudio completo](./case-study-logix.md)** — restricciones, alternativas evaluadas, decisiones de ingeniería, límites actuales y qué habría hecho diferente.

---

## Facturación electrónica en tiempo real

**El problema:** la integración de facturación electrónica había sido heredada de forma incompleta, operando bajo un esquema de envío por lotes que dejó de ser viable frente a un cambio normativo que exigía procesamiento en tiempo real durante cada venta. El sistema de origen presentaba inconsistencias de datos entre distintos tipos de transacción, generando errores de cálculo fiscal recurrentes y exponiendo a la operación a un riesgo de incumplimiento regulatorio. Tampoco existía la capacidad de generar ni publicar el código QR y el CUFE exigidos en la tirilla de venta, ni un tiempo de respuesta compatible con la velocidad operativa requerida en caja.

**Lo que hice:** asumí la responsabilidad técnica completa del proyecto. Analicé el origen y la trazabilidad de los datos fiscales generados por el sistema de punto de venta, y diseñé un mecanismo de recálculo para reconstruir los valores correctos a partir de datos incompletos o inconsistentes. Migré el esquema de envío de lotes a tiempo real, dentro de un tiempo límite definido para no afectar la atención en caja, e implementé la generación y publicación del código QR y el CUFE exigidos en la tirilla de venta.

**Resultado:** el sistema pasó de un esquema de envío por lotes con riesgo de incumplimiento normativo, a un procesamiento en tiempo real dentro del tiempo límite operativo exigido.

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
