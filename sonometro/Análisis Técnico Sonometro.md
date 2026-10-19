# Análisis Técnico — Proyecto Sonómetro IoT ESP32

## Lista de verificación del análisis

- Identificación completa de librerías y dependencias.
- Reconstrucción de la arquitectura funcional del firmware.
- Análisis detallado del flujo de ejecución (`setup()`, `loop()`, BLE, WiFi, SD, RTC y LCD).
- Detección de hardware y periféricos reales basados en evidencia del código.
- Evaluación de riesgos técnicos, malas prácticas y posibles fallas de estabilidad.
- Documentación para recompilar, mantener y depurar el proyecto heredado.
- Diferenciación entre elementos confirmados, probables y especulativos.

---

# Librerías y dependencias detectadas

| Librería | Función | ¿Incluida en ESP32? | ¿Instalar manualmente? | Fuente/gestor | Observaciones |
|---|---|---|---|---|---|
| Arduino.h | Base del framework Arduino | Sí | No | Core ESP32 | Fundamental |
| analogWrite.h | PWM estilo Arduino para ESP32 | No siempre | Probablemente sí | Library Manager / GitHub | Puede generar conflictos según versión del core ESP32 |
| SPI.h | Comunicación SPI | Sí | No | Core ESP32 | Utilizada por SD |
| SD.h | Manejo tarjeta SD | Sí | No | Core ESP32 | Requiere SPI |
| RTClib.h | Manejo RTC DS3231 | No | Sí | Adafruit RTClib | Dependencia externa |
| FS.h | Sistema de archivos | Sí | No | Core ESP32 | Utilizada por SD |
| Wire.h | I2C | Sí | No | Core ESP32 | Requerida por RTC |
| WiFiClient.h | Cliente TCP WiFi | Sí | No | Core ESP32 | HTTP |
| WiFi.h | Manejo WiFi ESP32 | Sí | No | Core ESP32 | STA mode |
| ESPmDNS.h | mDNS | Sí | No | Core ESP32 | Incluida pero NO usada |
| HTTPClient.h | Cliente HTTP | Sí | No | Core ESP32 | POST HTTP |
| Arduino_JSON.h | JSON Arduino | No | Sí | Arduino_JSON | Incluida pero NO usada |
| json.h | JSON adicional | Dudosa | Probablemente sí | Desconocido | Posible librería personalizada/obsoleta |
| WiFiMulti.h | Gestión múltiples AP WiFi | Sí | No | Core ESP32 | Reconexión |
| LiquidCrystal.h | LCD paralelo HD44780 | Sí | No | Arduino | LCD 16x2 |
| BLEDevice.h | BLE ESP32 | Sí | No | Core ESP32 | Configuración BLE |
| BLEUtils.h | Utilidades BLE | Sí | No | Core ESP32 | BLE |
| BLEServer.h | BLE Server | Sí | No | Core ESP32 | BLE GATT |
| BLEScan.h | BLE Scan | Sí | No | Core ESP32 | Incluida pero NO usada |
| BLEAdvertisedDevice.h | BLE Advertising | Sí | No | Core ESP32 | Incluida pero NO usada |
| stdio.h | C estándar | Sí | No | Toolchain | Bajo nivel |
| string | STL string | Sí | No | Toolchain C++ | Uso parcial |
| iostream | Streams C++ | Sí | No | Toolchain C++ | NO usada |

---

# Objetivo del proyecto

## Confirmado

El firmware implementa un sonómetro IoT basado en ESP32 con:

- Medición de nivel sonoro mediante entrada analógica
- Visualización local en LCD
- Indicadores visuales mediante LEDs
- Registro histórico en tarjeta SD
- Comunicación WiFi con servidor HTTP
- Configuración BLE
- RTC para timestamp
- Generación de estadísticas/promedios

---

# Arquitectura general

| Módulo | Función |
|---|---|
| Sensor analógico | Lectura nivel sonoro |
| RTC DS3231 | Fecha/hora |
| SD Card | Persistencia |
| LCD 16x2 | Visualización |
| LEDs | Indicadores de estado |
| BLE | Configuración remota |
| WiFi | Comunicación servidor |
| HTTP Client | Envío de datos |
| Sistema de alertas | Umbrales acústicos |

