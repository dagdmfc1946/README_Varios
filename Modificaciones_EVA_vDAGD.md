# Bitácora de cambios y guía de compilación
**Proyecto:** SONÓMETRO EVA  
**Archivo fuente:** `eva_4_2.ino`  
**Archivo modificado:** `eva_5_0.ino`
**ID del dispositivo:** `SONO-XXXX`  
**PC de trabajo:** `diegogarcia`  
**Fecha:** Julio 2026

---

## Contexto

El código `eva_4_2.ino` ha sido usado en producciones previas del dispositivo y se considera estable. El objetivo de esta sesión fue verificar que el código puede compilarse y cargarse desde un **PC nuevo**, para validar que ese equipo puede ser usado en futuros procesos de producción.

---

## Problema 1 — Error de compilación BLE: `std::string` vs `String`

### Causa
El PC nuevo tiene instalado el **core ESP32 3.3.11** (rama 3.x). En versiones anteriores del core (rama 1.x), el método `BLECharacteristic::getValue()` retornaba `std::string`. En el core 3.x este método fue modificado para retornar `String` (tipo Arduino), lo que genera un error de conversión de tipos al compilar.

### Error mostrado por el IDE
```
error: conversion from 'String' to non-scalar type 'std::string'
{aka 'std::__cxx11::basic_string<char>'} requested
```
Aparecía en las líneas correspondientes a los 4 callbacks BLE:
- `WifiCallback::onWrite` 
- `AlertsCallback::onWrite`
- `StatisticsCallback::onWrite`
- `ServerCallback::onWrite`

### Cambio aplicado al código
Se modificaron **únicamente 4 líneas** (una por cada callback BLE), cambiando el tipo de dato de la variable local que recibe el valor BLE:

| Línea | Antes | Después |
|---|---|---|
| 572 | `std::string wifiData = overallFeature -> getValue();` | `String wifiData = overallFeature -> getValue();` |
| 595 | `std::string alarmData = overallFeature -> getValue();` | `String alarmData = overallFeature -> getValue();` |
| 618 | `std::string averageData = overallFeature -> getValue();` | `String averageData = overallFeature -> getValue();` |
| 641 | `std::string serverData = overallFeature -> getValue();` | `String serverData = overallFeature -> getValue();` |

> El resto del código en cada callback (`.length()`, `.c_str()`, los bucles de debug) no requirió ningún cambio porque la clase `String` de Arduino implementa los mismos métodos.

El archivo resultante con estos cambios aplicados es **`eva_4_2.ino`** (versión modificada).

---

## Problema 2 — Error de compilación: librería `AnalogWrite_ESP32` incompatible

### Causa
La librería `AnalogWrite_ESP32` instalada en el PC usaba internamente las funciones `ledcSetup()` y `ledcAttachPin()`, que existían en el core ESP32 1.x pero fueron **eliminadas en el core 3.x** y reemplazadas por `ledcAttach()`. Al compilar, la librería externa fallaba con errores dentro de su propio código fuente.

### Error mostrado por el IDE
```
error: 'ledcSetup' was not declared in this scope
error: 'ledcAttachPin' was not declared in this scope; did you mean 'ledcAttach'?
```

### Acción realizada
Se **desinstalaron** las siguientes librerías desde el Gestor de Bibliotecas del IDE:
- `AnalogWrite_ESP32` (incompatible con core 3.x)
- `ArduinoBLE` (librería para Arduino Nano 33, no compatible con ESP32 — generaba conflicto con la librería BLE nativa del core)

> En el core ESP32 3.x, `analogWrite()` ya está disponible de forma nativa sin necesidad de ninguna librería externa.

---

## Estado final del código modificado

El archivo `eva_4_2.ino` con los cambios del **Problema 1** aplicados compila correctamente bajo:
- **Arduino IDE:** versión reciente (2.x)
- **Core ESP32:** 3.3.11
- **Librerías desinstaladas:** `AnalogWrite_ESP32`, `ArduinoBLE`

Los únicos cambios respecto al código original son las 4 líneas de `std::string` → `String` descritas arriba.

---

## Instrucciones para compilar y cargar el código SIN modificar el código original

Si la política del proyecto **no permite modificar el código fuente**, la solución es usar el **core ESP32 versión 1.0.6** (rama 1.x), que es la versión con la que el código fue desarrollado originalmente.

