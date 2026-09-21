# ANÁLISIS DE RENDIMIENTO, ARQUITECTURA TÉCNICA Y ESTRATEGIA DE MIGRACIÓN: SISTEMA DE LLAMADO DE ENFERMERÍA (QUALITYMEDICAL - FCV)

## 1. RESUMEN EJECUTIVO

El presente documento consolida la evaluación técnica realizada sobre la arquitectura del hardware y software empleado en el **Sistema de Llamado de Enfermería (Lámparas de Pasillo / Estaciones de Enfermería)** desarrollado por **QualityMedical - Fundación Cardiovascular de Colombia (FCV)**.

A partir del análisis de los archivos de configuración y firmware para el **Arduino Mega 2560** (con *Shield Ethernet*), se identificaron los cuellos de botella operativos que provocan eventos de congelamiento (*hanging*), fallas de conexión con los servidores de backend y pérdida intermitente de paquetes en despliegues con más de 380 dispositivos. Asimismo, se establecen los planes de optimización a corto plazo (vía software) y la estrategia de migración tecnológica a mediano/largo plazo empleando la plataforma **ESP32**.

---

## 2. DIAGNÓSTICO DE MEMORIA: IMPACTO DEL *STACK OVERFLOW* Y COMENTARIOS DEL CÓDIGO

### 2.1. Mitología vs. Realidad de los Comentarios en C/C++
* **Pregunta frecuente:** ¿Los comentarios extensos en el código fuente o en las 14 librerías importadas contribuyen al *Stack Overflow* o reducen la eficiencia del microcontrolador?
* **Dictamen técnico:** **No.** El proceso de compilación a través de la herramienta `avr-gcc` realiza una fase de preprocesamiento en la cual **se eliminan el 100% de los comentarios** (`//` y `/* ... */`). El archivo ejecutable resultante (`.hex`) cargado en el ATmega2560 no retiene metadatos de texto explicativo. Por lo tanto, no ocupan espacio en la memoria Flash ni en la SRAM.

### 2.2. Anatomía de la Memoria SRAM en el ATmega2560 (8 KB)
La consola del **Arduino IDE** reporta los siguientes valores durante la compilación del firmware `codigo_lampara.ino`:

* **SRAM Utilizada:** 6,466 bytes (78\%)
* **SRAM Libre:** 1,726 bytes (22\%)

La estructura de memoria dinámica se distribuye en tres regiones:

1. **Variables Globales / Estáticas (78% de ocupación):** Reservada desde el arranque (*boot*). Contiene variables globales, *buffers* de librerías de red y, principalmente, **cadenas de texto literales (strings con comillas dobles `""`)**.
2. **Heap (Memoria Dinámica):** Asignación estática/dinámica reservada para estructuras de red y objetos dinámicos.
3. **Stack (Pila de Ejecución):** Espacio donde se almacenan variables locales de funciones, parámetros transferidos y direcciones de retorno de ejecución.

```
+-------------------------------------------------------------------+
|               MEMORIA SRAM TOTAL ATMEGA2560 (8192 Bytes)          |
+------------------------------------+------------------------------+
|   Variables Globales y Cadenas     |      Stack + Heap Libre      |
|   de Texto Literales (78%)         |           (22%)              |
|   6,466 Bytes                       |         1,726 Bytes          |
+------------------------------------+------------------------------+
                                     ^
                                     |---> Riesgo de Colisión (Stack Overflow)
```

### 2.3. Mecanismo del *Stack Overflow*
Dado que solo se cuenta con **1,726 bytes libres**, cuando el sistema recibe múltiples llamadas de enfermería o peticiones simultáneas desde la red:
* Las funciones anidadas incrementan el tamaño de la Pila (*Stack*).
* Si el *Stack* sobrepasa los 1,726 bytes, se produce una **colisión directa contra la memoria de variables globales**.
* **Efecto en producción:** Corrupción de punteros de memoria, reinicio intempestivo (*watchdog reset*) o congelamiento total del Arduino Mega.

---

## 3. OPTIMIZACIÓN A CORTO PLAZO VÍA SOFTWARE

Para mitigar los problemas de *Stack Overflow* sin costo de hardware ni cambios de esquemático, se deben implementar las siguientes mejores prácticas:

### 3.1. Implementación de la `Macro F()`
Por defecto, toda cadena de texto en Arduino (ejemplo: `Serial.print("Error de conexion");`) se copia automáticamente a la memoria SRAM al iniciar el dispositivo.

* **Solución:** Envolver todas las cadenas constantes presentes en `codigo_lampara.ino` y en las librerías bajo la macro `F()`.

```cpp
// ❌ INCORRECTO: Ocupa memoria RAM (SRAM)
Serial.println("Inicializando modulo de red y servidor Web...");

// ✅ CORRECTO: Fuerza el almacenamiento en Memoria FLASH (donde hay +70% disponible)
Serial.println(F("Inicializando modulo de red y servidor Web..."));
```

**Beneficios esperados:**
* Reducción del uso de SRAM global de un **78% a un ~45-50%**.
* Incremento de la Pila de Ejecución a **+4,000 bytes libres**, eliminando los colapsos por *Stack Overflow*.

### 3.2. Eliminación de Funciones Bloqueantes
* Reemplazar cualquier instancia de `delay()` por el uso de temporizadores no bloqueantes mediante `millis()`.
* Mantener la tasa de ejecuciones por segundo del `loop()` a su máximo rendimiento para evitar perder tramas en el controlador Ethernet.

---

## 4. ANÁLISIS DE LIMITACIONES DEL HARDWARE ACTUAL (ARDUINO MEGA + SHIELD ETHERNET)

