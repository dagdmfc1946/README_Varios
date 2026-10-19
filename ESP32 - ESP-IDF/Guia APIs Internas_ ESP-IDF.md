# APIs Internas de ESP-IDF – Guía Técnica

## 1. ¿Qué son las APIs internas de ESP-IDF?

Las APIs de ESP-IDF son funciones, estructuras y módulos que permiten interactuar directamente con:

- Hardware (GPIO, UART, SPI, etc.)
- Sistema (memoria, reset, información del chip)
- Sistema operativo (FreeRTOS)
- Periféricos (WiFi, BLE, ADC, etc.)

En Arduino (ESP32), estas APIs siguen existiendo, pero están:
- Envuelta (wrappers)
- Parcialmente ocultas

---

## 2. APIs clave que debes dominar

### 🔴 2.1 Sistema (Core del MCU)

**Librerías:**
```c
#include "esp_system.h"
#include "esp_chip_info.h"
#include "esp_flash.h"
```

**Funciones:**
```c
esp_restart();
esp_get_free_heap_size();
esp_get_minimum_free_heap_size();
esp_chip_info();
esp_flash_get_size();
```

**Uso:**
- Control del sistema
- Información del hardware
- Estado de memoria

---

### 🔴 2.2 FreeRTOS (Obligatorio)

**Librerías:**
```c
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
```

**Funciones:**
```c
xTaskCreate();
vTaskDelay();
vTaskDelete();
```

**Conceptos clave:**
- Multitarea
- Scheduler
- No bloqueo del CPU

---

### 🔴 2.3 GPIO

**Librería:**
```c
#include "driver/gpio.h"
```

**Funciones:**
```c
gpio_set_direction();
gpio_set_level();
gpio_get_level();
gpio_reset_pin();
```

---

### 🟠 2.4 Logging

**Librería:**
```c
#include "esp_log.h"
```

**Funciones:**
```c
ESP_LOGI();
ESP_LOGW();
ESP_LOGE();
```

**Ventajas:**
- Mejores que `printf`
- Niveles de log
- Filtrado

---

### 🟠 2.5 NVS (Memoria no volátil)

**Librerías:**
```c
#include "nvs_flash.h"
#include "nvs.h"
```

**Funciones:**
```c
nvs_flash_init();
nvs_open();
nvs_set_i32();
nvs_get_i32();
```

**Uso:**
- Guardar configuraciones persistentes

---

### 🟠 2.6 WiFi / Networking

**Librerías:**
```c
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_netif.h"
```

**Funciones:**
```c
esp_wifi_init();
esp_wifi_start();
esp_event_handler_register();
```

---

### 🟡 2.7 Interrupciones

**Funciones:**
```c
gpio_install_isr_service();
gpio_isr_handler_add();
```

**Consideraciones:**
- No usar `printf`
- No usar memoria dinámica
- Código mínimo

---

### 🟡 2.8 Drivers de periféricos

**UART**
```c
#include "driver/uart.h"
```

**I2C**
```c
#include "driver/i2c.h"
```

**SPI**
```c
#include "driver/spi_master.h"
```

---

### 🟡 2.9 Memoria avanzada

**Librería:**
```c
#include "esp_heap_caps.h"
```

**Funciones:**
```c
heap_caps_malloc();
heap_caps_get_free_size();
```

---

## 3. Arduino vs ESP-IDF

| Característica | Arduino | ESP-IDF |
|--------------|--------|--------|
| Nivel | Alto | Bajo |
| Control | Limitado | Total |
| API | Wrappers | Directa |
| RTOS | Oculto | Explícito |

### Ejemplo:

**Arduino:**
```cpp
ESP.restart();
ESP.getFreeHeap();
```

**ESP-IDF:**
```c
esp_restart();
esp_get_free_heap_size();
```

---

## 4. Librerías vs Componentes

| Arduino | ESP-IDF |
|--------|--------|
| Librerías | Componentes |
| .h + .cpp | Estructura modular |
| Manual | Integrado en build |

### Estructura típica:
```
components/
main/
```

### Ejemplo:
```
components/mi_driver/
 ├── mi_driver.c
 ├── mi_driver.h
 └── CMakeLists.txt
```

---

## 5. Documentación

### 📘 Oficial
https://docs.espressif.com/projects/esp-idf/en/latest/

---

### 📂 API Reference

Ejemplo GPIO:
https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/gpio.html

Incluye:
- Funciones
- Estructuras
- Ejemplos

---

### 📁 Ejemplos oficiales

Ruta local:
```
esp-idf/examples/
```

Ejemplos importantes:
- GPIO
- FreeRTOS
- WiFi
- NVS

---

### 🔬 Código fuente

```
esp-idf/components/
```

Permite:
- Entender implementación real
- Ver funcionamiento interno

---

## 6. Estrategia de aprendizaje

Para cada función:

1. Buscar en documentación
2. Revisar ejemplo oficial
3. Identificar:
   - Contexto (task / ISR)
   - Dependencias
4. Implementar prueba

---

## 7. Errores comunes

- Usar solo Arduino API
- Ignorar FreeRTOS
- No revisar ejemplos oficiales
- No validar retornos (`ESP_OK`)

---

## 8. Conclusión

- ESP-IDF = control total del hardware
- Arduino = capa de abstracción
- Dominio real = uso directo de APIs ESP-IDF

---

## Nota técnica
ESP-IDF está basado en:
- C
- FreeRTOS
- HAL modular

---

## Nota adicional
El conocimiento transferible está en:
- RTOS
- Drivers
- Manejo de memoria

---

## Nota de seguridad
Uso incorrecto de APIs puede causar:
- Fallos del sistema
- Corrupción de memoria
- Reinicios inesperados

Siempre:
- Validar retornos
- Minimizar código en ISR
- Controlar uso de memoria