### Paso 1 — Verificar la URL del gestor de placas

1. Abrir Arduino IDE.
2. Ir a **Archivo → Preferencias**.
3. En el campo **"URLs adicionales de gestor de placas"**, verificar que esté presente la siguiente URL:
```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```
4. Si no está, agregarla y hacer clic en **OK**.

### Paso 2 — Instalar el core ESP32 versión 1.0.6

1. Ir a **Herramientas → Placa → Gestor de placas**.
2. Buscar `esp32`.
3. En el desplegable de versión del paquete **"esp32 by Espressif Systems"**, seleccionar la versión **1.0.6**.
4. Hacer clic en **Instalar**.
5. Esperar a que finalice la instalación.

> El core 1.0.6 y el core 3.x pueden coexistir en el mismo PC. No es necesario desinstalar el 3.x.

### Paso 3 — Seleccionar el core correcto antes de compilar

1. Ir a **Herramientas → Placa → esp32** (del core 1.0.6, no del 3.x).
2. Seleccionar **ESP32 Dev Module** (o la placa específica del hardware).
3. Verificar en **Herramientas → Placa** que la versión seleccionada corresponde al core 1.0.6 y no al 3.x.

### Paso 4 — Verificar las librerías instaladas

Con el core 1.0.6 las librerías requeridas son:

| Librería | Versión recomendada | Fuente |
|---|---|---|
| `RTClib` by Adafruit | 2.x | Gestor de Bibliotecas |
| `LiquidCrystal` by Arduino | cualquiera | Gestor de Bibliotecas |
| `Arduino_JSON` by Arduino | cualquiera | Gestor de Bibliotecas |
| `AnalogWrite_ESP32` | compatible con core 1.x | Gestor de Bibliotecas |

> Con el core 1.0.6, la librería `AnalogWrite_ESP32` **sí es compatible** y debe estar instalada.  
> Con el core 3.x, esa misma librería **no es compatible** y debe desinstalarse.

Verificar también que **no esté instalada** la librería `ArduinoBLE` (es para Arduino Nano 33, no para ESP32 — genera conflictos con la librería BLE nativa del core).

### Paso 5 — Abrir y verificar el código

1. Abrir el archivo `sonometro_QM.ino` (código original sin modificar) o `eva_4_2.ino` según corresponda.
2. Ir a **Sketch → Verificar/Compilar** (Ctrl+R).
3. Confirmar que no haya errores. Solo deben aparecer avisos (`warning`) sobre librerías duplicadas, que no impiden la compilación.

### Paso 6 — Cargar el código al dispositivo

1. Conectar la ESP32 al PC por USB.
2. Ir a **Herramientas → Puerto** y seleccionar el puerto COM asignado al dispositivo.
3. Ir a **Sketch → Subir** (Ctrl+U).
4. Esperar a que finalice la carga. El IDE mostrará **"Done uploading"** al terminar.

### Paso 7 — Verificar funcionamiento en el dispositivo

Antes del primer encendido con firmware nuevo, confirmar que la tarjeta SD contiene los 4 archivos de configuración en la raíz:

| Archivo | Formato | Ejemplo |
|---|---|---|
| `/archivo_wifi.txt` | `SSID>PASSWORD` | `MiRed>mipass123` |
| `/archivo_alertas.txt` | `green-yellow-red` (dBA) | `60.0-75.0-90.0` |
| `/archivo_estadistica.txt` | (por definir internamente) | — |
| `/archivo_servidor.txt` | URL completa del servidor | `http://192.168.1.100:4000/graphql` |

Para ver los logs de arranque, activar el modo debug descomentando la línea al inicio del archivo:
```cpp
#define DEBUG 1
```
y abrir el **Monitor Serie** a **115200 baudios**.

---

## Resumen de decisiones

| Situación | Solución aplicada |
|---|---|
| PC nuevo con core ESP32 3.x | Cambiar 4 líneas `std::string` → `String` en los callbacks BLE |
| Librería `AnalogWrite_ESP32` incompatible con core 3.x | Desinstalarla (core 3.x tiene `analogWrite` nativo) |
| Librería `ArduinoBLE` en conflicto | Desinstalarla (no es para ESP32) |
| Política de no modificar el código original | Instalar core ESP32 1.0.6 y usar `AnalogWrite_ESP32` compatible |