# Documentación Técnica — Sonómetro IoT ESP32
**ID del dispositivo:** `SONO-0039`  
**Archivo fuente:** `sonometro_QM.ino`  
**Plataforma:** ESP32 / Arduino IDE  
**Fecha de análisis:** Mayo 2026

---

## Tabla de contenidos
1. [Objetivo del proyecto](#1-objetivo-del-proyecto)
2. [Arquitectura general](#2-arquitectura-general)
3. [Librerías y dependencias](#3-librerías-y-dependencias)
4. [Flujo del programa](#4-flujo-del-programa)
5. [setup()](#5-setup)
6. [loop()](#6-loop)
7. [Funciones principales](#7-funciones-principales)
8. [Manejo de WiFi](#8-manejo-de-wifi)
9. [Bluetooth BLE](#9-bluetooth-ble)
10. [Hardware detectado](#10-hardware-detectado)
11. [Archivos de configuración en SD](#11-archivos-de-configuración-en-sd)
12. [Riesgos y problemas encontrados](#12-riesgos-y-problemas-encontrados)
13. [Recomendaciones técnicas](#13-recomendaciones-técnicas)
14. [Resumen ejecutivo](#14-resumen-ejecutivo)
15. [Guía de compilación](#15-guía-de-compilación)

---

## 1. Objetivo del proyecto

Sistema de monitoreo acústico ambiental (sonómetro) basado en ESP32. Mide niveles de presión sonora en dBA mediante un sensor analógico externo, los registra en una tarjeta SD con timestamp, los muestra en una pantalla LCD, activa LEDs de alerta según umbrales configurables y envía los datos a un servidor remoto vía HTTP/GraphQL. La configuración del dispositivo (WiFi, umbrales, servidor) se gestiona de forma inalámbrica mediante BLE desde una aplicación móvil.

---

## 2. Arquitectura general

```
[Sensor analógico dBA] ──ADC──> [ESP32]
                                    │
                  ┌─────────────────┼──────────────────────┐
                  │                 │                        │
            [LCD 16x2]       [LEDs R/Y/G]           [Tarjeta SD]
                                    │                  (log CSV)
                            [WiFi STA mode]
                                    │
                         [Servidor HTTP/GraphQL]
                                    │
                              [BLE Server]
                                    │
                         [App móvil de configuración]
```

**Modo de red:** WiFi Station (STA) — el ESP32 se conecta a una red existente.  
**Protocolo de envío:** HTTP POST con payload GraphQL (mutation).  
**Configuración remota:** BLE GATT Server con 4 características escribibles.

---

## 3. Librerías y dependencias

| Librería | Función en el proyecto | ¿Incluida en core ESP32? | ¿Instalar manualmente? | Fuente/Gestor | Observaciones |
|---|---|---|---|---|---|
| `Arduino.h` | Base del framework Arduino | ✅ Sí | No | Core ESP32 | — |
| `analogWrite.h` | Habilita `analogWrite()` en ESP32 (no existe nativamente) | ❌ No | **Sí** | Gestor: `analogWrite` by ERROWsquared o `ESP32AnalogWrite` | **CRÍTICO**: En ESP32 `analogWrite` no existe en el core estándar. Verificar cuál versión usa el proyecto. Posible conflicto con `ledc`. |
| `SPI.h` | Bus SPI (para la tarjeta SD) | ✅ Sí | No | Core ESP32 | — |
| `SD.h` | Lectura/escritura en tarjeta SD vía SPI | ✅ Sí | No | Core ESP32 | — |
| `RTClib.h` | Comunicación con el RTC DS3231 vía I2C | ❌ No | **Sí** | Gestor: `RTClib` by Adafruit | Versión recomendada: 2.x |
| `FS.h` | Sistema de archivos base (abstracción para SD) | ✅ Sí | No | Core ESP32 | — |
| `Wire.h` | Comunicación I2C (RTC DS3231) | ✅ Sí | No | Core ESP32 | — |
| `WiFiClient.h` | Cliente TCP/WiFi | ✅ Sí | No | Core ESP32 | Incluido en `WiFi.h` |
| `WiFi.h` | Gestión de conectividad WiFi | ✅ Sí | No | Core ESP32 | — |
| `ESPmDNS.h` | Resolución de nombres mDNS | ✅ Sí | No | Core ESP32 | **Incluido pero no usado** en el código — posible residuo. |
| `HTTPClient.h` | Envío de peticiones HTTP POST | ✅ Sí | No | Core ESP32 | — |
| `Arduino_JSON.h` | Parseo/generación JSON | ❌ No | **Sí** | Gestor: `Arduino_JSON` by Arduino | Diferente de `ArduinoJson` de Benoit Blanchon. No confundir. |
| `json.h` | Librería JSON adicional | ⚠️ Incierto | **Posiblemente sí** | Desconocido | **ALERTA**: No es una librería estándar conocida con ese nombre exacto. Podría ser un header local, parte de otra lib, o un archivo personalizado no adjunto. Verificar físicamente en el proyecto. |
| `WiFiMulti.h` | Gestión de múltiples APs WiFi con failover | ✅ Sí | No | Core ESP32 | — |
| `LiquidCrystal.h` | Control de LCD HD44780 en modo paralelo 4 bits | ⚠️ Parcial | **Sí (recomendado)** | Gestor: `LiquidCrystal` by Arduino | El core ESP32 no la incluye por defecto. Instalar desde el gestor. |
| `BLEDevice.h` | Inicialización del stack BLE | ✅ Sí | No | Core ESP32 | — |
| `BLEUtils.h` | Utilidades BLE | ✅ Sí | No | Core ESP32 | — |
| `BLEServer.h` | Servidor GATT BLE | ✅ Sí | No | Core ESP32 | — |
| `BLEScan.h` | Escaneo BLE | ✅ Sí | No | Core ESP32 | **Incluido pero no usado** en el código. |
| `BLEAdvertisedDevice.h` | Manejo de dispositivos anunciados BLE | ✅ Sí | No | Core ESP32 | **Incluido pero no usado** activamente. |
| `stdio.h` | Funciones estándar C (sprintf, printf) | ✅ Sí | No | C stdlib | — |
| `string` | Strings estándar C++ | ✅ Sí | No | C++ stdlib | Usado en BLE callbacks (`std::string`) |
| `iostream` | Streams C++ | ✅ Sí | No | C++ stdlib | **Incluido pero no usado** — residuo de desarrollo. |

### Resumen de instalación manual requerida:
1. `analogWrite` (ESP32AnalogWrite)
2. `RTClib` by Adafruit
3. `Arduino_JSON` by Arduino
4. `LiquidCrystal` by Arduino
5. `json.h` — **verificar si existe como archivo local en el proyecto original**

---

## 4. Flujo del programa

```
INICIO
  │
  ├─ setup()
  │     ├─ Configurar pines GPIO
  │     ├─ Iniciar WiFi (con ssid/pass vacíos inicialmente)
  │     ├─ Iniciar LCD 16x2
  │     ├─ Montar SD card
  │     ├─ Iniciar RTC DS3231
  │     ├─ Ajustar hora RTC si es necesario
  │     ├─ bleConnection() → levantar servidor BLE
  │     └─ readSdFiles() → cargar config desde SD (WiFi, umbrales, servidor)
  │
  └─ loop() [ciclo continuo]
        ├─ wifiConnection() → verificar/mantener WiFi
        ├─ [si controlFlag1] → writeSdFile() → crear nuevo CSV en SD
        ├─ Leer hora RTC
        ├─ [si 23:34:59] → cerrar archivo del día y preparar nuevo
        └─ [resto del tiempo]
              ├─ getSensorValue() → leer ADC → calcular dBA → LCD → calcAverage()
              ├─ backlightButton() → control retroiluminación
              ├─ [si valor > umbral amarillo] → isTheValueSendable() → HTTP POST
              ├─ checkData() → filtro anti-spike → actualizar LEDs
              ├─ logSdCard() → construir string CSV
              └─ appendFile() → escribir en SD
```

---

## 5. setup()

Ejecutado una sola vez al encender el dispositivo:

1. **GPIO**: Configura LED_RED (13), LED_YELLOW (12), LED_GREEN (4), LED_WIFI (32), LED_BLINK (14) como salidas. BUTTON (34) como entrada. Apaga todos los LEDs inicialmente.
2. **ADC**: Resolución a 10 bits (`analogReadResolution(10)`) → rango 0–1023.
3. **WiFi**: Llama `WiFi.begin()` y `wifiMulti.addAP()` con ssid/pass **todavía vacíos** (los carga después desde la SD). Este es un error de orden de inicialización (ver sección de riesgos).
4. **LCD**: Inicializa pantalla 16x2.
5. **SD**: Monta la tarjeta. Si falla, imprime error y hace `return` — el setup continúa parcialmente sin SD.
6. **RTC**: Verifica el módulo DS3231. Si no responde, entra en bucle infinito (`while(1)`).
7. **Hora**: Si el RTC perdió energía o la hora difiere de la hora de compilación, ajusta el RTC.
8. **BLE**: Levanta el servidor GATT con 4 características.
9. **SD Config**: Lee los 4 archivos de configuración y carga ssid, pass, umbrales y URL del servidor.

**Problema crítico:** WiFi se inicializa antes de leer la SD, por lo que la primera conexión siempre falla con credenciales vacías.

---

## 6. loop()

Ciclo principal sin FreeRTOS explícito (single-thread en core 1 del ESP32):

- **`wifiConnection()`**: Se llama en cada iteración. Maneja estado WiFi, blinking del LED_WIFI y reintento periódico de conexión. Cuando son las 59:59 de cualquier hora, envía datos de conexión al servidor.
- **`controlFlag1`**: Bandera para crear el archivo CSV del día. Se activa al inicio y a las 23:34:59.
- **Rotación de archivo**: A las 23:34:59 se escribe un marcador de fin de archivo JSON y se prepara el nuevo CSV del día siguiente.
- **Ciclo de medición**: `getSensorValue()` tiene un `delay(200)` interno → frecuencia de muestreo aprox. 5 Hz.

**No usa FreeRTOS**, no hay tareas paralelas, no hay interrupciones de hardware declaradas.

---

## 7. Funciones principales

### `getSensorValue()`
Lee el pin ADC 36, convierte a voltaje y luego a dBA:
```
analogValue  = analogRead(36)          // 0–1023
voltageValue = analogValue / 1023 * 3.8  // 0–3.8V
currentSensorValue = voltageValue * 50   // 0–190 dBA (escala lineal)
```
Luego llama `lcdScreen()` y `calcAverage()`. Incluye `delay(200)`.

**Observación**: La conversión ADC → dBA es completamente lineal. Esto implica que el sensor analógico externo ya entrega una señal proporcional a dBA. No hay corrección logarítmica en el firmware — toda la conversión es responsabilidad del hardware del sensor.

### `checkData()`
Implementa un filtro de dos muestras para reducir spikes:
- Almacena dos valores consecutivos (`controlValue1`, `controlValue2`).
- Si el valor anterior es mayor y supera el umbral amarillo (máx 130 dBA), usa el mayor y mantiene el estado.
- Si no supera el umbral, descarta y reinicia.
- Actualiza los LEDs con `turnOn()`.

**Alerta**: La lógica de `controlCount` tiene un caso `else` que asigna 1 sin hacer nada, creando una rama muerta. Ver sección de riesgos.

### `turnOn(float currentValue)`
Controla los 3 LEDs de alerta según umbrales leídos de la SD:

| Condición | LED Verde | LED Amarillo | LED Rojo |
|---|---|---|---|
| valor ≤ greenValue | PWM 200 (dim) | HIGH (apagado) | HIGH (apagado) |
| greenValue < valor ≤ yellowValue | LOW (apagado) | HIGH (apagado) | HIGH (apagado) |
| yellowValue < valor ≤ redValue | PWM 255 (encendido) | LOW (encendido) | HIGH (apagado) |
| valor > redValue | PWM 255 (encendido) | HIGH (apagado) | LOW (encendido) |

**Problema lógico**: En el rango verde (`valor ≤ greenValue`), el LED verde usa `analogWrite(200)` (encendido tenue), pero en el rango amarillo el LED verde se apaga (`LOW`). Esto significa que el verde NO está encendido cuando se supera el umbral verde. La lógica de LEDs es anti-intuitiva: los LEDs apagados tienen `HIGH` y los encendidos tienen `LOW` o `analogWrite`, lo que sugiere **LEDs en configuración de cátodo común o lógica invertida**.

### `isTheValueSendable()`
Decide si enviar un valor al servidor usando una ventana de tiempo de 10 segundos:
- Dentro de la ventana: solo envía si el cambio supera ±5% (ratio 21/20).
- Fuera de la ventana: envía si no se había enviado nada en esa ventana; si sí se envió, resetea el flag.

**Observación**: El umbral del ±5% se calcula como `valor * 21/20` (aumento) y `valor * 20/21` (disminución). Esto es correcto matemáticamente.

### `sendData(int dataType, float dataToSend)`
Construye una query GraphQL mutation y la envía por HTTP POST:

```
dataType == 0: newMeasurement(input:{data:{deviceID:"SONO-0039"}})  → ping de conexión
dataType == 1: newMeasurement(input:{data:{value:X, deviceID:"SONO-0039"}})  → medición
dataType == 2: newMeasurementProm(input:{data:{value:X, deviceID:"SONO-0039"}})  → promedio
```

- Puerto: no especificado → el servidor define el puerto en la URL almacenada en SD.
- Sin autenticación HTTP.
- Sin TLS/HTTPS verificado (depende de la URL en `archivo_servidor.txt`).

### `calcAverage()`
Acumula suma y conteo de muestras no nulas. Envía promedio vía GraphQL a las **06:59:58–59** y **18:59:58–59**. Requiere mínimo 11 muestras. Esto divide el día en dos períodos: nocturno (7pm–7am) y diurno (7am–7pm).

### `logSdCard()`
Construye la cadena CSV a escribir:
```
valor,HH:MM:SS,DD/MM/YYYY
```
**No escribe directamente** — solo construye `dataToWrite`. La escritura la hace `appendFile()` en el `loop()`.

### `writeSdFile()`
Crea un nuevo archivo CSV con nombre basado en fecha/hora:
```
/YYYYMMDD_HHMMSS.csv
```
Escribe el header: `value,time`.

### `readSdFiles()`
Lee 4 archivos de la SD y parsea su contenido con la función `split()`:

- `archivo_wifi.txt` → formato: `SSID>PASSWORD`
- `archivo_alertas.txt` → formato: `greenValue-yellowValue-redValue`
- `archivo_estadistica.txt` → formato desconocido (cargado pero no usado en el código analizado)
- `archivo_servidor.txt` → URL completa del servidor GraphQL

### `split(String data, char mark, int index)`
Implementación manual de split por delimitador. Extrae el token en la posición `index` del string `data` separado por `mark`.

### `bleConnection()`
Levanta un servidor GATT BLE con:
- **Nombre del dispositivo**: `SONO-0039`
- **1 servicio** con UUID `4fafc201-...`
- **4 características** READ/WRITE: WiFi, Alarmas, Estadísticas, Servidor

Cada característica tiene un callback que, al recibir escritura, sobreescribe el archivo correspondiente en la SD.

### `compareDateTime()`
Compara hora y minuto de la hora de compilación (`__DATE__`, `__TIME__`) con el RTC. Si difieren, retorna `true` para forzar la actualización del RTC.

---

## 8. Manejo de WiFi

| Parámetro | Valor |
|---|---|
| Modo | STA (Station) — se conecta a red existente |
| SSID/Password | Leídos desde `/archivo_wifi.txt` en la SD |
| IP | DHCP (no hay configuración de IP estática) |
| Hostname | `SONO-0039` |
| Reconexión | Manual mediante `wifiMulti.run()` cada ~100 iteraciones del loop |
| Protocolo de datos | HTTP POST con payload JSON/GraphQL |
| Puerto | Definido en la URL del servidor (no especificado en el código) |
| Seguridad | Ninguna autenticación adicional. Sin TLS forzado. |
| mDNS | Librería incluida pero **no utilizada** |

### Lógica de reconexión detallada:
```
Si WiFi desconectado:
  Cada ciclo con millis > lastWifiConnectionTime + 1250ms:
    → reinicia temporizador
    → controlFlag2 = false (inicia nueva ventana de blinking)
  
  Si controlFlag2 = true:
    → Parpadea LED_WIFI (ON_TIME=750ms, OFF_TIME=500ms)
    → Cada 100 iteraciones: llama wifiMulti.run() para reconectar
  
Si WiFi conectado:
  LED_WIFI = LOW (apagado)
  A las XX:59:59 → sendData(0,0) → ping al servidor
```

**Problema**: `controlFlag2` se setea en la misma iteración donde se evalúa, creando inconsistencia. El parpadeo puede no funcionar de forma confiable.

**Problema grave**: En `setup()`, `WiFi.begin()` se llama con `ssid` y `pass` vacíos porque `readSdFiles()` aún no se ha ejecutado. La conexión inicial siempre falla. La reconexión en el `loop()` eventualmente conecta cuando se llama `wifiMulti.run()` con las credenciales ya cargadas — pero solo después de ~100 iteraciones.

---

## 9. Bluetooth BLE

| Parámetro | Valor |
|---|---|
| Modo | GATT Server (Peripheral/Slave) |
| Nombre BLE | `SONO-0039` |
| Service UUID | `4fafc201-1fb5-459e-8fcc-c5c9c331914b` |
| Advertising | Permanente desde el `setup()` |
| Seguridad | Ninguna (sin pairing, sin bonding, sin autenticación) |

### Características BLE:

| Característica | UUID | Acción al escribir |
|---|---|---|
| WiFi | `beb5483e-...-26a8` | Sobreescribe `/archivo_Wifi.txt` en SD |
| Alarmas | `beb5483e-...-26a9` | Sobreescribe `/archivo_alertas.txt` en SD |
| Estadísticas | `4fafc201-...-9141` | Sobreescribe `/archivo_estadistica.txt` en SD |
| Servidor | `4fafc201-...-9142` | Sobreescribe `/archivo_servidor.txt` en SD |

**Importante**: Modificar estas características desde BLE sobreescribe la configuración en la SD, pero el dispositivo **no recarga** los valores en RAM automáticamente. Requiere reinicio para que surtan efecto.

**Bug de inconsistencia de nombre de archivo**: En la lectura (`readSdFiles`) el archivo WiFi se lee como `/archivo_wifi.txt` (minúsculas), pero en el callback BLE (`WifiCallback::onWrite`) se escribe como `/archivo_Wifi.txt` (con mayúscula). En sistemas de archivos FAT (tarjeta SD) esto puede o no ser un problema dependiendo de la implementación — en FAT32 es case-insensitive, pero puede generar dos archivos distintos en algunos entornos.

---

## 10. Hardware detectado

### Pines GPIO utilizados:

| Pin | Tipo | Función |
|---|---|---|
| 36 | ADC (entrada analógica, solo input) | Señal analógica del sensor de sonido |
| 4 | Salida digital/PWM | LED Verde |
| 12 | Salida digital | LED Amarillo |
| 13 | Salida digital | LED Rojo |
| 32 | Salida digital | LED WiFi |
| 14 | Salida digital | LED Blink (retroiluminación LCD) |
| 34 | Entrada digital (solo input) | Botón retroiluminación |
| 0 | Salida digital | LCD RS |
| 15 | Salida digital | LCD RW |
| 2 | Salida digital | LCD EN |
| 25 | Salida digital | LCD D4 |
| 26 | Salida digital | LCD D5 |
| 27 | Salida digital | LCD D6 |
| 33 | Salida digital | LCD D7 |
| SDA (21) | I2C | RTC DS3231 *(pin implícito)* |
| SCL (22) | I2C | RTC DS3231 *(pin implícito)* |
| MOSI/MISO/SCK/CS | SPI | Tarjeta SD *(pines SPI estándar del ESP32)* |

### Componentes externos confirmados:

| Componente | Confirmado/Probable | Evidencia |
|---|---|---|
| Sensor de sonido analógico (salida 0–VREF en dBA) | ✅ Confirmado | `SENSOR_PIN 36`, `VREF 3.8`, conversión lineal |
| LCD HD44780 16x2 (interfaz paralela 4 bits) | ✅ Confirmado | `LiquidCrystal(RS,RW,EN,D4..D7)`, `lcd.begin(16,2)` |
| RTC DS3231 (I2C) | ✅ Confirmado | `RTC_DS3231 rtc`, `rtc.begin()` |
| Tarjeta microSD (SPI) | ✅ Confirmado | `SD.begin()` |
| 3x LEDs de alerta (Rojo, Amarillo, Verde) | ✅ Confirmado | Pines 13, 12, 4 |
| LED WiFi | ✅ Confirmado | Pin 32 |
| LED Blink / Retroiluminación | ✅ Confirmado | Pin 14 |
| Botón pulsador | ✅ Confirmado | Pin 34 |
| App móvil BLE (Android/iOS) | ⚠️ Probable | BLE GATT server con 4 características — implica app cliente externa no incluida |

---

## 11. Archivos de configuración en SD

El dispositivo depende de 4 archivos presentes en la raíz de la SD antes del primer arranque:

| Archivo | Formato | Ejemplo |
|---|---|---|
| `/archivo_wifi.txt` | `SSID>PASSWORD` | `MiRedWiFi>mipassword123` |
| `/archivo_alertas.txt` | `green-yellow-red` (en dBA) | `60.0-75.0-90.0` |
| `/archivo_estadistica.txt` | Desconocido (no usado en el código) | — |
| `/archivo_servidor.txt` | URL completa del endpoint GraphQL | `http://192.168.1.100:4000/graphql` |

**Si alguno de estos archivos no existe, el dispositivo opera con valores vacíos/cero para esos parámetros**, sin ningún error explícito en producción (el modo DEBUG está comentado por defecto).

---

## 12. Riesgos y problemas encontrados

### 🔴 Críticos

**1. WiFi se inicializa antes de leer la SD**
`WiFi.begin(ssid, pass)` se llama en `setup()` con arrays vacíos. Las credenciales reales se cargan en `readSdFiles()` que se llama después. La conexión WiFi inicial siempre falla.

**2. Sin manejo de error en `SD.begin()`**
Si la SD falla, `setup()` hace `return` y continúa sin SD. El `loop()` seguirá llamando `appendFile()` sobre archivos inexistentes indefinidamente, sin reportar errores (DEBUG desactivado).

**3. `json.h` no identificada**
Esta librería no corresponde a ninguna librería pública estándar conocida. Si no existe en el entorno de compilación, el proyecto no compilará.

**4. Sin seguridad BLE**
Cualquier dispositivo en rango puede conectarse y sobrescribir la configuración WiFi, los umbrales de alerta y la URL del servidor. Riesgo de sabotaje o desconfiguración no intencional.

### 🟡 Importantes

**5. `turnOn()` tiene lógica invertida**
En los rangos verde y amarillo, el LED verde se comporta de forma contraintuitiva (se apaga cuando el nivel supera el umbral verde). Puede ser intencional si los LEDs están en configuración pull-up, pero debería documentarse.

**6. `checkData()` tiene una rama muerta**
El bloque `else` del primer `if(controlCount==1)` asigna `controlCount=1` sin ejecutar ninguna acción útil. El flujo lógico de este filtro es difícil de seguir y probablemente contenga un error de diseño.

**7. Tiempo de escritura SD en el loop crítico**
`appendFile()` se llama en cada iteración del `loop()`. Las operaciones de SD son bloqueantes. Combinado con `delay(200)` en `getSensorValue()`, el ciclo real puede tomar entre 200–500ms o más por iteración.

**8. `compareDateTime()` usa hora de compilación**
Cada vez que el firmware se recompila y reflashea (aunque sea por otro motivo), el RTC se actualiza a la hora de compilación. Si se flashea fuera de horario, el RTC quedará con hora incorrecta hasta el próximo ciclo normal.

**9. `averageSum` no se resetea si `averageCounter <= 10`**
Si en las ventanas de las 6:59 o 18:59 hay menos de 11 muestras, `averageSum` y `averageCounter` NO se resetean. Se acumulan en el siguiente período, contaminando el promedio.

**10. `millis()` almacenado en `long` con signo**
`lastWifiConnectionTime`, `currentTimeWindow` y `lastTimeWindow` son `long`. `millis()` retorna `unsigned long`. Después de ~24.8 días, `millis()` desborda y los valores signed pueden volverse negativos, rompiendo las comparaciones de tiempo.

**11. `fileName` tiene 22 bytes pero el formato genera 22 caracteres + null**
`sprintf(fileName, "/%02d%02d%02d_%02d%02d%02d.csv", ...)` genera `/YYYYMMDD_HHMMSS.csv` = 20 chars + null = 21 bytes. El buffer de 22 es justo. Cualquier cambio de formato podría causar buffer overflow.

**12. Librerías `ESPmDNS`, `BLEScan`, `BLEAdvertisedDevice`, `iostream` incluidas pero no usadas**
Incrementan innecesariamente el tamaño del firmware y el tiempo de compilación.

### 🟢 Menores

**13. Nombre del display en LCD**
El LCD siempre muestra `FCV EVA` en la primera línea. Parece un nombre de proyecto/institución hardcodeado. No configurable.

**14. `statisticsFile` cargado pero nunca utilizado**
El contenido de `/archivo_estadistica.txt` se lee en memoria pero no se usa en ninguna función del código analizado. Posible funcionalidad incompleta o eliminada.

**15. Hora de rotación de archivo fijada a 23:34:59**
La condición `(hourNow == 23) && (minuteNow == 34) && (secondNow == 59)` es muy específica. Si el ESP32 está procesando en ese segundo exacto y lo "salta" por latencia, no se creará el nuevo archivo hasta el día siguiente.

---

## 13. Recomendaciones técnicas

1. **Corregir el orden en `setup()`**: Llamar `readSdFiles()` antes de `WiFi.begin()` y `wifiMulti.addAP()`.

2. **Añadir `analogReadResolution(10)` antes de `analogRead()`** (ya está, pero verificar que ocurra antes del primer `getSensorValue()`).

3. **Cambiar `long` a `unsigned long`** en `lastWifiConnectionTime`, `currentTimeWindow`, `lastTimeWindow`.

4. **Resetear `averageSum` y `averageCounter`** incluso cuando `averageCounter <= 10`.

5. **Implementar autenticación BLE básica**: Al menos un PIN de emparejamiento para proteger las características de escritura.

6. **Agregar validación de archivos SD**: Verificar existencia de los 4 archivos de configuración al inicio y entrar en modo de error si faltan.

7. **Mover `delay(200)` fuera de `getSensorValue()`**: Centralizar los delays en `loop()` mejora la legibilidad y el control del timing.

8. **Eliminar librerías no usadas**: `ESPmDNS.h`, `BLEScan.h`, `BLEAdvertisedDevice.h`, `iostream`.

9. **Unificar el nombre del archivo WiFi**: `archivo_wifi.txt` vs `archivo_Wifi.txt` debe ser consistente.

10. **Documentar la app BLE**: Existe una aplicación móvil externa que interactúa con este dispositivo. Debe documentarse su nombre, plataforma y protocolo de datos esperado en cada característica.

---

## 14. Resumen ejecutivo

El proyecto es un **sonómetro IoT industrial** con capacidad de registro local (SD), visualización (LCD + LEDs), reporte remoto (HTTP/GraphQL) y configuración inalámbrica (BLE). El código está **funcional pero con deuda técnica significativa**: el problema más grave es la inicialización de WiFi antes de leer las credenciales de la SD, lo que hace que la primera conexión siempre falle. La ausencia de seguridad en BLE es un riesgo operacional. El proyecto parece estar en estado **parcialmente terminado** (hay código y archivos referenciados pero no implementados, como `statisticsFile`). Para mantenerlo en producción, los bugs de `long`/`unsigned long` y el reset del promedio deben corregirse prioritariamente.

---

## 15. Guía de compilación

### Entorno necesario:
- **Arduino IDE** 1.8.x o 2.x
- **Board package**: `esp32 by Espressif Systems` → versión 2.x (recomendado 2.0.14)
  - URL: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
- **Board seleccionada**: `ESP32 Dev Module` (o la específica del hardware)

### Librerías a instalar desde el Gestor de Bibliotecas:

| Librería | Nombre exacto en gestor |
|---|---|
| RTClib | `RTClib` by Adafruit |
| LiquidCrystal | `LiquidCrystal` by Arduino |
| Arduino_JSON | `Arduino_JSON` by Arduino |
| analogWrite | `ESP32AnalogWrite` by Kevin Harrington (verificar compatibilidad) |

### ⚠️ Antes de compilar, verificar:
- [ ] Qué librería es `json.h` — buscar en el proyecto original un archivo `json.h` o `json.cpp` local.
- [ ] Que `ESP32AnalogWrite` no entre en conflicto con el core ESP32 2.x (en versiones recientes `analogWrite` ya está incluido de forma nativa, lo que causaría redefinición).
- [ ] Que el board package ESP32 sea compatible con la versión de todas las librerías.

### Verificación física en la placa:
- [ ] Confirmar pines SPI de la tarjeta SD (MOSI, MISO, SCK, CS) según el esquemático del hardware.
- [ ] Confirmar pines I2C del RTC DS3231 (SDA=21, SCL=22 por defecto en ESP32).
- [ ] Confirmar que el sensor analógico en pin 36 entrega voltaje en rango 0–3.8V.
- [ ] Confirmar que los 4 archivos de configuración están presentes en la SD con el formato correcto antes del primer encendido.
- [ ] Activar `#define DEBUG 1` para verificar el funcionamiento en el primer arranque.

### Archivos SD requeridos antes del primer encendido:
```
/archivo_wifi.txt       → SSID>PASSWORD
/archivo_alertas.txt    → greenDBA-yellowDBA-redDBA
/archivo_estadistica.txt → (contenido por definir)
/archivo_servidor.txt   → http://IP:PUERTO/ruta
```
