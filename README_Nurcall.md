# PROYECTO: Llamados de Enfermería (NURCALL) 👩🏻‍⚕️🏥
> **Proceso:** PCC Equipos Biomédicos — QualityMedical (FCV).

> **Documento elaborado por:** Ing. Diego Andrés García Díaz.

> **Cargo:** Profesional Dirección Técnica.

> **Versión:** 1.  

---

## 📋 1. Propósito y Alcance

### Propósito
Es una solución diseñada para garantizar la seguridad del paciente, permitiéndole alertar de manera rápida, clara y en tiempo real al personal médico-asistencial sobre cualquier eventualidad o necesidad dentro del entorno hospitalario.

El sistema conecta la habitación del paciente (cabecera de cama y baño) con la estación de enfermería y el servidor hospitalario (**SHIPP**), gestionando alertas visuales y sonoras priorizadas por colores para optimizar los tiempos de respuesta del personal de salud y servicios generales.

### Objetivos
Este repositorio tiene como finalidad centralizar toda la información técnica necesaria para el ciclo de vida del producto, incluyendo:

- Diseño electrónico.
- Diseño industrial.
- Desarrollo de firmware.
- Programación de microcontroladores.
- Desarrollo de scripts y herramientas.
- Gestión de direcciones MAC.
- Archivos de impresión de etiquetas.
- Manuales e instructivos.
- Diagramas de funcionamiento.
- Información de dispositivos instalados.
- Copias de respaldo cuando sean necesarias.

---

## 🛠️ 2. Componentes del Sistema y Funcionamiento

El proyecto está conformado por 5 subsistemas principales interconectados:

1. **Lámpara de Pasillo (Nodo Central):**
   * Ubicada sobre la puerta de la habitación.
   * Emite alertas visuales (LEDs de alta intensidad) y sonoras multitono según el tipo de llamado.
   * Actúa como concentrador de comunicación entre los dispositivos de la habitación y el servidor central.

2. **Panel de Paciente:**
   * Instalado en la cabecera de la cama.
   * Cuenta con teclado de membrana, LEDs de estado, un cordón pulsador extensor para el paciente y un **botón gris de desactivación** para el personal asistencial.

3. **Panel de Baño (WC):**
   * Dispositivo estanco e inoxidable ubicado en el área del baño.
   * Posee un extensor de cadena/palanca que conmuta la señal de lectura para activar un llamado de emergencia inmediato.

4. **Timbre / Brazalete Inalámbrico:**
   * Dispositivo RF de muñeca o antebrazo que permite al paciente activar llamadas de emergencia desde cualquier punto de la habitación.

5. **Estación de Enfermería y Sistema SHIPP:**
   * Interfaz gráfica en pantalla táctil y software de monitoreo que despliega la ubicación exacta (número de habitación/cama) y el tipo de alerta activa.

---

## 🚦 3. Matriz de Alertas y Códigos de Color

| Código / Color | Tipo de Alerta | Activación | Desactivación |
| :--- | :--- | :--- | :--- |
| **🔴 Rojo** | **Llamado de Enfermería / WC** | Pulsador de cama, cadena de baño o brazalete RF. | Botón Gris (Panel Paciente). |
| **🔵 Azul** | **Código Azul (Paro Cardiorrespiratorio)** | Botón Azul en Panel Paciente. | Botón Gris (Panel Paciente). |
| **🟢 Verde** | **Cama Disponible / Habitación Lista** | Botón Verde en Panel Paciente. | Botón Gris (Panel Paciente). |
| **⚪ Blanco** | **Servicios Generales / Aseo** | Botón Blanco en Panel Paciente. | Botón Gris (Panel Paciente). |

---

## 📂 4. Estructura del Repositorio

A continuación se detalla la organización de archivos y carpetas que conforman este proyecto (prácticamente el contenido de todas las carpetas quedo dentro de archivos **.zip**):

```text
/
├── 1.Diag. Flujo/
│   └── Diagramas de flujo del sistema.

├── 2.Diseño Electrónico/
│   └── Esquemáticos, PCB, librerías y archivos electrónicos.

├── 3.Diseño Industrial/
│   └── Modelos mecánicos, planos y archivos CAD.

├── 4.MAC/
│   └── Administración de direcciones MAC de los dispositivos.

├── 5.Scripts_(Codigos)/
│   └── Scripts, utilidades y herramientas de desarrollo.

├── 6.microSD_Grabadas/
│   └── Archivos utilizados para la programación de tarjetas microSD.

├── 7.microSD_PRUEBAS/
│   └── Archivos destinados a validaciones y pruebas.

├── 8.Impresion_etiquetas/
│   └── Plantillas y archivos para impresión de etiquetas.

├── 9.Manuales_e_Instructivos/
│   └── Manuales técnicos y documentación del sistema.

├── 10.Dispositivos_Instalados/
│   └── Información relacionada con equipos instalados.

└── Backup_PC_Gustavo/
    └── Copias de respaldo históricas del proyecto.
```

**NOTA:** Para poder hacer uso/visualización del proyecto, en primer lugar debe clonar el respositorio, seguidamente debe descomprimir cada uno de los archivos **.zip** faltantes para compeltar el contenido de todas las carpetas.

---

## ⚙️ 5. Especificaciones Técnicas Resumidas

