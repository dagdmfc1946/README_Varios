# Guía de Respaldo, Clonación y Control de Calidad para Raspberry Pi CM4

Esta guía reúne de forma clara y estructurada todas las consultas, respuestas y procedimientos técnicos tratados durante la sesión. Ha sido diseñada como una referencia técnica completa orientada al control de calidad y la correcta administración de almacenamiento en entornos eMMC de Raspberry Pi.

---

## 1. Bitácora de Dudas Resolutivas

### Consulta 1: Detección de unidades en Windows y selección de destino
* **Duda:** Al conectar la Raspberry Pi CM4 (32GB eMMC), el explorador de Windows solo asigna letra a la unidad pequeña `bootfs` (F: - 256MB), mientras que la partición principal de casi 29GB figura como desconocida o invisible. ¿Se puede generar el respaldo completo seleccionando únicamente la unidad F: en Win32 Disk Imager?
* **Respuesta:** **Sí, es completamente posible.** Win32 Disk Imager no trabaja a nivel de archivos individuales ni de letras de particiones lógicas, sino a nivel de **bloques físicos**. Al seleccionar la letra asignada a la partición de arranque (como la `[F:\]`), el programa toma como referencia todo el lector/medio de almacenamiento físico asociado. Clonará de forma secuencial desde el sector inicial hasta el final, abarcando las particiones visibles (FAT32) y las invisibles en Windows (ext4 de Linux).

### Consulta 2: Impacto de la casilla "Read Only Allocated Partitions"
* **Duda:** ¿Qué efecto tiene dejar seleccionada la casilla *Read Only Allocated Partitions* durante la fase de lectura y cómo afecta el tamaño final del archivo `.img`?
* **Respuesta:** Esta casilla le ordena a la herramienta consultar al sistema operativo anfitrión qué sectores se encuentran formalmente asignados para acortar el tiempo de copia. Dado que Windows no reconoce de manera nativa los sistemas de archivos de Linux, existe el riesgo de que ignore la partición raíz (`/`). Sin embargo, si al finalizar el proceso el archivo `.img` generado mide el tamaño nominal total de la tarjeta (~29.1 GB a 32 GB), se confirma que Win32 Disk Imager logró evadir la restricción e hizo un volcado físico íntegro de principio a fin. Para un control de calidad óptimo, lo recomendable es **desmarcar siempre esta casilla**.

### Consulta 3: Compatibilidad al migrar entre diferentes capacidades de Hardware
* **Duda:** ¿Existe algún problema o incompatibilidad al escribir una imagen generada previamente en una CM4 con especificaciones inferiores (2GB RAM / 8GB eMMC) dentro de una placa CM4 superior (8GB RAM / 32GB eMMC)?
* **Respuesta:** **No hay ningún inconveniente.** El proceso es totalmente compatible. El sistema operativo integrado (Raspberry Pi OS) mapeará y gestionará de manera inmediata y automática los nuevos 8GB de memoria RAM disponibles en el primer arranque. Con respecto al espacio de almacenamiento sobrante (los ~24GB de diferencia en eMMC), este se mantendrá inicialmente invisible como "espacio no asignado" hasta que se proceda con su respectiva expansión por software.

---

## 2. Procedimiento Técnico Paso a Paso

### Fase A: Lectura y Generación de Imagen Íntegra (.img) con Win32 Disk Imager
1. **Conexión:** Conecta la unidad eMMC de tu Raspberry Pi CM4 a la computadora mediante la interfaz de flasheo correspondiente.
2. **Definición de Ruta:** Abre Win32 Disk Imager. Haz clic en el ícono de la carpeta azul para designar el directorio de salida y asigna un nombre descriptivo al archivo (ej. `backup_prototipoKR_SHA256.img`).
3. **Selección del Dispositivo:** Despliega el menú **Device** y elige la unidad asignada por Windows (ej. `[F:\]`). 
4. **Desactivar Restricciones:** Asegúrate de **desmarcar por completo** la casilla *Read Only Allocated Partitions* para forzar un volcado físico total.
5. **Configuración de Control de Calidad (Huella Digital):** En el desplegable de la sección *Hash*, selecciona el estándar criptográfico **SHA256**.
6. **Ejecución:** Haz clic en el botón **Read**. El programa mantendrá el botón *Generate* bloqueado mientras calcula matemáticamente el Hash en tiempo real conforme copia los sectores.
7. **Verificación Física:** Al completarse la barra al 100%, haz clic en el botón **Copy** para almacenar la cadena de texto alfanumérica obtenida. Pégala en un documento de texto plano (`.txt`) con el mismo nombre de tu archivo de imagen para fines documentales.

