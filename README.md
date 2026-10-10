# Portfolio — Jerson Guerrero

**Ingeniero de software | Python/Django · Integración de sistemas empresariales · TypeScript/Node.js**

Trabajo en el desarrollo y evolución de sistemas empresariales en producción, principalmente en entornos de retail. Desde 2022 he participado en proyectos de logística, compras, facturación y plataformas de cara al cliente, asumiendo responsabilidades de diseño, implementación, diagnóstico de incidentes y mantenimiento a largo plazo.

Mi especialidad es **conectar sistemas que tienen reglas, datos y restricciones diferentes**, sin perder de vista la operación que depende de ellos. He liderado técnicamente la evolución de LogiX, trabajado en integraciones con el ERP Siesa y desarrollado un servidor MCP interno en TypeScript/Node.js para que agentes autorizados consulten y correlacionen datos de varias aplicaciones.

**Áreas de trabajo:** arquitectura y evolución de aplicaciones Django; integraciones de ERP, APIs y sistemas heredados; diseño de datos y lógica SQL; backend TypeScript/Node.js y herramientas MCP; despliegue, diagnóstico de producción y observabilidad con Docker, Linux y Grafana.

Este repositorio documenta **decisiones de ingeniería, límites, resultados y aprendizajes**, no solamente tecnologías. El código de los proyectos empresariales pertenece a las organizaciones correspondientes y no se publica; se omiten configuraciones internas y datos sensibles.

## Casos de estudio

- **[LogiX: evolución de una plataforma crítica de distribución logística](./case-study-logix.md)** — reconstrucción bajo restricciones, ownership prolongado en producción, evolución hacia despliegues distribuidos, y retrospectiva sobre las consecuencias de las decisiones iniciales.
- **[MCP: herramientas seguras para agentes sobre sistemas empresariales](./case-study-mcp.md)** — backend TypeScript/Node.js, integración de cuatro sistemas mediante PostgreSQL y HTTP, control de acceso, diagnóstico operativo y trazabilidad de activos fijos.
- **[Siesa: integración de ERP corporativo en compras, distribución y facturación](./case-study-siesa.md)** — desarrollo de integración para compras, configuración de servicios mediante Connekta y Siesa Integrador, y diagnóstico de inconsistencias POS–ERP.

## Resumen

| Proyecto | Problema | Rol | Escala |
|---|---|---|---|
| LogiX (distribución logística) | Sistema heredado que bloqueaba la operación, y su posterior crecimiento más allá del alcance original | Líder de proyecto y ownership continuo | 3 centros + 24 tiendas |
| MCP interno (herramientas para agentes) | Consultas de soporte y trazabilidad de activos fragmentadas entre aplicaciones | Diseño, desarrollo y despliegue | 4 sistemas integrados; uso interno por el equipo |
| Facturación electrónica | Migración de envío por lotes a tiempo real bajo exigencia normativa | Responsable técnico | Red de puntos de venta |
| Integración ERP Siesa | Sincronización y consistencia de datos entre ERP y sistemas operativos | Desarrollo, configuración y optimización según el frente | Compras, distribución y facturación |
| Registro de clientes | Fricción y baja calidad de datos en la inscripción | Líder de proyecto | 24 tiendas |

## Sistemas en producción

- **Portal de clientes** (registro, fidelización, PQR): https://client.masxmenos.com.co/cliente
- **Portal de compras a proveedores** (integración con ERP Siesa): https://comprasproveedor.masxmenos.com.co/

---

## LogiX — Plataforma de Gestión Logística y Distribución

**El problema:** el sistema de distribución dejó de responder a una operación que exige alta productividad y velocidad logística. Su integración inicial con el ERP introdujo dificultades de sincronización, inconsistencias de datos y dependencias que lo hacían fallar en etapas críticas, comprometiendo la confiabilidad de la información. A esto se sumaba que el sistema había sido construido para resolver una necesidad puntual y no una proyección del negocio, lo que dificultaba su escalabilidad y su mantenimiento a largo plazo.

**Lo que hice:** lideré el proyecto en todo su ciclo de vida de desarrollo de software —análisis, diseño, desarrollo, pruebas y despliegue— y mantengo su ownership técnico desde entonces. Implementé mecanismos de concurrencia e idempotencia en los procesos de sincronización con el ERP. La arquitectura inicial fue monolítica, condicionada por restricciones de tiempo y capacidad del equipo frente a una propuesta desacoplada que habría sido preferible técnicamente.

