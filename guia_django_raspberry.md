# Guía Completa de Desarrollo y Despliegue Local con Django y Flask (Raspberry Pi CM4)

Este documento recopila las consultas, explicaciones y recomendaciones técnicas abordadas para optimizar el funcionamiento de aplicaciones Python en entornos de hardware embebido.

---

## 1. Conceptos Fundamentales

### ¿Qué es un Framework?
Un **framework** es una estructura o plantilla de trabajo prefabricada que ayuda a los programadores a desarrollar aplicaciones de forma más rápida y organizada. En lugar de escribir todo el código desde cero, proporciona los "bloques de construcción" básicos (como herramientas de seguridad, conexión a bases de datos y manejo de usuarios). 
*   **Metáfora:** Piensa en él como el **chasis de un automóvil**: ya tiene las ruedas y el motor base; el desarrollador solo se encarga de elegir el diseño y los detalles de los interiores.

### Django vs. Flask
Tanto **Django** como **Flask** son frameworks creados para el lenguaje de programación **Python**, pero tienen filosofías de trabajo completamente opuestas:

*   🧱 **Django ("Con todo incluido" / *Batteries-Included*):** Diseñado para crear aplicaciones web grandes y complejas rápidamente. Trae herramientas integradas para casi todo (panel de administración automático, ORM para bases de datos y seguridad avanzada). Es como una **casa prefabricada**: ya viene con las habitaciones divididas; no puedes cambiar fácilmente la distribución, pero te mudas de inmediato.
*   🍃 **Flask (Microframework):** Es minimalista, ligero y no impone una estructura fija de archivos. Solo proporciona lo mínimo para que una página web funcione. Si necesitas bases de datos o usuarios, debes elegir e instalar herramientas externas. Es como un **terreno vacío**: tú decides exactamente dónde poner cada ladrillo, lo que otorga total libertad pero exige más configuración.

---

## 2. Aplicaciones en Dispositivos Físicos (Edge Computing)

Cuando un sistema web se ejecuta dentro de un hardware embebido como una **Raspberry Pi CM4** para interactuar en el sitio, se denomina aplicación de **computación en el borde** (*Edge Computing*). No se requiere un despliegue tradicional en la nube (internet) para que funcione; todo puede ejecutarse de forma **100% local** en cada dispositivo. Sin embargo, el término "despliegue" aquí aplica a la configuración interna del sistema operativo para garantizar estabilidad, persistencia y seguridad.

---

## 3. Problema del Almacenamiento Lleno por Logs (Solución)

Al ejecutar Django mediante `systemd`, Linux captura obligatoriamente todo lo que el programa imprima en la consola (Standard Output) y lo almacena en los logs del sistema (`journald`). Esto, sumado al flujo constante de peticiones HTTP en entornos limitados (como una eMMC de 8GB), satura rápidamente el espacio.

Para estabilizar el almacenamiento de raíz, se aplican tres soluciones complementarias:

### Paso 1: Desactivar logs de peticiones en Django
Agrega esta configuración al final de tu archivo `settings.py` para silenciar el output de solicitudes HTTP:

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'null': {
            'class': 'logging.NullHandler',
        },
    },
    'loggers': {
        'django.server': {
            'handlers': ['null'],
            'level': 'CRITICAL',
            'propagate': False,
        },
    },
}
```

### Paso 2: Forzar un límite estricto a Systemd (`journald`)
Edita el archivo de configuración de logs de Linux para evitar la acumulación infinita en el eMMC:

1. Abre el archivo: `sudo nano /etc/systemd/journald.conf`
2. Modifica o añade las siguientes líneas dentro de `[Journal]`:
   ```ini
   Storage=volatile       # Guarda los logs SOLO en la memoria RAM (se borran al reiniciar)
   SystemMaxUse=20M       # Espacio máximo en disco
   RuntimeMaxUse=20M      # Espacio máximo en RAM
   ```
3. Guarda (`Ctrl+O`, `Enter`) y sal (`Ctrl+X`).
4. Aplica los cambios y libera el espacio ocupado:
   ```bash
   sudo systemctl restart systemd-journald
   sudo journalctl --vacuum-size=20M
   ```

### Paso 3: Desviar el output del servicio a la basura (`/dev/null`)
Edita el archivo `.service` de tu aplicación en `systemd` (`sudo nano /etc/systemd/system/tu_servicio.service`) y añade estas dos líneas dentro de la sección `[Service]`:

```ini
StandardOutput=null
StandardError=null
```
Recarga el demonio: `sudo systemctl daemon-reload && sudo systemctl restart tu_servicio`

---

## 4. Migración Profesional: De `runserver` a Gunicorn

### El peligro de `runserver`
El comando `python manage.py runserver` es exclusivamente para desarrollo. Es síncrono (monohilo): si el dispositivo envía datos a internet y la conexión se ralentiza, **todo el proceso de Django se congela** y no atenderá ninguna acción local. Además, consume CPU monitorizando cambios de código en tiempo real y si ocurre un error crítico, el servidor muere por completo.

### Las ventajas de Gunicorn
**Gunicorn** (*Green Unicorn*) es un servidor web de producción de grado industrial para Python. 
1. **Silencioso:** No genera logs de acceso de forma masiva por defecto.
2. **Inmortal (Workers):** Crea procesos clonados (*workers*). Si un proceso falla, Gunicorn lo destruye y crea uno nuevo en milisegundos, evitando que la Raspberry Pi se cuelgue.
3. **Eficiente:** Consume mucha menos CPU y RAM que `runserver`.

### Pasos para migrar e implementar Gunicorn

1. **Instalar en la Raspberry Pi:**
   ```bash
   pip install gunicorn
   ```

2. **Probar el funcionamiento (Desde la raíz del proyecto):**
   ```bash
   gunicorn mi_proyecto.wsgi:application --bind 0.0.0.0:8000 --workers 2
   ```
   *(Reemplaza `mi_proyecto` por el nombre de la carpeta contenedora de tu archivo `wsgi.py`)*

3. **Automatización definitiva en Systemd:**
   Modifica tu archivo `.service` para arrancar con Gunicorn en producción local de manera limpia y resiliente. Ejemplo de estructura recomendada:

   ```ini
   [Unit]
   Description=Servicio Django con Gunicorn para Raspberry Pi
   After=network.target

   [Service]
   User=pi
   WorkingDirectory=/home/pi/tu_proyecto
   ExecStart=/home/pi/tu_entorno_virtual/bin/gunicorn mi_proyecto.wsgi:application --bind 0.0.0.0:8000 --workers 2
   Restart=always
   StandardOutput=null
   StandardError=null

   [Install]
   WantedBy=multi-user.target
   ```

---
*Fin del documento de respaldo técnico.*
