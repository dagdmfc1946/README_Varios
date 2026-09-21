# PROYECTO: Llamados de Enfermería (NURCALL) 👩🏻‍⚕️🏥
## Arquitectura del Sistema NURCALL y Plataforma SHIPP
> **Proceso:** PCC Equipos Biomédicos — QualityMedical (FCV).

> **Documento elaborado por:** Ing. Diego Andrés García Díaz.

> **Cargo:** Profesional Dirección Técnica.

> **Versión:** 1.  

---

Este documento describe detalladamente la arquitectura de software, infraestructura de red, modelo de integración y pila tecnológica del **Sistema de Llamado de Enfermería (NURCALL)** e interfaz de gestión hospitalaria **SHIPP**.

---

## 🏗️ 1. Visión General de la Arquitectura

La arquitectura de **SHIPP / NURCALL** se basa en un modelo distribuido y orientado a servicios que conecta los dispositivos físicos de las habitaciones con los servidores centrales, el software de administración, las estaciones de enfermería y la Historia Clínica Electrónica hospitalaria (**SAHI**).

```text
                               +-----------------------------+
                               |     SAHI (Servicios FCV)    |
                               |  - Historia Clínica / M.A.  |
                               +--------------+--------------+
                                              | 
                                              | WebService (Modo Lectura)
                                              v
  +--------------------+         +----------------------------+         +----------------------------+
  |  NURCALL Hardware  |         |   SERVIDOR DE APLICACIONES |         |  BD SHIPP-MySQL (MySQL 5.5)|
  |  - Lámparas        |<------->|   - Tomcat 7 / JDK 1.7.80  |<------->|  - Eventos de llamados     |
  |  - Paneles Cama/WC | Sockets |   - WebService / WebServlet|  SQL    |  - Mismo Servidor FCVIDC   |
  +--------------------+         +--------------+-------------+         +----------------------------+
                                                |
                                                | HTTP / Sockets
                                                v
                               +------------------------------+
                               |     INTERFACES Y MONITORES   |
                               | - Monitores Estación Enferm. |
                               | - Aplicación Web Admin       |
                               | - Módulo de Estadísticas     |
                               +------------------------------+
```

---

## 🔄 2. Modelo de Integración con SAHI y Autenticación

### 2.1. Integración SHIPP ↔ SAHI (Sistema de Atención Hospitalaria Integrado)
* **Consumo de Datos (Modo Lectura):** SHIPP consulta el `WebService` de SAHI para obtener información del paciente (nombre, habitación, cama asignada). **SHIPP no escribe ni modifica datos en SAHI.**
* **Bases de Datos Independientes:** La base de datos `SHIPP-NURCALL` y la base de datos `FCVIDC` (SAHI) conviven en el mismo servidor físico/de BD, pero se mantienen estrictamente aisladas como esquemas independientes.

### 2.2. Autenticación LDAP
* El acceso a la plataforma SHIPP utiliza **LDAP** (*Lightweight Directory Access Protocol*). Permite a los médicos y enfermeras iniciar sesión con sus mismas credenciales del dominio/red hospitalaria sin duplicar usuarios.

---

## 💻 3. Componentes del Servidor de Aplicaciones y Middleware

El corazón del sistema es la capa de **Servicios Web** ejecutada sobre un servidor de aplicaciones dedicado:

1. **WebService:** Interfaz estándar (XML/JSON) empleada para la comunicación e intercambio de datos estructurados entre SAHI y SHIPP.
2. **WebServlet:** Componentes desarrollados en Java que se ejecutan sobre Tomcat. Reciben peticiones HTTP, procesan la lógica de negocio y realizan consultas/operaciones en la base de datos.
3. **Sockets (Comunicación en Tiempo Real):** Canal permanente e infinitamente abierto entre el servidor y los dispositivos/monitores. Garantiza latencia mínima e instantaneidad al transmitir un evento de alarma desde el panel de la cama hasta la pantalla de enfermería.

---

## 🖥️ 4. Interfaces de Usuario y Consumidores

* **NURCALL (Hardware):** Dispositivos físicos de habitación (Lámparas, Paneles, Cadena de Baño) conectados al servidor vía Sockets/Ethernet.
* **Monitor Estación de Enfermería:** Pantalla interactiva en el puesto asistencial (organizada por servicios como Urgencias, UCI, Piso) que muestra en tiempo real las llamadas activas.
* **Monitoreo de Estaciones:** Vista general para supervisores y control centralizado del estado de todas las estaciones del hospital.
* **Aplicación Web (Administración):** Portal administrativo (`https://www...`) para configuración de usuarios, módulos, habitaciones, camas y mapeo físico de dispositivos.
* **Módulo de Estadísticas y Reportes:** Motor de análisis que consulta `SHIPP-MySQL` para calcular tiempos de respuesta, cantidad de llamados por turno, por cama o por área.

---

## ⚙️ 5. Especificaciones Técnicas e Infraestructura

### 5.1. Servidor de Aplicaciones (Backend)
* **Sistema Operativo:** Linux Debian 8
* **Entorno de Ejecución:** JDK 1.7.80 (Java Development Kit)
* **Servidor Web / Servlet Container:** Apache Tomcat 7
* **Recursos Hardware:** Mínimo 16 GB RAM / 500 GB Almacenamiento

### 5.2. Servidor de Base de Datos
* **Sistema Operativo:** Windows Server 2012 R2 o Linux Debian 8.6
* **Motor BD:** MySQL 5.5
* **Esquema Central:** `SHIPP-MySQL` / `SHIPP-NURCALL`
* **Hardware Mínimo:** 16 GB RAM, Procesador de mínimo 4 núcleos, 500 GB en disco SAS (preferiblemente de 10.000 RPM en adelante).

### 5.3. Consideraciones Críticas de Red
* **Cableado Estructurado UTP:** **Está prohibido el uso de redes Wi-Fi** para los dispositivos NURCALL y monitores. La conexión debe ser 100% por cable UTP para garantizar latencia mínima y cero pérdida de paquetes en eventos críticos.
* **Direccionamiento IP Reservado:** Todos los dispositivos físicas de llamadas (NURCALL), lámparas y monitores de estación deben contar con **IPs fijas/reservadas** en el servidor DHCP/Router.
* **Infraestructura Dedicada:** Se requiere conmutación mediante Switches/Routers que garanticen el ancho de banda y la estabilidad del tráfico local.