**La evolución:** al crecer más allá de su alcance original, LogiX pasó de una instalación única a un modelo de instancias independientes que se comunican mediante APIs REST. Adopté contenedores progresivamente para las nuevas instancias, logrando aislamiento y facilitando una eventual migración a infraestructura dedicada, todavía pendiente. Refactoricé el producto para soportar operaciones con reglas distintas sobre una misma base de código y amplié su papel en el dominio: de consumir información del ERP a también originar determinadas operaciones y absorber procesos que antes obligaban a los usuarios a trabajar directamente sobre él.

**Observabilidad:** implementé un tablero en Grafana para dar seguimiento a las integraciones LogiX–Siesa mediante información registrada por la aplicación: volumen de solicitudes, respuestas ausentes, errores y latencias. Se utiliza para analizar el comportamiento de una dependencia crítica con evidencia operativa.

**Resultado:** LogiX soporta hoy procesos de distribución en 3 centros de distribución y 24 tiendas, en producción y bajo evolución continua. En la primera entrega, la importación de datos pasó de unos cinco minutos a unos tres segundos y el despacho promedio por unidad de carga, de unos tres minutos a unos diez segundos. Son resultados históricos; posteriormente fue necesario reducir la concurrencia hacia el ERP.

📄 **[Leer el caso de estudio completo](./case-study-logix.md)** — restricciones, alternativas evaluadas, decisiones de ingeniería, límites actuales y qué habría hecho diferente.

---

## MCP interno — Herramientas para agentes de IA y consulta multi-sistema

**El problema:** diagnosticar incidencias operativas exigía consultar distintas aplicaciones y reconstruir relaciones entre sus datos. Después surgió otra necesidad: rastrear activos fijos cuya historia estaba fragmentada entre sistemas históricos y actuales, sin saber de antemano en cuáles había registros.

**Lo que hice:** diseñé y desplegué un servidor MCP (Model Context Protocol) en TypeScript/Node.js, con transporte Streamable HTTP, validación de contratos mediante Zod y ejecución en Docker. Actualmente expone herramientas de consulta sobre **cuatro sistemas empresariales**. Los tres sistemas originales se consultan mediante PostgreSQL con usuarios de solo lectura y permisos mínimos; la nueva aplicación de activos se integra mediante HTTP `GET` y un token Bearer con permisos de lectura concedidos en el sistema de origen. Los agentes usan herramientas delimitadas, sin recibir las credenciales de los sistemas.

**Resultado:** el servicio se utiliza internamente por integrantes del equipo desde distintos clientes compatibles con MCP. Además de apoyar incidencias de cuentas, proveedores y citas, ahora permite consultar activos, facturas y traslados históricos. En pruebas con agentes se consultaron movimientos y se correlacionó información entre el histórico y la aplicación vigente, incluyendo casos donde un activo estaba presente en un sistema pero no en el otro. La versión actual es **exclusivamente de consulta**: no modifica ni sincroniza registros, y queda pendiente integrar otra fuente utilizada en la gestión de activos.

📄 **[Leer el caso de estudio completo](./case-study-mcp.md)** — decisiones técnicas, controles de acceso y evolución hacia la trazabilidad de activos.

---

## Facturación electrónica en tiempo real

**El problema:** heredé una integración de facturación electrónica incompleta, basada en envíos por lotes que debían adaptarse a la validación en tiempo real exigida por la normativa. El sistema de punto de venta almacenaba datos fiscales incompletos y trataba de manera diferente ventas, devoluciones y descuentos; esas inconsistencias provocaban errores de cálculo. Además, faltaba incorporar a la tirilla de venta el código QR y el CUFE proporcionados por el proveedor tecnológico, sin afectar el tiempo de atención en caja.

**Lo que hice:** asumí la responsabilidad técnica de la integración. Investigué la trazabilidad de los datos del punto de venta y desarrollé mecanismos de recálculo mediante PL/pgSQL para reconstruir valores fiscales a partir de registros incompletos o inconsistentes. Adapté el flujo de envío para procesar los comprobantes en tiempo real, dentro de un límite operativo de cinco segundos por factura. Integré el webservice del proveedor tecnológico de facturación electrónica, que realiza la comunicación directa con la DIAN y devuelve el CUFE ya generado y el QR codificado en base64. Mi trabajo incluyó obtener esos datos, generar y publicar la imagen del QR para su incorporación a la impresión mediante el sistema legado; no consistió en generar el CUFE ni en integrar directamente con la DIAN.

