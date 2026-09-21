# Guía de Rendimiento y Optimización para Raspberry Pi CM4

Este documento recopila la sesión de diagnóstico de rendimiento y proporciona una guía estructurada para optimizar y distribuir el consumo de recursos (**CPU, GPU y RAM**) en un módulo **Raspberry Pi Compute Module 4 (CM4)** con **2GB de RAM** y **8GB de eMMC**.

---

## 1. Resumen del Diagnóstico Inicial

Durante la ejecución de la infraestructura actual, se identificaron los siguientes comportamientos en el sistema mediante la herramienta `htop`:

*   **Saturación de CPU:** El núcleo 4 de la CPU se encontraba al **100% de capacidad**, mientras que los núcleos 1, 2 y 3 permanecían en niveles bajos (3% - 28%).
*   **Consumo de RAM:** Estable en **819 MB de 1.81 GB disponibles**, un rango seguro pero con margen de optimización limitado para una variante de 2GB.
*   **Procesos Críticos Identificados:**
    *   `vspm-pm6750 --se0_path=/dev/ttyAMA1`: Consumo combinado de ~50% de CPU total entre dos hilos/procesos leyendo el puerto serie.
    *   `python ./monitor/raspberry/gpio_keyboard.py`: Consumo de ~16.2% de CPU monitoreando pines GPIO.

---

## 2. Comandos Esenciales de Monitoreo

Para continuar supervisando la salud del sistema desde la terminal, utiliza los siguientes comandos:

```bash
# Monitoreo general interactivo de CPU, RAM y procesos
htop

# Verificar la temperatura del SoC (Evitar Thermal Throttling > 80°C)
vcgencmd measure_temp

# Verificar la frecuencia de reloj actual del procesador (Por defecto 1.5 GHz)
vcgencmd measure_clock arm

# Monitorear la lectura/escritura (I/O) en la memoria eMMC en tiempo real
sudo iotop
```

---

## 3. Estrategias de Optimización por Recurso

Dada la combinación de un entorno web pesado (**Django, HTML, CSS**), scripts de automatización (**Python GPIO**) y un binario nativo de bajo nivel (**C**), se recomiendan las siguientes acciones específicas:

### 🧠 Optimización de Memoria RAM (Límite: 2GB)
1.  **Sustituir el servidor de desarrollo de Django:** Nunca uses `python manage.py runserver` en producción. Consume mucha RAM y CPU al recargar archivos. Utiliza **Gunicorn** o **uWSGI** combinado con un servidor ligero como **Nginx** para despachar el contenido estático (HTML/CSS) de forma directa sin tocar Python.
2.  **Ajustar Workers:** Configura Gunicorn para usar máximo **2 workers** (`gunicorn --workers=2`). Cada worker de Django duplica el consumo de RAM.
3.  **Reducir la GPU Memory Split:** Si la Raspberry CM4 no está conectada a una pantalla 4K ni procesa interfaces gráficas locales (renderiza la web por red), reduce la memoria asignada a la GPU al mínimo absoluto (16MB o 32MB) editando el archivo `/boot/config.txt`:
    ```ini
    gpu_mem=16
    ```

### ⚡ Optimización de CPU y Distribución de Núcleos (Multiprocesamiento)
1.  **Mitigar el bucle infinito en Python (`gpio_keyboard.py`):** Si el script lee los pines GPIO dentro de un ciclo `while True:`, añade un retraso microscópico obligatorio. Pasar de ciclos infinitos a un muestreo de 10ms reduce radicalmente el consumo de CPU:
    ```python
    import time
    # Dentro del bucle principal
    time.sleep(0.01) # Baja el uso del núcleo del 100% a <5%
    ```
2.  **Librería Multiprocessing en Python:** Para forzar a Python a delegar tareas pesadas a otros núcleos en lugar de saturar uno solo debido al GIL (Global Interpreter Lock), fragmenta las funciones independientes utilizando procesos reales:
    ```python
    from multiprocessing import Process
    # Lanza tus funciones en procesos paralelos que el OS balanceará
    ```
3.  **Afinidad de Procesos en el Binario de C (`vspm-pm6750`):** Si el ejecutable compilado en C no cuenta con lógica nativa multi-hilo (`pthreads`) y se adueña de un solo núcleo, puedes balancearlo o limitar su afinidad externamente desde Linux usando `taskset`:
    ```bash
    # Forzar al proceso a ejecutarse exclusivamente en los núcleos 0, 1 y 2
    taskset -cp 0,1,2 <PID_DEL_PROCESO>
    ```

### 💾 Optimización de Almacenamiento eMMC (Límite: 8GB)
El almacenamiento eMMC de 8GB es veloz pero muy limitado en espacio. Si se llena, el rendimiento del sistema colapsará dramáticamente.
1.  **Desactivar Logs agresivos:** Django y Nginx pueden generar gigabytes de registros en pocos días. Configura el archivo `settings.py` de Django con niveles de Log reducidos (`WARNING` o `ERROR`) y configura `logrotate` en Linux para comprimir y eliminar logs viejos frecuentemente.
2.  **Limpieza de Caché:** Limpia regularmente los archivos temporales de Python y del gestor de paquetes:
    ```bash
    sudo apt-clean
    rm -rf __pycache__
    ```

---
*Documento generado con fines de diagnóstico técnico e ingeniería de sistemas embebidos.*
