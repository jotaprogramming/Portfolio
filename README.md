# Portfolio — Jerson Guerrero

**Ingeniero de software especializado en sistemas empresariales**

![Python y Django](https://img.shields.io/badge/Python-Django-475569?style=flat-square)
![TypeScript y Node.js](https://img.shields.io/badge/TypeScript-Node.js-475569?style=flat-square)
![PostgreSQL](https://img.shields.io/badge/Datos-PostgreSQL-475569?style=flat-square)
![Docker](https://img.shields.io/badge/Infraestructura-Docker-475569?style=flat-square)
![MCP](https://img.shields.io/badge/Agent%20Tooling-MCP-475569?style=flat-square)

Desarrollo y evoluciono aplicaciones empresariales en producción, principalmente en retail. Mi trabajo abarca arquitectura backend, integración de sistemas, diagnóstico de incidentes, despliegue y mantenimiento a largo plazo.

He liderado técnicamente LogiX, participado en integraciones con Siesa y desarrollado un servidor MCP para agentes que consultan información empresarial mediante herramientas autorizadas. Me centro en **hacer interoperables sistemas con reglas distintas**, preservando la consistencia de los datos y la operación.

Este repositorio documenta **decisiones de ingeniería, límites, resultados y aprendizajes**, no solamente tecnologías. El código de los proyectos empresariales pertenece a las organizaciones correspondientes y no se publica; se omiten configuraciones internas y datos sensibles.

## Casos de estudio

- **[LogiX: evolución de una plataforma crítica de distribución logística](./case-study-logix.md)** — reconstrucción bajo restricciones, ownership prolongado en producción, evolución hacia despliegues distribuidos, y retrospectiva sobre las consecuencias de las decisiones iniciales.
- **[MCP: herramientas seguras para agentes sobre sistemas empresariales](./case-study-mcp.md)** — backend TypeScript/Node.js, integración de cuatro sistemas mediante PostgreSQL y HTTP, control de acceso, diagnóstico operativo y trazabilidad de activos fijos.
- **[Siesa: integración de ERP corporativo en compras, distribución y facturación](./case-study-siesa.md)** — desarrollo de integración para compras, configuración de servicios mediante Connekta y Siesa Integrador, y diagnóstico de inconsistencias POS–ERP.
- **[Facturación electrónica: reconstrucción de datos fiscales en tiempo real](./case-study-facturacion-electronica.md)** — adaptación de una integración heredada con Conexus, tratamiento de inconsistencias fiscales y publicación de QR/CUFE en la impresión de caja.

## LogiX — Plataforma de Gestión Logística y Distribución

Lideré la reconstrucción de un sistema logístico heredado y mantengo su evolución técnica. Desarrollé la primera versión con **Django** bajo restricciones de equipo y plazo. Posteriormente la plataforma evolucionó a instancias independientes con APIs REST, nuevos despliegues en Docker y reglas configurables por operación. Implementé observabilidad de su integración con Siesa mediante **Grafana**.

**Alcance:** distribución para **3 centros y 24 tiendas**. En la primera entrega, la importación pasó de unos cinco minutos a tres segundos y el despacho promedio por unidad de carga, de tres minutos a diez segundos. Estas cifras son históricas y no representan el rendimiento actual.

📄 **[Caso de estudio: LogiX](./case-study-logix.md)** — arquitectura, evolución, resultados y límites.

---

## MCP interno — Herramientas para agentes de IA

Desarrollé y desplegué un servidor **MCP en TypeScript/Node.js** que conecta cuatro sistemas empresariales mediante herramientas de consulta autorizadas. Se utiliza internamente para apoyar diagnósticos de proveedores, cuentas y citas. En pruebas también permitió consultar movimientos históricos y relacionar registros de activos entre dos fuentes.

Utiliza Streamable HTTP, Zod y Docker, con acceso de solo lectura mediante PostgreSQL o HTTP según la fuente. **No modifica ni sincroniza datos.**

📄 **[Caso de estudio: MCP](./case-study-mcp.md)** — contratos, seguridad y pruebas operativas.

---

## Facturación electrónica en tiempo real (Conexus)

Asumí una integración heredada por lotes que necesitaba procesar facturas en tiempo real, pese a recibir datos fiscales incompletos del POS. Analicé **más de veinte casos de inconsistencia** y desarrollé reconstrucción y recálculo mediante **PL/pgSQL**, sin modificar los registros originales.

Integré el webservice de **Conexus**, proveedor que gestiona la comunicación directa con la DIAN y devuelve CUFE y QR. Adapté esos resultados a la impresión y traté casos de devoluciones y descuentos. Los **cinco segundos por factura** eran una restricción operativa, no una latencia media medida.

📄 **[Caso de estudio: facturación electrónica](./case-study-facturacion-electronica.md)** — datos fiscales, integración y restricciones.

---

## Integración ERP corporativo (Siesa)

Participé en tres frentes con Siesa, ERP corporativo de referencia: desarrollé la integración de un portal **Django** para compras y requisiciones; configuré consultas, conectores y servicios proporcionados mediante **Connekta y Siesa Integrador** para distribución; y diagnostiqué y optimicé una integración **POS–Siesa** iniciada por un compañero.

Mantuve y evolucioné estos flujos después de su entrega inicial en 2024. La comunicación SOAP de los servicios configurados la realizan las plataformas de integración.

📄 **[Caso de estudio: Siesa](./case-study-siesa.md)** — responsabilidades y decisiones técnicas.

---

## Modernización del registro de clientes

Lideré la simplificación del registro para fidelización y facturación electrónica: reducción de fricción y datos innecesarios, estandarización, separación de acceso público y consolidación interna, y verificación de identidad por escaneo de documento.

**Alcance:** una operación de **24 tiendas**.

---

## Tecnologías y experiencia práctica

Agrupo las tecnologías por el tipo de trabajo realizado; no todas representan el mismo nivel de profundidad.

- **Backend y aplicaciones:** Python · Django · Django REST Framework · TypeScript/Node.js (servidor MCP interno) · Nuxt (SSR).
- **Bases de datos y lógica de negocio:** PostgreSQL · SQL Server · SQL · PL/pgSQL · PL/Python · Prisma ORM.
- **Frontend:** JavaScript · TypeScript · Vue.js · Nuxt · React · Bootstrap · Tailwind CSS.
- **Integración y automatización:** APIs REST · JSON/HTTP · servicios web · MCP (Model Context Protocol) · Streamable HTTP · Zod · Siesa ERP · Connekta · Siesa Integrador · n8n.
- **Infraestructura y observabilidad:** Docker · Linux (Ubuntu Server) · Nginx · Apache · Gunicorn · Grafana · Uptime Kuma (configuración de monitores) · Git.

Mi experiencia con Node.js en producción se concentra en Nuxt y en el backend del MCP.

---

## Proyectos personales / académicos

- [bolsaempleo2023](https://github.com/jotaprogramming/bolsaempleo2023) — Proyecto de grado: bolsa de empleo para egresados, desarrollado en Django.
- [planservices](https://github.com/jotaprogramming/planservices) — Mi primer proyecto real, construido en Python/Flask.