### Fase B: Escritura de Imagen e Implementación Automática con Raspberry Pi Imager
1. **Preparación:** Inicia la aplicación **Raspberry Pi Imager** en tu estación de trabajo.
2. **Carga de Archivo:** En el apartado de selección de sistema operativo, elige la opción de cargar una imagen personalizada (*Use Custom*) y busca tu archivo `.img` de respaldo.
3. **Asignación de Destino:** Selecciona la nueva Compute Module 4 de destino conectada a tu PC.
4. **Flujo de Grabación:** Haz clic en **Write**. El software iniciará de inmediato dos subprocesos internos consecutivos:
   * **Etapa de Escritura:** Transfiere el contenido exacto de los bloques del archivo origen al chip eMMC físico.
   * **Etapa de Verificación Automatizada:** Al concluir la copia, lee el almacenamiento físico para computar el hash en tiempo real. Lo valida directamente contra la firma del archivo origen.
5. **Aprobación de Control de Calidad:** Si el programa despliega la ventana flotante con la leyenda **"Write Successful"** (Escritura exitosa), la copia es idéntica y el proceso ha concluido satisfactoriamente.

### Fase C: Expansión de Capacidad Post-Clonación
Una vez que el sistema se inicia por primera vez en el nuevo hardware con 32GB de eMMC, recupera el espacio remanente ejecutando lo siguiente:
1. Accede a la interfaz de comandos o terminal de la Raspberry Pi y ejecuta el panel maestro:
   ```bash
   sudo raspi-config
   ```
2. Desplázate con el teclado hacia la sección **Advanced Options**.
3. Selecciona la opción **Expand Filesystem** y presiona la tecla Enter.
4. Confirma la acción, dirígete a **Finish** y autoriza el reinicio inmediato del sistema. El software redimensionará de manera segura la partición ext4 cubriendo la totalidad del chip.

---

## 3. Fundamentos de Integridad y Control de Calidad (Hashes)

La sección **Hash** es el pilar de validación matemática en la gestión de imágenes de disco. Actúa como una firma digital de longitud fija asociada rigurosamente a la estructura binaria del archivo. Modificar un único bit de datos alterará por completo la cadena alfanumérica resultante (Efecto Avalancha).

### Análisis Comparativo de Algoritmos Disponibles

| Algoritmo | Nivel de Seguridad | Costo de Procesamiento | Caso de Uso Recomendado |
| :--- | :--- | :--- | :--- |
| **None** | Nulo (Sin comprobación) | Ninguno (Máxima rapidez) | Copias informales o de baja prioridad donde el tiempo es el factor crítico. |
| **MD5** | Estándar / Medio | Bajo | Verificaciones rápidas en hardware antiguo o copias rutinarias de desarrollo. |
| **SHA256** | Alto (Estándar Industrial) | Alto | Entornos de producción, despliegues masivos y auditorías rígidas de control de calidad. |

### Procedimiento Manual de Verificación con PowerShell (Auditoría Cruzada)

Si necesitas contrastar de forma independiente la integridad de tu clonación sin depender únicamente de la interfaz del software de grabación, implementa un control cruzado manual:

1. **Hash de Origen:** Obtén la firma digital del archivo de respaldo almacenado en el disco de tu PC abriendo PowerShell y ejecutando:
   ```powershell
   Get-FileHash "C:\Ruta\De\Tu\backup_prototipoKR_SHA256.img" -Algorithm SHA256
   ```
2. **Hash de Destino:** Tras haber grabado la imagen en la nueva CM4 utilizando Raspberry Pi Imager, mantén conectada la unidad a la PC, abre **Win32 Disk Imager**, selecciona su unidad correspondiente (`Device`), cambia el desplegable a **SHA256** y pulsa en **Verify Only** (o *Generate* según la disponibilidad del botón).
3. **Cotejo:** Si la cadena de caracteres alfanuméricos provista por PowerShell coincide de manera exacta con la calculada sobre el medio físico por Win32 Disk Imager, la transferencia de datos cuenta con una fiabilidad del 100%.