---

# Flujo del programa

## Secuencia de arranque

```text
setup()
 ├── Configuración GPIO
 ├── Inicialización LCD
 ├── Inicialización SD
 ├── Inicialización RTC
 ├── Ajuste automático fecha/hora
 ├── Inicialización BLE
 ├── Lectura archivos SD
 ├── Configuración WiFi
 └── Inicio operativo
```

## Flujo principal

```text
loop()
 ├── Verificar WiFi
 ├── Crear archivo si es nuevo día
 ├── Leer sensor
 ├── Actualizar LCD
 ├── Calcular promedio
 ├── Evaluar alertas
 ├── Registrar SD
 ├── Enviar datos HTTP
 └── Control LEDs
```

---

# setup()

## Inicialización GPIO

### LEDs

| Pin | Función |
|---|---|
| GPIO13 | LED rojo |
| GPIO12 | LED amarillo |
| GPIO4 | LED verde |
| GPIO32 | LED WiFi |
| GPIO14 | Backlight LCD |

### Entrada

| Pin | Función |
|---|---|
| GPIO34 | Botón backlight |

### Sensor

| Pin | Función |
|---|---|
| GPIO36 | Entrada analógica sensor sonoro |

---

# LCD paralelo

| LCD | GPIO |
|---|---|
| RS | 0 |
| RW | 15 |
| EN | 2 |
| D4 | 25 |
| D5 | 26 |
| D6 | 27 |
| D7 | 33 |

## Observación crítica

El uso de GPIO0, GPIO2 y GPIO15 es riesgoso en ESP32 porque son pines de strapping/boot.

Puede causar:
- fallas de arranque,
- boot mode incorrecto,
- bloqueos intermitentes.

---

# RTC detectado

## Confirmado

Hardware detectado:

- DS3231

Comunicación:
- I2C (`Wire.h`)

---

# Sistema SD

## Archivos detectados

| Archivo | Función |
|---|---|
| /archivo_wifi.txt | SSID y password |
| /archivo_alertas.txt | Umbrales |
| /archivo_estadistica.txt | Configuración estadísticas |
| /archivo_servidor.txt | URL servidor |

---

# Formato esperado archivos

## WiFi

```text
SSID>PASSWORD
```

## Alertas

```text
valorVerde-valorAmarillo-valorRojo
```

## Servidor

```text
http://servidor/graphql
```

---

# loop()

| Función | Propósito |
|---|---|
| wifiConnection() | Gestión WiFi |
| getSensorValue() | Lectura sensor |
| backlightButton() | Control backlight |
| checkData() | Filtrado/alertas |
| logSdCard() | Construcción log |
| appendFile() | Escritura SD |

---

# Funcionamiento del sensor

## Conversión

```cpp
voltageValue = analogValue / 1023 * VREF;
currentSensorValue = voltageValue * 50;
```

## Confirmado

El sensor entrega:
- salida analógica proporcional al nivel acústico.

## Probable

Existe:
- módulo sonómetro analógico externo,
- preamplificador,
- o salida calibrada de dBA.

---

# Manejo de WiFi

## Tipo de funcionamiento

Confirmado:
- STA

NO existe:
- AP
- AP+STA

---

# Reconexión

Se utiliza `WiFiMulti`, pero incorrectamente.

Problema:
`wifiMulti.addAP(...)` se ejecuta repetidamente dentro del `loop()`.

Consecuencias:
- consumo innecesario de memoria,
- duplicación de AP,
- degradación de estabilidad.

---

# IP

## Confirmado

- DHCP

NO se detecta IP fija.

---

# Comunicación servidor

## Confirmado

Se utiliza:
- HTTP POST
- payload GraphQL

---

# Riesgos de seguridad

## Detectados

### 1. Credenciales WiFi en texto plano

Almacenadas en SD sin cifrado.

### 2. BLE inseguro

Sin:
- pairing,
- autenticación,
- cifrado.

### 3. HTTP sin HTTPS

Riesgos:
- MITM,
- interceptación,
- manipulación tráfico.

---

# BLE detectado

## Función

Configuración remota del dispositivo.

## UUIDs detectados

