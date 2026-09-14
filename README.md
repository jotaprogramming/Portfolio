# Casos de estudio — Jerson Guerrero

Resumen de proyectos que he liderado en Supermercados Más por Menos SAS (retail, 24 tiendas). El código es propiedad de la empresa y no es público; este repositorio documenta el problema, la solución y el impacto de cada proyecto.

## Sistemas en producción

- **Portal de clientes** (registro, fidelización, PQR): https://client.masxmenos.com.co/cliente
- **Portal de compras a proveedores** (integración con ERP Siesa): https://comprasproveedor.masxmenos.com.co/

---

## LogiX — Sistema de Gestión Logística y Distribución

**El problema:** el sistema de distribución dependía 100% de un ERP externo inestable, enviando traslados uno a uno de forma síncrona. Los operarios llegaban a esperar hasta 3 minutos para imprimir un sticker de despacho.

**Lo que hice:** lideré el diagnóstico (la causa raíz no era el tiempo de integración, sino la falta de concurrencia e idempotencia en la persistencia de datos) y el rediseño de la arquitectura, resolviendo además un bug crítico de duplicidad en traslados de mercancía con un mecanismo de bloqueo/compromiso secuencial.

**Resultado:** el sistema pasó de resolver fallos puntuales de comunicación a sostener toda la operación de distribución en 3 centros y 24 tiendas.

---

## Facturación electrónica en tiempo real

**El problema:** integración fiscal heredada e incompleta, con datos de origen (POS) inconsistentes entre tipos de transacción, bajo la exigencia normativa de envío en tiempo real.

**Lo que hice:** diseñé un mecanismo de recálculo en base de datos para reconstruir valores fiscales correctos a partir de datos incompletos, cumpliendo un SLA de segundos por transacción sin afectar la velocidad en caja, incluyendo la impresión de CUFE y QR en tirilla.

---

## Integración ERP corporativo (Siesa)

**El problema:** múltiples sistemas internos (gestión comercial, compras, ventas, distribución) necesitaban datos consistentes y actualizados desde el ERP corporativo.

**Lo que hice:** integré Siesa con cada uno de estos sistemas, sosteniendo en producción los flujos críticos de inventario, proveedores y clientes.

---

## Modernización del registro de clientes

**El problema:** el proceso de inscripción a fidelización y facturación electrónica era largo, pedía datos innecesarios, y no seguía el formato exigido por la normativa fiscal.

**Lo que hice:** rediseñé el flujo de captura de datos, lo reduje a lo estrictamente necesario, y agregué verificación automática de identidad mediante escaneo de cédula.

---

## Proyectos personales / académicos

- [bolsaempleo2023](https://github.com/jotaprogramming/bolsaempleo2023) — Proyecto de grado: bolsa de empleo para egresados, desarrollado en Django.
- [planservices](https://github.com/jotaprogramming/planservices) — Mi primer proyecto real, construido en Python/Flask.
