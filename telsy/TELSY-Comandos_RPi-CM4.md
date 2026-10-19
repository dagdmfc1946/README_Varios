# TELSY — Comandos CM4 e Imagen MASTER

## Propósito

Recopilación de los comandos indicados durante el análisis del prototipo TELSY con Raspberry Pi Compute Module 4 (CM4): diagnóstico, almacenamiento, configuración, LXDE Autostart, apagado seguro y preparación de la imagen MASTER.

> **IMPORTANTE:** no ejecutar comandos de escritura sobre discos, especialmente `dd`, hasta identificar inequívocamente el dispositivo correcto. Un error puede destruir información.

---

## 1. Conexión SSH

```bash
ssh usuario@IP_DEL_PROTOTIPO
```

**Descripción:** establece una conexión SSH con la CM4.

**Uso principal:** administrar la Raspberry Pi remotamente.

---

## 2. Sistema operativo

```bash
cat /etc/os-release
```

**Descripción:** muestra información de la distribución Linux.

**Uso principal:** confirmar distribución y versión.

**Prototipo:** Debian 11 / Bullseye.

---

## 3. Kernel

```bash
uname -a
```

**Descripción:** muestra información del kernel, arquitectura y sistema.

**Uso principal:** documentar el entorno de ejecución.

---

## 4. Python

```bash
python3 --version
```

**Descripción:** muestra la versión de Python 3.

**Uso principal:** verificar el intérprete utilizado.

**Prototipo:** Python 3.9.2.

---

## 5. Arquitectura

```bash
dpkg --print-architecture
```

**Descripción:** muestra la arquitectura de paquetes configurada.

**Uso principal:** confirmar `arm64`, `armhf`, etc.

---

# Almacenamiento

## 6. Identificar discos

```bash
lsblk
```

**Descripción:** muestra dispositivos de almacenamiento y particiones.

**Uso principal:** identificar eMMC, microSD y discos USB.

> **Comando obligatorio antes de `dd`.**

---

