# README - Solución de Problemas de Pantalla o Display DSI en Raspberry Pi 🍇🐍👨🏽‍💻

## Descripción del Problema

> [!NOTE]
> Este documento describe la solución completa para un problema donde la Raspberry Pi  no muestra imagen en la pantalla o display DSI ni en el puerto HDMI, a pesar de tener conectados ambos dispositivos.

### Síntomas Observados

- **Display DSI**: Pantalla completamente negra, sin imagen y posiblemente sin respuesta táctil.
- **Puerto HDMI**: Sin señal de video.
- **Mensajes de error**: 
  - `"Se descarta fichero de bloqueo incorrecto: /etc/xdg/lxsession/LXDE-pi/.autostart.swp"`
  - `"your 131072x1 screen size is bogus. expect trouble"`
  - `"[drm] forcing HDMI-A-1 connector on"`
  - `"Bogus possible_crtcs: [ENCODER:31:DSI-31] possible_crtcs=0x0"`

---

## Diagnóstico del Problema

### 1. Archivos de Bloqueo Corruptos
- **Problema**: Archivos `.swp` y `.lock` corruptos en el sistema de sesión LXDE.
- **Ubicación**: `/etc/xdg/lxsession/LXDE-pi/`
- **Causa**: Apagado incorrecto del sistema o conflictos de sesión.

### 2. Configuración de Pantalla Incorrecta
- **Problema**: `ignore_lcd=1` en `/boot/config.txt` deshabilitaba el display DSI.
- **Efecto**: El sistema ignoraba completamente la pantalla DSI.

### 3. Forzado de HDMI en el Kernel
- **Problema**: Línea `video=HDMI-A-1:1280x720M@60D` en `/boot/cmdline.txt`
- **Efecto**: El kernel forzaba HDMI e ignoraba el display DSI.

### 4. Conflicto de Modo KMS
- **Problema**: Configuraciones mezcladas (overlays KMS/firmware KMS y overlays genéricos) interferían con el display DSI.
- **Efecto**: El driver `vc4-drm` no se enlazaba correctamente al DSI y se priorizaba HDMI.

---

## Solución Paso a Paso

### Paso 1: Limpieza de Archivos de Bloqueo

```bash
# Eliminar archivos de bloqueo corruptos
sudo rm -f /etc/xdg/lxsession/LXDE-pi/.autostart.lock
sudo rm -f /etc/xdg/lxsession/LXDE-pi/.autostart.swp
sudo rm -f /tmp/.X*-lock
sudo rm -f /tmp/.X11-unix/X*

# Corregir permisos del directorio de sesión
sudo chown -R pi:pi /etc/xdg/lxsession/LXDE-pi/
sudo chmod 755 /etc/xdg/lxsession/LXDE-pi/
```

### Paso 2: Corrección de Configuración de Pantalla

**Editar `/boot/config.txt`:**
```bash
sudo nano /boot/config.txt
```

**Cambiar:**
```
ignore_lcd=1
```

**Por:**
```
#ignore_lcd=1
```

### Paso 3: Eliminación de Forzado HDMI en el Kernel

**Editar `/boot/cmdline.txt`:**
```bash
sudo nano /boot/cmdline.txt
```

**Eliminar la línea:**
```
video=HDMI-A-1:1280x720M@60D
```

### Paso 4: Habilitar KMS nativo y priorizar DSI

**Editar `/boot/config.txt`:**
> [!IMPORTANT]
> - Asegurarse que esté esta línea UNA sola vez:
```
dtoverlay=vc4-kms-v3d
```
- Mantener ACTIVOS los overlays del panel/táctil DSI (propietarios del fabricante):
```
dtoverlay=WS_xinchDSI_Screen,SCREEN_type=8,I2C_bus=10
dtoverlay=WS_xinchDSI_Touch,I2C_bus=10,invertedx,swappedxy
```
- Dejar COMENTADOS (o eliminar) los overlays genéricos que no se usen o causen algún error o falla:
```
#dtoverlay=vc4-dsi
#dtoverlay=vc4-dsi-ts
#dtoverlay=vc4-kms-dsi-7inch
#dtoverlay=vc4-kms-dsi-generic
```

### Paso 5: HDMI solo como respaldo (opcional)

> [!NOTE]
> En la mayoría de casos no es necesario forzar HDMI. Si se requiere de emergencia o respaldo, se puede habilitarlo temporalmente:
```
#hdmi_force_hotplug=1
#hdmi_drive=2
#hdmi_group=1
#hdmi_mode=4
```
> [!TIP]
> Recomendación: mantenlos comentados para no interferir con el display DSI.