* **Comunicación Local:** Bus RS-485 (Topología Maestro-Esclavo).
* **Comunicación de Red:** TCP/IP (Ethernet Cat 6A) para integración con SHIPP / SAHI.
* **Alimentación:** Red eléctrica hospitalaria regulada / crítica (UPS).
---

## 🌐 6. Arquitectura General del Sistema

El sistema **NURCALL** está compuesto por nodos de hardware interconectados en topología industrial mediante el bus de comunicación **RS-485** (Protocolo Maestro-Esclavo) e integrados a la red Ethernet del hospital vía TCP/IP para la gestión de datos centralizada.

```
                  +-----------------------------------+
                  |      Servidor Hospitalario /      |
                  |     Estación Enfermería (SHIPP)   |
                  +----------------- RAIL -----------------+
                                    | Ethernet (TCP/IP)
                                    v
                       +-------------------------+
                       | LÁMPARA DE PASILLO      | (Maestro Local)
                       | - Arduino Mega 2560     |
                       | - Alertas Visual/Sonora |
                       | - Módulo microSD / RTC  |
                       +------------+------------+
                                    |
                                    | Bus RS-485
                  +-----------------+-----------------+
                  |                                   |
                  v                                   v
      +-----------------------+           +-----------------------+
      | PANEL PACIENTE (cama) |           | PANEL PACIENTE (cama) |
      | - ATmega328-AU        |           | - ATmega328-AU        |
      | - Teclado / LEDs      |           | - Teclado / LEDs      |
      | - Cordón Pulsador     |           | - Cordón Pulsador     |
      +-----------+-----------+           +-----------+-----------+
                  |                                   |
                  v                                   v
      +-----------------------+           +-----------------------+
      |    PANEL BAÑO (WC)    |           |    PANEL BAÑO (WC)    |
      | - Extensor Palanca    |           | - Extensor Palanca    |
      +-----------------------+           +-----------------------+
```

---

## 🎛️ 7. Componentes Principales del Hardware

### 3.1. Lámpara de Pasillo (Nodo Central de Habitación)
* **Función:** Actúa como controlador maestro de la habitación y gateway de comunicación con el servidor.
* **Componentes clave:**
  * Microcontrolador principal **Arduino Mega 2560** (ATmega2560).
  * Shield Ethernet para comunicación TCP/IP con servidor SHIPP.
  * Módulo RS-485 para bus de campo con paneles de paciente.
  * Tarjetas LED de alta luminosidad (4 colores: Rojo, Azul, Verde, Blanco).
  * Módulo de audio y parlante para alertas sonoras multitono.
  * Módulo RTC (Reloj de Tiempo Real) y lector MicroSD para almacenamiento local y funcionamiento en modo autónomo/degradado (*Sistema Independiente*).

### 3.2. Panel Paciente
* **Función:** Interfaz de interacción directa en la cabecera de la cama del paciente.
* **Componentes clave:**
  * Microcontrolador **ATmega328**.
  * Teclado de membrana con 4 botones y LEDs indicadores integrados.
  * Entrada para cordón pulsador extensor (Alarma roja) con detección de desconexión o fallo de cable.
  * Módulo de recepción para pulsera / timbre inalámbrico RF.
  * Transceptor RS-485 para comunicación con la Lámpara de Pasillo.

### 3.3. Panel de Baño (WC)
* **Función:** Dispositivo de emergencia estanco e inoxidable para áreas húmedas.
* **Mecanismo:** Pulsador de cadena / extensor de palanca con señal de lectura y GND en reposo. Al halar la cadena, conmuta la señal provocando la activación inmediata del código de baño.

### 3.4. Pulsera / Timbre Inalámbrico
* Dispositivo portátil de antebrazo para que el paciente active el llamado desde cualquier punto de la habitación.

---

## 🏭 8. Flujo Operativo de Producción (POE)

El proceso de producción estandarizado se divide en las siguientes etapas clave:

1. **Gestión Administrativa y Materiales:** Programación de PCC, orden de producción (OP), solicitud, inspección y limpieza de Materia Prima (MP).
2. **Ensamble y Soldadura PCB:**
   * Soldadura THT/SMD en tarjeta Lámpara Pasillo (transistores, integrados de potencia, RS-485, conectores JST XH, shield Ethernet).
   * Soldadura THT/SMD en tarjeta Panel Paciente (ATmega328, oscilador, chip LINX RF, conectores RJ45 y membrana).
   * Armado de cableado aéreo RJ45 para Panel de Baño.
3. **Carga de Firmware:**
   * Programación de Arduino Mega via Arduino IDE (2 códigos principales).
   * Programación de ATmega328 via MikroC / Microchip Studio y configuración de *fuses*.
4. **Ensamblaje Mecánico:** Montaje de carcasas, difusores, tarjetas LED, parlante, fusible y cableado interno.
5. **Pruebas de Funcionamiento:**
   * Validación de entradas/salidas, bus RS-485, red Ethernet y conmutación de alarmas.
   * Prueba específica de ciclo de vida e integridad de señal para código de baño (WC).

---

# Estado del proyecto

El contenido de este repositorio se encuentra finalmente cargado. Conforme avance el proyecto puede que se incorporen nuevos documentos técnicos, diagramas, manuales, firmware, procedimientos, registros relacionados o actualizaciones a los Llamados de Enfermería (NURCALL).