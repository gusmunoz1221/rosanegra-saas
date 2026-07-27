# 🌑 Rosa Negra SaaS - Plataforma de Gestión Gastronómica Multi-Tenant

##  Resumen Ejecutivo
Rosa Negra es una plataforma B2B multi-tenant de grado enterprise diseñada para digitalizar y optimizar las operaciones del sector gastronómico. Al conectar el flujo físico del salón con una capa digital de alto rendimiento, la plataforma automatiza el enrutamiento de pedidos, orquesta monitores de cocina (KDS) en tiempo real y ofrece una experiencia de pago descentralizada y sin fricciones.

Diseñada para escalar, Rosa Negra permite a los negocios operar con cero fricción, ofreciendo gestión dinámica de capacidad y liquidaciones financieras de precisión absoluta.

---

##  Arquitectura e Ingeniería Técnica

El backend está construido sobre una arquitectura orientada a funcionalidades (Feature-Based Architecture), garantizando alta cohesión y bajo acoplamiento entre los dominios de negocio.

* **Motor Financiero Zero-Trust:** Núcleo contable determinista que procesa pagos fraccionados, liquidaciones por ítem y distribución de propinas. Emplea validación quirúrgica para prevenir desajustes por redondeo decimal y garantiza la integridad contable de los combos.
* **Orquestación en Tiempo Real (STOMP/WebSockets):** Arquitectura orientada a eventos para comunicación bidireccional instantánea. Operaciones críticas como `PAYMENT_RECEIVED` o `NEW_ORDER` se transmiten en tiempo real a los monitores KDS y clientes POS, eliminando la sobrecarga de consultas (polling).
* **Concurrencia Avanzada y Bloqueos:** Implementación de bloqueo pesimista (`PESSIMISTIC_WRITE`) para transacciones financieras críticas, previniendo la corrupción de datos durante la ingesta concurrente de webhooks de pago.
* **Persistencia Optimizada (Prevención N+1):** Uso estratégico de `@BatchSize` e hidratación determinista de grafos (EntityGraphs) que asegura iteraciones O(1) en la base de datos durante el procesamiento de cargas pesadas, protegiendo el pool de conexiones HikariCP.
* **Multi-Tenancy Estricto:** Aislamiento de base de datos endurecido donde cada transacción está limitada a un `@CurrentTenantId` específico. Los eventos webhooks no coincidentes son rechazados automáticamente para evitar fugas de datos entre inquilinos.

---

##  Ecosistema e Infraestructura Modular

La plataforma es modular por diseño, permitiendo la activación dinámica de características (Feature Toggles) basándose en el nivel de suscripción de cada inquilino.

###  Motor Core
* **Menú Digital y Branding:** Catálogo público de alto rendimiento con horarios operativos dinámicos y personalización estética bajo un tema espacial minimalista.
* **Aprovisionamiento QR Universal:** Generación automática de códigos de enrutamiento institucionales mediante ZXing, eliminando la dependencia de APIs de terceros.

###  Módulos Operativos
* **KDS (Kitchen Display System):** Cola de producción en tiempo real con validación estricta de máquina de estados (`PENDIENTE`, `PREPARANDO`, `LISTO`).
* **Punto de Venta (POS) y Enrutamiento de Staff:** Acceso seguro en terminales para que los mozos abran mesas, despachen pedidos y administren la capacidad del salón.
* **Reservas Dinámicas:** Motor de asignación de mesas con validación de cupos en tiempo real.

###  Módulos Financieros y Fintech
* **Self-Service y Checkout QR Dinámico:** Generación de QR dinámicos estándar EMVCo. Los clientes pueden escanear, dividir cuentas por ítem y saldar sus deudas de forma independiente.
* **Integración Mercado Pago (OAuth):** Ingesta automatizada de webhooks para pagos In-Store. El sistema desacopla la latencia de red de las transacciones de base de datos mediante procesamiento asíncrono de eventos.
* **Analytics y Exportación de Datos:** Procesamiento por lotes nocturno automatizado (`@Scheduled`) para consolidación de ingresos y reportes en formato CSV.

---

##  Seguridad y Control de Acceso (RBAC)

La seguridad se aplica en las capas de controladores y servicios mediante Spring Security y tokens JWT sin estado (Stateless), utilizando directivas granulares `@PreAuthorize`.

| Rol | Alcance | Permisos |
|---|---|---|
| **SUPER_ADMIN** | Global | Orquestación del SaaS. Aprovisiona inquilinos, gestiona planes de suscripción y activa funciones premium. |
| **ADMIN** | Tenant | Dueño del establecimiento. Acceso total a la ingeniería del menú, gestión de personal y reportes financieros. |
| **SUPERVISOR** | Tenant | Encargado de turno. Supervisa operaciones de salón, autoriza anulaciones de pedidos y monitorea el rendimiento del personal. |
| **WAITER** | Tenant | Staff de salón. Acceso limitado a asignación de mesas, despacho de pedidos e inicio de cobros. |
| **COOK** | Tenant | Personal de cocina. Acceso restringido estrictamente a la interfaz KDS para transiciones de estados de producción. |

---

##  Stack Tecnológico

**Core y Web**
* Java 17+
* Spring Boot 3.x
* Spring Web MVC / WebSockets (STOMP)

**Persistencia y Datos**
* PostgreSQL (Compatible con Neon DB / Supabase)
* Spring Data JPA / Hibernate 6
* HikariCP

**Seguridad e Integraciones**
* Spring Security 6 y JWT
* Mercado Pago SDK (Checkout Pro e In-Store)
* Cloudinary API (Gestión de multimedia)

---

## ⚠️ Nota sobre el Estado del Repositorio
Este repositorio representa la arquitectura central de una plataforma SaaS comercial activa. Por motivos de seguridad y cumplimiento normativo, ciertas configuraciones de infraestructura, pipelines de CI/CD y módulos operativos propietarios han sido omitidos de este espejo público.