**Resultado:** la integración pasó del envío por lotes a la validación en tiempo real a través del proveedor tecnológico, con tratamiento de inconsistencias fiscales y entrega del QR/CUFE necesarios para la tirilla de venta.

---

## Integración ERP corporativo (Siesa)

**El problema:** Siesa ya era el ERP corporativo y la fuente de referencia para los procesos comerciales y transaccionales. El desafío era integrar las aplicaciones de compras, distribución y facturación, cuyos flujos, transformaciones y datos operativos podían quedar desactualizados o inconsistentes respecto al ERP.

**Lo que hice:** desarrollé la adaptación e integración de un portal de proveedores en Django para consultar sugeridos de compra y gestionar órdenes y requisiciones. En el frente de distribución configuré consultas, conectores y servicios web suministrados para su consumo mediante **Connekta y Siesa Integrador**; estas plataformas gestionan la comunicación con los servicios SOAP de Siesa, que yo no desarrollé. Posteriormente revisé y optimicé una integración **POS–Siesa** cuyo desarrollo inicial había realizado otro integrante del equipo, investigando diferencias de cálculo y cuentas por cobrar.

**La evolución:** mantuve y optimicé los flujos tras la salida inicial a producción en 2024. En compras, la consulta de sugeridos evolucionó hacia una carga dinámica por proveedor. Más adelante, determinadas responsabilidades de distribución pasaron del portal de compras a LogiX.

**Resultado:** las aplicaciones operativas pudieron intercambiar información con el ERP y evolucionar sus flujos, mientras el diagnóstico y las correcciones contribuyeron a tratar inconsistencias entre los sistemas consumidores y Siesa. No existe una medición consolidada de reducción de errores o tiempos que pueda atribuir exclusivamente a este proyecto.

📄 **[Leer el caso de estudio completo](./case-study-siesa.md)** — responsabilidades, configuración de integración, decisiones, evolución y límites del sistema.

---

## Modernización del registro de clientes

**El problema:** el proceso de inscripción a los programas de fidelización y facturación electrónica presentaba un formulario extenso, con captura de datos innecesarios y sin estandarización, lo que generaba fricción en el registro, mala calidad de datos, y duplicidad de puntos de acceso por tienda.

**Lo que hice:** lideré el rediseño del proceso de inscripción, definiendo qué datos eran realmente necesarios según la normativa fiscal vigente, y diseñando una arquitectura desacoplada entre la capa pública de cara al cliente y el sistema interno de consolidación de datos. Implementé un flujo de captura guiado paso a paso, y agregué verificación automática de identidad mediante escaneo de cédula.

**Resultado:** el proceso de inscripción pasó de un formulario extenso y con datos inconsistentes, a un flujo simplificado, estandarizado y verificado automáticamente.

---

## Tecnologías y experiencia práctica

Agrupo las tecnologías por el tipo de trabajo realizado; no todas representan el mismo nivel de profundidad.

- **Backend y aplicaciones:** Python · Django · Django REST Framework · TypeScript/Node.js (servidor MCP interno) · Nuxt (SSR).
- **Bases de datos y lógica de negocio:** PostgreSQL · SQL Server · SQL · PL/pgSQL · PL/Python · Prisma ORM.
- **Frontend:** JavaScript · TypeScript · Vue.js · Nuxt · React · Bootstrap · Tailwind CSS.
- **Integración y automatización:** APIs REST · JSON/HTTP · servicios web · MCP (Model Context Protocol) · Streamable HTTP · Zod · Siesa ERP · Connekta · Siesa Integrador · n8n.
- **Infraestructura y observabilidad:** Docker · Linux (Ubuntu Server) · Nginx · Apache · Gunicorn · Grafana · Uptime Kuma (configuración de monitores) · Git.

Mi experiencia con Node.js en producción se concentra en Nuxt y en el backend del MCP; mi trabajo con Connekta y Siesa Integrador es de configuración e integración, no de desarrollo de sus servicios SOAP.

---

## Proyectos personales / académicos

- [bolsaempleo2023](https://github.com/jotaprogramming/bolsaempleo2023) — Proyecto de grado: bolsa de empleo para egresados, desarrollado en Django.
- [planservices](https://github.com/jotaprogramming/planservices) — Mi primer proyecto real, construido en Python/Flask.