Ante un despliegue de **380+ dispositivos activos con proyección a 480+ unidades**, la adición de memorias RAM externas (como ICs SRAM SPI 23LC1024) **no resuelve el problema estructural**.

### 4.1. Cuellos de Botella de la Arquitectura Existente
1. **Limitación de Sockets en Hardware Ethernet (W5100/W5500):** La *Shield* posee entre **4 y 8 sockets TCP/IP harcodeados** por hardware. Si más de 8 dispositivos intentan comunicarse simultáneamente con el microcontrolador, las tramas entrantes son descartadas por falta de sockets libres.
2. **Procesador Mononúcleo de 8 bits a 16 MHz:** El ATmega2560 procesa una instrucción a la vez de forma secuencial. Actividades como la lectura de la tarjeta microSD, actualización de pantalla LCD o procesamiento de librerías I2C bloquean la atención de paquetes de red durante milisegundos críticos.

---

## 5. ESTRATEGIA DE MIGRACIÓN TECNOLÓGICA: ADOPCIÓN DE ESP32

Para garantizar un estándar de disponibilidad hospitalaria de grado industrial, se establece la migración hacia la plataforma **ESP32 (32-Bit Dual Core)**.

### 5.1. Comparativa Técnica de Plataformas

| Parámetro Técnico | Arduino Mega 2560 (Actual) | ESP32 (Propuesto) | Factor de Mejora |
| :--- | :--- | :--- | :--- |
| **Arquitectura** | 8 bits AVR | 32 bits Xtensa LX6 Dual-Core | Multitarea Real |
| **Frecuencia de Reloj** | 16 MHz | 240 MHz | **15x más rápido** |
| **Memoria SRAM** | 8 KB (0.008 MB) | 520 KB | **65x mayor capacidad** |
| **Memoria Flash** | 256 KB | 4 MB / 8 MB / 16 MB | **16x a 64x mayor** |
| **Sistema Operativo** | Ninguno (Secuencial) | FreeRTOS (RTOS nativo) | Determinismo de eventos |
| **Conectividad** | SPI externo (Shield) | MAC Ethernet nativo, Wi-Fi, BT | Integrado |

### 5.2. Arquitectura Multinúcleo con FreeRTOS
La adopción de ESP32 permite separar las responsabilidades del sistema en dos núcleos independientes:
* **Core 0 (Procesador de Comunicaciones):** Consagrado exclusivamente al manejo de la pila TCP/IP, cliente MQTT, peticiones HTTP y conectividad de red.
* **Core 1 (Procesador de Aplicación):** Dedicado al control de entradas/salidas (pulsadores de llamados), gestión de la tarjeta microSD, control de pantalla y audio.

```
                      +---------------------------------------+
                      |         ESP32 DUAL-CORE (240 MHz)      |
                      +-------------------+-------------------+
                                          |
                 +------------------------+------------------------+
                 |                                                 |
                 v                                                 v
    +--------------------------+                      +--------------------------+
    |     CORE 0: RED / OS     |                      |   CORE 1: APLICACIÓN     |
    +--------------------------+                      +--------------------------+
    | * Pila TCP/IP            |                      | * Lectura de Pulsadores  |
    | * Cliente MQTT / Web     |                      | * Manejo microSD / RTC   |
    | * Conexión Servidor      |                      | * Control Pantalla / LCD |
    +--------------------------+                      +--------------------------+
```

---

## 6. IMPACTO DEL INFRAESTRUCTURA DE SERVIDORES Y BACKEND

Los eventos de trabas o cuellos de botella reportados al escalar el número de dispositivos no corresponden únicamente al microcontrolador local, sino también al desempeño de la infraestructura central.

### 6.1. Efecto Dominó por Saturación del Servidor
1. **Aumento de Latencia y Timeouts:** Si la CPU o RAM del servidor se saturan, la respuesta de confirmación a los llamados se demora. El cliente (Arduino/ESP32) mantiene conexiones abiertas esperando respuesta, agotando los sockets locales.
2. **Bloqueo por Modelo HTTP Polling:** Realizar peticiones HTTP periódicas desde 380+ dispositivos genera un consumo masivo de recursos en el servidor Web/Base de Datos.

### 6.2. Recomendaciones para la Infraestructura de Servidores
1. **Migración de Protocolo (HTTP -> MQTT):**
   * Reemplazar las peticiones HTTP tradicionales por el protocolo **MQTT** utilizando un *broker* de alto rendimiento como **Mosquitto** o **EMQX**.
   * Disminución del 80% en el consumo de ancho de banda y reducción sustancial de carga en la CPU del servidor.
2. **Dimensionamiento de Servidor Backend:**
   * **Monitoreo de Recursos:** Verificar el consumo de CPU, RAM y conexiones TCP concurrentes en el servidor durante picos de operación.
   * **Base de Datos:** Implementar índices apropiados en las tablas de almacenamiento de eventos de llamados de enfermería.

---

## 7. FICHA DE CONTROL DE DOCUMENTO Y AUTORÍA

| **INFORMACIÓN DE CONTROL DOCUMENTAL** | **DATOS INSTITUCIONALES** |
| :--- | :--- |
| **Título del Documento:** | Análisis de Rendimiento, Arquitectura y Migración Técnica |
| **Proyecto:** | Llamados de Enfermería (NURCALL) |
| **Entidad / Empresa:** | QualityMedical / Fundación Cardiovascular de Colombia (FCV) |
| **Autor:** | Ing. Diego Andrés García Díaz |
| **Cargo:** | Profesional de Dirección Técnica |
| **Fecha de Elaboración:** | 1 de agosto de 2026 |
| **Versión del Documento:** | 1.0 |