## 7. Información detallada de discos

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS
```

Si `MOUNTPOINTS` no está disponible:

```bash
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINT
```

**Descripción:** muestra nombre, tamaño, filesystem, etiqueta y montaje.

---

## 8. Tabla de particiones

```bash
sudo fdisk -l
```

**Descripción:** lista discos y tablas de particiones.

**Uso principal:** confirmar estructura y capacidad.

---

## 9. Montaje de `/dev/sda2`

```bash
findmnt /dev/sda2
```

**Descripción:** muestra dónde está montada una partición.

**Contexto:** `/dev/sda2` correspondía a la microSD temporal del backup.

---

## 10. Montajes de eMMC/microSD

```bash
mount | grep -E 'mmcblk0|sda'
```

**Descripción:** filtra los sistemas de archivos montados relacionados con `mmcblk0` y `sda`.

---

## 11. Directorio temporal

```bash
ls -lah /tmp/tmp.gdzlVd5Vc1
```

**Descripción:** inspecciona un directorio temporal concreto.

**Nota:** puede no existir posteriormente.

---

## 12. Espacio disponible

```bash
df -h
```

**Descripción:** muestra espacio usado y disponible.

---

## 13. Uso de espacio en `/`

```bash
sudo du -xhd1 / 2>/dev/null | sort -h
```

**Descripción:** calcula el uso de los directorios de primer nivel.

**Parámetros:** `-x` evita cruzar otros filesystems, `-h` usa unidades legibles, `-d1` limita profundidad, `sort -h` ordena por tamaño.

---

## 14. Uso de `/home/pi`

```bash
sudo du -xhd2 /home/pi 2>/dev/null | sort -h | tail -30
```

**Descripción:** analiza hasta dos niveles dentro de `/home/pi`.

**Uso principal:** localizar proyectos y directorios grandes.

---

## 15. Uso de `/opt`

```bash
sudo du -xhd1 /opt 2>/dev/null | sort -h
```

**Descripción:** muestra uso de espacio dentro de `/opt`.

**Hallazgo:** `/opt/Wolfram` ocupaba aproximadamente 4 GB.

---

## 16. Directorios de `/opt`

```bash
sudo find /opt -maxdepth 2 -type d -print
```

**Descripción:** lista directorios de `/opt` hasta profundidad 2.

---

## 17. Uso de `/root`

```bash
sudo du -xhd1 /root 2>/dev/null | sort -h
```

**Descripción:** muestra uso de `/root`.

**Hallazgo:** `/root/.rustup` ocupaba aproximadamente 1,5 GB.

---

# Software y servicios

## 18. Paquetes relacionados

```bash
dpkg -l | grep -E 'python3|django|smbus|serial|gpio|channels|daphne|plymouth'
```

**Descripción:** filtra paquetes instalados relacionados con Python, Django, serial, GPIO, Channels, Daphne y Plymouth.

---

## 19. Paquetes Python

```bash
python3 -m pip list
```

**Descripción:** lista paquetes Python del intérprete indicado.

**Nota:** preferible a `pip list` porque asocia explícitamente pip con `python3`.

---

## 20. Servicios habilitados

```bash
systemctl list-unit-files --type=service --state=enabled
```

**Descripción:** muestra servicios configurados para iniciar automáticamente.

---

## 21. Servicios activos

```bash
systemctl --type=service --state=running
```

**Descripción:** muestra servicios actualmente activos.

---

## 22. Buscar servicios de aplicación

```bash
sudo find /etc/systemd -type f | grep -i -E 'django|daphne|python|nombre_del_proyecto'
```

**Descripción:** busca configuraciones systemd relacionadas con la aplicación.

**Nota:** `nombre_del_proyecto` era un marcador. El análisis determinó que el arranque funcional actual se realiza mediante LXDE Autostart.

---

## 23. Dispositivos serie

```bash
ls /dev/tty*
```

**Descripción:** lista dispositivos TTY.

**Uso principal:** identificar UART y USB-Serial.

---

## 24. Interfaces I²C

```bash
ls /dev/i2c*
```

**Descripción:** lista interfaces I²C disponibles.

---

## 25. Estado de I²C

```bash
sudo raspi-config nonint get_i2c
```

**Descripción:** consulta si I²C está habilitado.

---

# Configuración

## 26. Configuración activa de `config.txt`

```bash
grep -v '^#' /boot/config.txt | sed '/^$/d'
```

**Descripción:** muestra líneas activas y no vacías de `config.txt`.

**Uso:** revisar overlays, UART, I²C, cámara, pantalla, etc.

---

## 27. Parámetros de arranque

```bash
cat /boot/cmdline.txt
```

**Descripción:** muestra los parámetros utilizados por el kernel durante el arranque.

---

# LXDE Autostart

## 28. Autostart del usuario

```bash
ls -lah /home/pi/.config/autostart
```

**Descripción:** lista archivos de inicio automático del usuario `pi`.

---

## 29. Mostrar Autostart del usuario

```bash
for f in /home/pi/.config/autostart/*; do
    echo "===== $f ====="
    cat "$f"
    echo
done
```

**Descripción:** muestra el contenido de todos los archivos de autostart del usuario.

---

## 30. Configuración LXDE-pi

```bash
ls -lah /home/pi/.config/lxsession/LXDE-pi/
```

**Descripción:** lista configuración LXDE-pi del usuario.

---

## 31. Buscar archivos LXDE

```bash
find /home/pi/.config/lxsession/LXDE-pi -maxdepth 2 -type f -print
```

**Descripción:** localiza archivos de configuración de LXDE-pi.

---

## 32. Autostart global — IMPORTANTE

```bash
cat /etc/xdg/lxsession/LXDE-pi/autostart
```

**Descripción:** muestra la configuración global de inicio automático de LXDE-pi.

**En el prototipo se confirmó:**

```text
@sh /home/pi/telsy-monitor/startserver.sh
@sh /home/pi/telsy-monitor/startweb.sh
@sh /home/pi/telsy-monitor/run_driver.sh
```

Estas rutas son relevantes para la imagen MASTER.

---

## 33. Buscar Autostart global

```bash
find /etc/xdg -maxdepth 4 -type f 2>/dev/null | grep -i autostart
```

**Descripción:** localiza archivos `autostart` dentro de `/etc/xdg`.

---

# Automatización

## 34. Cron del usuario

```bash
crontab -l
```

**Descripción:** muestra tareas cron del usuario actual.

---

## 35. Cron de root

```bash
sudo crontab -l
```

**Descripción:** muestra tareas cron de root.

---

## 36. Comprobar `rc.local`

```bash
ls -lah /etc/rc.local
```

**Descripción:** comprueba si existe `rc.local`.

---

## 37. Mostrar `rc.local`

```bash
cat /etc/rc.local 2>/dev/null
```

**Descripción:** muestra su contenido.

---

## 38. Buscar configuración del usuario

```bash
find /home/pi/.config -maxdepth 4 -type f 2>/dev/null
```

**Descripción:** lista archivos de configuración del usuario.

---

## 39. Buscar proyectos

```bash
find /home/pi -maxdepth 3 -type d 2>/dev/null | head -100
```

**Descripción:** busca directorios de proyectos y respaldos.

**Proyectos relevantes encontrados:**

```text
GPS_APP
tvsm
tvsm-vspm
telsy-monitor
Repositorio_GITLAB_TelsyCOREA
```

`Repositorio_GITLAB_TelsyCOREA` se consideró backup inicial y no parte funcional del proyecto.

---

# Apagado

## 40. Apagado seguro

```bash
sudo shutdown -h now
```

**Descripción:** realiza un apagado controlado inmediato.

**Uso:** apagar la CM4 antes de retirar alimentación o modificar el estado de boot.

---

# Imagen MASTER

## 41. Crear imagen RAW

> ⚠️ **NO EJECUTAR TODAVÍA.** Primero identificar el eMMC mediante `lsblk` y `fdisk`.

```bash
sudo dd if=/dev/sdb of=BACKUP_RAW.img bs=16M status=progress conv=sync,noerror
```

**Descripción:** copia sector a sector un dispositivo completo a un archivo RAW.

| Parámetro | Función |
|---|---|
| `sudo` | Privilegios administrativos |
| `dd` | Copia por bloques |
| `if=` | Dispositivo de entrada |
| `of=` | Archivo de salida |
| `bs=16M` | Bloques de 16 MiB |
| `status=progress` | Muestra progreso |
| `conv=sync` | Rellena/sincroniza bloques |
| `noerror` | Continúa ante errores de lectura |

> `/dev/sdb` es solo un ejemplo. **Nunca asumirlo como eMMC.**

---

## 42. Verificar tamaño

```bash
ls -lh BACKUP_RAW.img
```

**Descripción:** muestra tamaño del archivo de imagen.

---

## 43. Calcular SHA-256

```bash
sha256sum BACKUP_RAW.img
```

**Descripción:** calcula el hash SHA-256.

**Uso:** verificar integridad de la imagen.

---

## 44. Guardar SHA-256

```bash
sha256sum BACKUP_RAW.img > BACKUP_RAW.sha256
```

**Descripción:** guarda el hash en un archivo.

---

# Estructura recomendada

```text
TELSY_MASTER/
├── 01_BACKUP/
│   ├── BACKUP_RAW.img
│   └── BACKUP_RAW.sha256
├── 02_WORKING/
│   └── MASTER_WORKING.img
├── 03_MASTER/
│   ├── TELSY_MASTER.img
│   └── TELSY_MASTER.sha256
└── 04_DOCUMENTACION/
    ├── hardware.txt
    ├── software.txt
    ├── particiones.txt
    └── procedimiento_produccion.md
```

---

# Comandos que NO ejecutar antes de preservar el prototipo

## Actualizar índices

```bash
sudo apt update
```

## Actualizar paquetes

```bash
sudo apt upgrade
```

## Actualización completa

```bash
sudo apt full-upgrade
```

**Motivo:** antes de generar la imagen de referencia no debemos alterar el estado funcional del prototipo.

---

# Estructura de almacenamiento observada

```text
/dev/mmcblk0       → eMMC de la CM4
/dev/mmcblk0p1     → partición boot
/dev/mmcblk0p2     → partición raíz
/dev/mmcblk0boot0  → área boot del eMMC
/dev/mmcblk0boot1  → área boot del eMMC

/dev/sda           → microSD temporal
/dev/sda1          → partición boot de la microSD
/dev/sda2          → partición rootfs de la microSD
```

> Esta identificación corresponde al estado observado durante el análisis. Antes de cualquier operación de escritura debe volver a verificarse.

---

# Diferencia eMMC / microSD

```text
CM4
 └── eMMC
      └── Sistema operativo funcional
```

```text
MicroSD
 └── Backup temporal
```

Por tanto:

> **La imagen MASTER debe obtenerse del eMMC del prototipo, no de la microSD temporal.**

---

# Flujo previsto para la imagen MASTER

```text
PROTOTIPO FUNCIONAL
        │
        ▼
APAGADO SEGURO
        │
        ▼
BOOT_OPTION_SW → RPIBOOT
        │
        ▼
USB-C MAIN BOARD → PC
        │
        ▼
rpiboot
        │
        ▼
eMMC visible en PC
        │
        ▼
lsblk / fdisk
        │
        ▼
IDENTIFICAR DISPOSITIVO CORRECTO
        │
        ▼
BACKUP_RAW.img
        │
        ▼
SHA-256
        │
        ▼
CONSERVAR BACKUP ORIGINAL
        │
        ▼
CM4 NUEVA #01
        │
        ▼
GRABAR IMAGEN
        │
        ▼
VALIDACIÓN
        │
        ▼
MASTER VALIDADA
        │
        ▼
CM4 #02 ... CM4 #50
```

---

# Procedimiento para el martes

## Fase A — Prototipo

1. No actualizar Debian.
2. No modificar paquetes.
3. No eliminar archivos del proyecto.
4. Retirar la microSD temporal.
5. Confirmar funcionamiento del prototipo.
6. Ejecutar:

```bash
sudo shutdown -h now
```

## Fase B — RPIBOOT

1. Colocar `BOOT_OPTION_SW` en la posición necesaria para RPIBOOT.
2. Conectar USB-C de la Main Board al PC.
3. Alimentar la Main Board según el procedimiento establecido.
4. Ejecutar `rpiboot` en el PC.
5. Esperar a que el eMMC aparezca en el PC.

> `rpiboot` se ejecutará en el PC, no en la CM4.

## Fase C — Identificar eMMC

```bash
lsblk
```

Después:

```bash
sudo fdisk -l
```

**No ejecutar `dd` hasta identificar inequívocamente el eMMC.**

## Fase D — Crear backup

```bash
sudo dd if=/DEV_EMMC of=BACKUP_RAW.img bs=16M status=progress conv=sync,noerror
```

Luego:

```bash
ls -lh BACKUP_RAW.img
```

```bash
sha256sum BACKUP_RAW.img
```

```bash
sha256sum BACKUP_RAW.img > BACKUP_RAW.sha256
```

---

# Regla crítica para `dd`

Nunca ejecutar:

```bash
sudo dd if=/dev/sdb ...
```

simplemente porque un tutorial lo indique.

Primero:

```bash
lsblk
```

Después:

```bash
sudo fdisk -l
```

Solo cuando esté confirmado el dispositivo:

```bash
sudo dd if=/DEV_EMMC ...
```

> **La identificación correcta del dispositivo es el punto crítico de seguridad de todo el procedimiento.**

---

# Producción de 50 unidades

No grabar inmediatamente las 50 CM4.

Proceso recomendado:

```text
Imagen del prototipo
        ↓
Backup RAW intacto
        ↓
Imagen de trabajo
        ↓
CM4 nueva #01
        ↓
Pruebas completas
        ↓
Validación
        ↓
MASTER
        ↓
CM4 #02 ... CM4 #50
```

La CM4 nueva #01 debe utilizarse como **unidad piloto**.

---

# Validación de la MASTER

Antes de utilizarla para producción validar:

1. Arranque.
2. Red.
3. Ethernet.
4. USB.
5. HDMI/LCD.
6. Cámara.
7. Sensores.
8. Speaker.
9. LEDs.
10. GPIO.
11. Serial.
12. Aplicación TELSY.
13. Autostart.
14. Servicios auxiliares.
15. Comunicación entre PCBs.
16. Reinicio.
17. Apagado y encendido.

---

# Estado conocido del prototipo

```text
Sistema operativo: Debian 11 / Bullseye
Arquitectura: ARM64
Python: 3.9.2

Almacenamiento interno:
    /dev/mmcblk0

MicroSD temporal:
    /dev/sda

Arranque de aplicación:
    LXDE Autostart

Comandos de Autostart:

    @sh /home/pi/telsy-monitor/startserver.sh
    @sh /home/pi/telsy-monitor/startweb.sh
    @sh /home/pi/telsy-monitor/run_driver.sh
```

Proyectos relevantes:

```text
/home/pi/GPS_APP
/home/pi/tvsm
/home/pi/tvsm-vspm
/home/pi/telsy-monitor
/home/pi/Repositorio_GITLAB_TelsyCOREA
```

---

# Comandos fundamentales para el martes

```bash
sudo shutdown -h now
```

```bash
lsblk
```

```bash
sudo fdisk -l
```

```bash
sudo dd if=/DEV_EMMC of=BACKUP_RAW.img bs=16M status=progress conv=sync,noerror
```

```bash
ls -lh BACKUP_RAW.img
```

```bash
sha256sum BACKUP_RAW.img
```

```bash
sha256sum BACKUP_RAW.img > BACKUP_RAW.sha256
```

---

# Nivel de precaución

| Comando | Riesgo | Uso |
|---|---:|---|
| `lsblk` | Bajo | Identificar discos |
| `fdisk -l` | Bajo | Consultar particiones |
| `findmnt` | Bajo | Consultar montajes |
| `df -h` | Bajo | Consultar espacio |
| `du` | Bajo | Consultar uso |
| `cat` | Bajo | Consultar archivos |
| `grep` | Bajo | Filtrar información |
| `find` | Bajo | Buscar archivos |
| `systemctl` de consulta | Bajo | Diagnóstico |
| `shutdown` | Medio | Apagar sistema |
| `apt update` | Medio | Actualizar índices |
| `apt upgrade` | Alto para este procedimiento | Modificar prototipo |
| `apt full-upgrade` | Alto para este procedimiento | Modificar dependencias |
| `dd` de lectura | Alto | Riesgo de seleccionar disco incorrecto |
| `dd` de escritura | **CRÍTICO** | Puede destruir un disco completo |

---

# Regla de oro

> **Primero preservar el prototipo. Después generar la imagen. Después validar una CM4 nueva. Finalmente producir las 50.**

No modificar el prototipo antes de tener una copia RAW íntegra y verificable.

---

# Próximo paso

El martes el proceso debe comenzar desde el prototipo funcional, **sin actualizarlo ni modificarlo**:

```text
eMMC del prototipo
        ↓
RPIBOOT
        ↓
PC
        ↓
lsblk / fdisk
        ↓
BACKUP_RAW.img
        ↓
SHA-256
```

Una vez obtenida y verificada la imagen, se dispondrá de una copia de referencia antes de realizar cualquier operación adicional.
