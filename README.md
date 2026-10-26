# Portfolio — Jerson Guerrero

**Software engineer specializing in enterprise systems**

![Python and Django](https://img.shields.io/badge/Python-Django-475569?style=flat-square)
![TypeScript and Node.js](https://img.shields.io/badge/TypeScript-Node.js-475569?style=flat-square)
![PostgreSQL](https://img.shields.io/badge/Datos-PostgreSQL-475569?style=flat-square)
![Docker](https://img.shields.io/badge/Infraestructura-Docker-475569?style=flat-square)
![MCP](https://img.shields.io/badge/Agent%20Tooling-MCP-475569?style=flat-square)

I develop and maintain enterprise applications in production, primarily in the retail sector. My work encompasses backend architecture, systems integration, incident diagnosis, deployment, and long-term maintenance.

I have served as the technical lead for LogiX, participated in integrations with Siesa, and developed an MCP server for agents who access business information using authorized tools. I focus on **making systems with different rules interoperable**, while preserving data consistency and operational integrity.

This repository documents **engineering decisions, constraints, results, and lessons learned**—not just technologies. The code for enterprise projects belongs to the respective organizations and is not published; internal configurations and sensitive data are omitted.

## Case Studies

- **[LogiX: Evolution of a Critical Logistics Distribution Platform](./case-study-logix.md)** — reconstruction under constraints, long-term ownership in production, evolution toward distributed deployments, and a retrospective on the consequences of initial decisions.
- **[MCP: Secure Tools for Agents on Enterprise Systems](./case-study-mcp.md)** — TypeScript/Node.js backend, integration of four systems via PostgreSQL and HTTP, access control, operational diagnostics, and fixed asset traceability.
- **[Siesa: Corporate ERP Integration for Purchasing, Distribution, and Billing](./case-study-siesa.md)** — development of integration for purchasing, configuration of services using Connekta and Siesa Integrador, and diagnosis of POS–ERP inconsistencies.
- **[Electronic Invoicing: Real-Time Reconstruction of Tax Data](./case-study-facturacion-electronica.md)** — Adaptation of a legacy integration with Conexus, resolution of tax inconsistencies, and publication of QR codes and CUFE codes on cash register receipts.

## LogiX — Logistics and Distribution Management Platform

I led the overhaul of a legacy logistics system and continue to oversee its technical evolution. I developed the first version using **Django** despite team and timeline constraints. The platform subsequently evolved to include independent instances with REST APIs, new Docker deployments, and configurable rules per operation. I implemented observability for its integration with Siesa using **Grafana**.

**Scope:** distribution for **3 centers and 24 stores**. In the first release, import time went from about five minutes to three seconds, and average dispatch time per load unit went from three minutes to ten seconds. These figures are historical and do not represent current performance.

📄 **[Case Study: LogiX](./case-study-logix.md)** — architecture, evolution, results, and limitations.

---

## Internal MCP — Tools for AI Agents

I developed and deployed an **MCP server in TypeScript/Node.js** that connects four enterprise systems using authorized query tools. It is used internally to support diagnostics for vendors, accounts, and appointments. During testing, it also allowed users to query historical transactions and link asset records across two sources.

It uses Streamable HTTP, Zod, and Docker, with read-only access via PostgreSQL or HTTP depending on the source. **It does not modify or synchronize data.**

📄 **[Case Study: MCP](./case-study-mcp.md)** — contracts, security, and operational testing.

---

## Real-Time Electronic Invoicing (Conexus)

I took over a legacy batch integration that needed to process invoices in real time, despite receiving incomplete tax data from the POS. I analyzed **more than twenty cases of inconsistencies** and developed a reconstruction and recalculation process using **PL/pgSQL**, without modifying the original records.

I integrated the web service from **Conexus**, a provider that manages direct communication with the DIAN and returns CUFE and QR codes. I adapted those results for printing and handled cases involving returns and discounts. The **five seconds per invoice** was an operational constraint, not an average measured latency.

📄 **[Case Study: Electronic Invoicing](./case-study-facturacion-electronica.md)** — tax data, integration, and constraints.

---

## Corporate ERP Integration (Siesa)

I worked on three fronts with Siesa, a leading corporate ERP system: I developed the integration of a **Django** portal for purchasing and requisitions; I configured queries, connectors, and services provided through **Connekta and Siesa Integrador** for distribution; and I diagnosed and optimized a **POS–Siesa** integration initiated by a colleague.

I maintained and enhanced these workflows after their initial delivery in 2024. SOAP communication for the configured services is handled by the integration platforms.

📄 **[Case Study: Siesa](./case-study-siesa.md)** — responsibilities and technical decisions.

---

## Modernization of the Customer Registration System

I led the effort to streamline the registration process for loyalty programs and electronic billing: reducing friction and unnecessary data, standardizing processes, separating public access from internal consolidation, and verifying identity through document scanning.

**Scope:** an operation comprising **24 stores**.

---

## Technologies and Practical Experience

I group technologies by the type of work performed; not all represent the same level of depth.

- **Backend and Applications:** Python · Django · Django REST Framework · TypeScript/Node.js (internal MCP server) · Nuxt (SSR).
- **Databases and Business Logic:** PostgreSQL · SQL Server · SQL · PL/pgSQL · PL/Python · Prisma ORM.
- **Frontend:** JavaScript · TypeScript · Vue.js · Nuxt · React · Bootstrap · Tailwind CSS.
- **Integration and automation:** REST APIs · JSON/HTTP · web services · MCP (Model Context Protocol) · Streamable HTTP · Zod · Siesa ERP · Connekta · Siesa Integrator · n8n.
- **Infrastructure and observability:** Docker · Linux (Ubuntu Server) · Nginx · Apache · Gunicorn · Grafana · Uptime Kuma (monitor configuration) · Git.

My experience with Node.js in production focuses on Nuxt and the MCP backend.

---

## Personal / Academic Projects

- [bolsaempleo2023](https://github.com/jotaprogramming/bolsaempleo2023) — Senior project: a job board for graduates, built with Django.
- [planservices](https://github.com/jotaprogramming/planservices) — My first real-world project, built with Python/Flask.