### Paso 6: Limpieza de duplicados y overlays en conflicto

```bash
# Dejar solo una línea de vc4-kms-v3d
sudo sed -i '/^dtoverlay=vc4-kms-v3d$/d' /boot/config.txt && echo 'dtoverlay=vc4-kms-v3d' | sudo tee -a /boot/config.txt

# Asegurar que NO queden overlays DSI genéricos si usas los del fabricante
sudo sed -i '/^dtoverlay=vc4-dsi$/d; /^dtoverlay=vc4-dsi-ts$/d; /^dtoverlay=vc4-kms-dsi-7inch$/d; /^dtoverlay=vc4-kms-dsi-generic$/d' /boot/config.txt
```

### Paso 7: Reinicio del Sistema

```bash
sudo reboot
```

---

## Configuración Final

### Archivo `/boot/config.txt` (estado final conocido bueno)

```
# Configuración básica
gpu_mem=128
display_auto_detect=1
max_framebuffers=2
disable_overscan=1

# DSI del fabricante (activos)
dtoverlay=WS_xinchDSI_Screen,SCREEN_type=8,I2C_bus=10
dtoverlay=WS_xinchDSI_Touch,I2C_bus=10,invertedx,swappedxy

# KMS nativo (una sola vez)
dtoverlay=vc4-kms-v3d

# Overlays DSI genéricos (dejarlos comentados)
#dtoverlay=vc4-dsi
#dtoverlay=vc4-dsi-ts
#dtoverlay=vc4-kms-dsi-7inch
#dtoverlay=vc4-kms-dsi-generic

# HDMI como respaldo (opcional; mantener comentado usualmente)
#hdmi_force_hotplug=1
#hdmi_drive=2
#hdmi_group=1
#hdmi_mode=4
```

### Archivo `/boot/cmdline.txt`

```
console=tty1 root=PARTUUID=4d68c4a5-02 rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles
```

---

## Verificación de la Solución

### Comandos de Verificación

```bash
# Verificar que no hay archivos de bloqueo
ls -la /etc/xdg/lxsession/LXDE-pi/ | grep -E "\.(lock|swp|tmp)"

# Verificar configuración KMS/DSI activa
grep -n "vc4\|kms\|WS_xinch" /boot/config.txt

# Verificar conectores DRM
ls -1 /sys/class/drm && for s in /sys/class/drm/*/status; do echo $s: $(cat $s 2>/dev/null); done

# Verificar logs de binding de vc4/DSI
sudo dmesg | grep -i -e vc4 -e dsi -e drm | tail -120
```

### Indicadores de Éxito

- ✅ **Display DSI**: Muestra imagen correctamente en la pantalla o display DSI.
- ✅ **Puerto HDMI**: Funciona como respaldo.
- ✅ **Sin errores**: No aparecen mensajes de error o de fallas.
- ✅ **Sistema estable**: LightDM y XWayland funcionan correctamente.

---

## Explicación Técnica

### ¿Por qué Funcionó?

1. **Eliminación de `ignore_lcd=1`**: Permite que el sistema detecte el display DSI.
2. **KMS nativo (vc4-kms-v3d)**: Proporciona acceso completo a los CRTCs para el DSI y HDMI.
3. **Sin forzados HDMI**: Evita que HDMI opaque o se superponga al display DSI.
4. **Overlays DSI del fabricante**: Aseguran timings/panel correctos (el táctil ya funcionaba).

### Bitácora de recuperación final (ejemplo de éxito)
> [!NOTE]
> Indicadores clave en `dmesg` cuando todo está OK:
```
vc4-drm gpu: bound fe700000.dsi (ops vc4_dsi_ops [vc4])
[drm] Initialized vc4 0.0.0 20140616 for gpu on minor 1
vc4-drm gpu: [drm] fb0: vc4drmfb frame buffer device
```
> [!IMPORTANT]
> Y conectores presentes:
```
card1-DSI-1
card1-HDMI-A-1
card1-HDMI-A-2
```

### Modo KMS vs Firmware KMS

- **Firmware KMS**: Limitado, puede causar conflictos con displays DSI.
- **KMS nativo**: Acceso completo al hardware, mejor compatibilidad con DSI.

---

## Prevención en caso de Problemas Futuros

### Recomendaciones