| UUID | Función |
|---|---|
| WIFI_UUID | Configuración WiFi |
| ALARM_UUID | Umbrales |
| AVERAGE_UUID | Estadísticas |
| SERVER_UUID | Servidor |

---

# Hardware detectado

| Hardware | Evidencia |
|---|---|
| ESP32 | Librerías |
| RTC DS3231 | RTClib |
| LCD 16x2 paralelo | LiquidCrystal |
| Tarjeta SD | SD.h |
| Sensor analógico sonido | GPIO36 |
| LEDs | GPIO |
| Pulsador | GPIO34 |
| BLE | BLE stack |

---

# Problemas encontrados

## Críticos

### 1. Buffer overflow potencial

```cpp
char serverName[64]
```

Si la URL supera 63 caracteres:
- corrupción de memoria.

### 2. Uso intensivo de `String`

Riesgos:
- fragmentación heap,
- resets aleatorios,
- watchdog.

### 3. Uso excesivo de `delay()`

Bloquea:
- WiFi,
- BLE,
- tareas internas ESP32.

### 4. Escritura SD continua

Riesgos:
- desgaste SD,
- corrupción,
- latencia.

### 5. LCD.clear() continuo

Produce:
- flickering,
- consumo innecesario.

### 6. Error lógico en WiFiMulti

`addAP()` repetitivo.

### 7. Conversión ADC potencialmente incorrecta

No hay:
- calibración,
- compensación,
- filtrado digital.

### 8. Posible bug por nombres de archivo

Se detecta inconsistencia entre:
- `/archivo_Wifi.txt`
- `/archivo_wifi.txt`

---

# Código obsoleto detectado

Diseño Arduino clásico bloqueante.

NO utiliza:
- FreeRTOS tasks,
- queues,
- timers ESP32,
- eventos WiFi.

---

# Funcionalidades ausentes

| Función | Estado |
|---|---|
| Watchdog handling | Ausente |
| OTA | Ausente |
| Logs robustos | Ausente |
| NTP | Ausente |
| HTTPS | Ausente |
| Reconexión robusta | Débil |
| Mutex/thread safety | Ausente |
| CRC/config validation | Ausente |

---

# Componentes posiblemente faltantes

## Probable

- App BLE móvil
- Documentación hardware
- Esquema PCB
- Backend GraphQL
- Procedimiento de calibración

---

# Requisitos para compilar

## Arduino IDE

Versión recomendada:
- Arduino IDE 2.x

## Core ESP32 recomendado

- ESP32 Arduino Core 2.0.14

---

# Librerías a instalar manualmente

| Librería | Autor |
|---|---|
| RTClib | Adafruit |
| Arduino_JSON | Arduino |
| analogWrite | Compatible ESP32 |

---

# Verificaciones físicas recomendadas

## Críticas

1. Modelo exacto del sensor acústico
2. RTC DS3231 y batería
3. SD FAT32
4. LCD 16x2 HD44780
5. Alimentación estable 3.3V

---

# Recomendaciones técnicas

## Prioridad alta

1. Migrar configuración a NVS/Preferences
2. Reemplazar `String`
3. Implementar HTTPS
4. Reescribir arquitectura con FreeRTOS
5. Agregar watchdog
6. Corregir uso de GPIO boot pins

---

# Resumen ejecutivo técnico

El proyecto corresponde a un sonómetro IoT basado en ESP32 con:

- adquisición analógica,
- almacenamiento SD,
- RTC DS3231,
- LCD,
- configuración BLE,
- transmisión HTTP GraphQL.

La arquitectura es funcional pero claramente legacy y presenta múltiples riesgos típicos de migraciones Arduino → ESP32.

## Estado general

| Área | Estado |
|---|---|
| Funcionalidad básica | Buena |
| Escalabilidad | Baja |
| Robustez | Media-baja |
| Seguridad | Baja |
| Mantenibilidad | Media |
| Calidad arquitectónica | Legacy |
| Riesgo producción | Medio-Alto |

---

# Documentación recomendada

1. Diagrama eléctrico
2. Mapa GPIO
3. Formato archivos SD
4. Protocolo BLE
5. API backend GraphQL
6. Flujo operativo
7. Procedimiento calibración
8. Procedimiento actualización firmware
9. Matriz versiones librerías
10. Manual mantenimiento técnico