> [!CAUTION]
> 1. **Apagado correcto**: Usar `sudo shutdown -h now` o `sudo reboot`.
> 2. **Backup de configuración**: Mantener copias de `/boot/config.txt` y `/boot/cmdline.txt`.
> 3. **Configuración incremental**: Hacer cambios uno por uno y probar.
> 4. **Monitoreo de logs**: Revisar `dmesg` periódicamente para detectar problemas temprano:

```bash
# 1. Ver logs recientes del sistema gráfico:
# Ver logs de los últimos 2 minutos relacionados con DSI/HDMI/DRM
sudo dmesg | grep -i "dsi\|hdmi\|drm\|vc4" | tail -50
# Ver logs de los últimos 5 minutos
sudo dmesg --since "5 minutes ago" | grep -i "display\|screen\|kms"

# 2. Monitoreo en tiempo real:
# Ver logs en tiempo real (útil durante reinicios)
sudo dmesg -w | grep -i "dsi\|hdmi\|drm"
# Ver logs de los últimos 10 minutos en tiempo real
sudo dmesg --since "10 minutes ago" -w

# 3. Búsquedas específicas por tipo de problema:
# Buscar errores específicos
sudo dmesg | grep -i "error\|fail\|warning" | tail -30
# Buscar problemas de overlays
sudo dmesg | grep -i "failed to load overlay\|overlay" | tail -20
# Buscar problemas de binding DRM
sudo dmesg | grep -i "bound\|binding\|gpu" | tail -30

# 4. Monitoreo periódico (para scripts):
# Ver logs de la última hora
sudo dmesg --since "1 hour ago" | grep -i "dsi\|hdmi"
# Ver logs del último boot
sudo dmesg | grep -i "vc4\|drm\|gpu" | tail -100
# Ver logs con timestamp
sudo dmesg -T | grep -i "display\|screen" | tail -50
```

```bash
# Ejemplos prácticos:

# 1. Para verificar que el DSI está funcionando:
# Buscar indicadores de éxito (como menciona el README)
sudo dmesg | grep -i "vc4.*bound\|drm.*bound\|gpu.*bound" | tail -20
# Ver si el DSI se enlazó correctamente
sudo dmesg | grep -i "fe700000.dsi\|vc4_dsi_ops" | tail -10

# 2. Para detectar problemas temprano:
# Monitoreo continuo durante cambios de configuración
sudo dmesg -w | grep -i "error\|fail\|warning\|dsi\|hdmi"
# Ver logs del último reinicio
sudo dmesg | grep -i "vc4\|drm\|kms" | tail -80
```

### Archivos de Backup
> [!WARNING]
> Crear backups para tener un respaldo si se realizan modificaciones adicionales.
```bash
# Crear backups antes de cambios
sudo cp /boot/config.txt /boot/config.txt.backup
sudo cp /boot/cmdline.txt /boot/cmdline.txt.backup
```

---

## Troubleshooting Adicional

### Si el Problema Persiste

1. **Verificar conectores físicos**: Cables en display DSI y HDMI que estén bien conectados.
2. **Revisar alimentación**: Display DSI requiere alimentación adecuada.
3. **Verificar overlays**: Asegurar que los overlays DSI sean correctos para el modelo del display DSI.
4. **Revisar logs del kernel**: `sudo dmesg | grep -i error`.

### Comandos de Diagnóstico

```bash
# Estado del sistema gráfico
sudo systemctl status lightdm

# Procesos gráficos activos
ps aux | grep -E "(X|wayland|mutter)"

# Información del framebuffer
cat /sys/class/graphics/fb0/modes

# Estado de los overlays
vcgencmd get_mem gpu
```

---

## Conclusión

Esta solución aborda múltiples capas del problema:
- **Nivel de archivos**: Limpieza de archivos corruptos.
- **Nivel de configuración**: Corrección de parámetros del sistema.
- **Nivel del kernel**: Configuración correcta de KMS y el display DSI.
- **Nivel de hardware**: Configuración adecuada para displays DSI.

El resultado es un sistema que funciona correctamente tanto con display DSI como con HDMI, proporcionando redundancia y estabilidad.

---

> [!NOTE]
> **Fecha de creación**: Agosto 2025  
> **Sistema**: Raspberry Pi con display DSI WS_xinchDSI  
> **Problema resuelto**: Pantalla negra en DSI y HDMI  
> **Estado**: ✅ FUNCIONANDO
> 
> **Made by:** @dagdmfc
> 
