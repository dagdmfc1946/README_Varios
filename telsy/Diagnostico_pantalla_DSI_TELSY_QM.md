# Diagnóstico y recuperación de pantalla DSI en Raspberry Pi CM4 – TELSY-QM

## 1. Objetivo

Documentar el procedimiento técnico utilizado para diagnosticar una condición de **pantalla negra** en un Raspberry Pi CM4 utilizado por TELSY-QM, equipado con una pantalla Waveshare DSI de 10 pulgadas.

El documento reúne:

- arquitectura gráfica involucrada;
- síntomas observables;
- evidencias obtenidas;
- interpretación de los registros;
- procedimiento de diagnóstico;
- posibles causas;
- acciones de recuperación;
- precauciones;
- criterios para determinar si el problema está resuelto;
- procedimiento recomendado para futuras incidencias.

> **Nota:** Este documento describe principalmente el caso analizado en el CM4 identificado durante la intervención como `TVSM01`. Las conclusiones deben considerarse aplicables al mismo hardware/software únicamente cuando se mantengan las mismas versiones, módulos, Device Tree y configuración.

---

# 2. Contexto del sistema

## 2.1. Hardware y software relevante

Sistema:

```text
Raspberry Pi CM4
RAM: 2 GB
eMMC: 8 GB
Arquitectura: aarch64
Kernel: 6.1.21-v8+
Kernel build: #1642
```

Versión del kernel observada:

```bash
uname -a
```

Resultado:

```text
Linux TVSM01 6.1.21-v8+ #1642 SMP PREEMPT Mon Apr 3 17:24:16 BST 2023 aarch64 GNU/Linux
```

## 2.2. Pantalla

Pantalla Waveshare DSI de 10 pulgadas, utilizando:

```text
WS_xinchDSI_Screen
WS_xinchDSI_Screen_Interface
WS_xinchDSI_Touch
WS_xinchDSI_Touch_Interface
```

Configuración relevante de `/boot/config.txt`:

```ini
dtoverlay=vc4-kms-v3d
dtoverlay=WS_xinchDSI_Screen,SCREEN_type=8,I2C_bus=10
dtoverlay=WS_xinchDSI_Touch,I2C_bus=10,invertedx,swappedxy
```

También se encontraron:

```ini
display_auto_detect=1
max_framebuffers=2
arm_64bit=1
disable_overscan=1
ignore_lcd=1
```

---

# 3. Arquitectura gráfica esperada

La pantalla no depende directamente de Xorg ni de Chromium.

La cadena funcional es aproximadamente:

```text
Device Tree / overlays
        |
        v
Waveshare DSI
WS_xinchDSI_Screen
        |
        v
DSI
fe700000.dsi
        |
        v
VC4 DRM
vc4-drm
        |
        v
/dev/dri/card1
        |
        +---- card1-DSI-1
        |
        +---- HDMI-A-1
        |
        +---- HDMI-A-2
        |
        v
Framebuffer / DRM framebuffer
fb0
        |
        v
Xorg
        |
        v
LightDM
        |
        v
LXDE / Chromium
```

El dispositivo:

```text
/dev/dri/card0
```

corresponde al componente V3D:

```text
v3d
```

Mientras que el dispositivo gráfico que debe proporcionar las salidas DSI/HDMI es:

```text
/dev/dri/card1
```

asociado con:

```text
vc4-drm
```

---

# 4. Síntoma inicial

Después de un reinicio del CM4, la pantalla permanecía negra.

Inicialmente se verificó que:

- el sistema podía iniciar;
- TELSY tenía servicios activos;
- LightDM estaba ejecutándose;
- Xorg intentaba iniciar;
- Xorg se detenía y volvía a iniciar;
- la pantalla no mostraba la interfaz gráfica.

El problema no correspondía inicialmente a una falla evidente de almacenamiento.

El almacenamiento del sistema se encontraba aproximadamente al 79 % de utilización, con alrededor de 1,5 GB disponibles, por lo que el eMMC no constituía la explicación principal de esta incidencia.

---

# 5. Primer hallazgo: Xorg no encontraba dispositivos gráficos

Los registros de Xorg mostraron:

```text
(EE) open /dev/fb0: No such file or directory
(EE) No devices detected.
(EE) no screens found
(EE) Server terminated with error (1).
```

Interpretación:

Xorg estaba intentando iniciar, pero el sistema operativo no estaba proporcionando un dispositivo gráfico utilizable para la pantalla.

Esto permitió establecer una distinción importante:

```text
Pantalla negra
        |
        +-- ¿Chromium?
        +-- ¿LXDE?
        +-- ¿LightDM?
        +-- ¿Xorg?
        |
        +-- Problema más abajo:
             DRM / VC4 / DSI / framebuffer
```

Por tanto, modificar `startweb.sh`, Chromium o la configuración de TELSY no era una solución apropiada mientras Xorg no tuviera un dispositivo gráfico válido.

---

# 6. Verificación de DRM

## 6.1. Estado defectuoso

En el estado de pantalla negra:

```bash
ls -l /sys/class/drm/
```

mostraba únicamente:

```text
card0
renderD128
```

`card0` correspondía a:

```text
../../devices/platform/v3dbus/fec00000.v3d/drm/card0
```

No existían:

```text
card1
card1-DSI-1
card1-HDMI-A-1
card1-HDMI-A-2
```

Además:

```bash
cat /proc/fb
```

no mostraba ningún framebuffer.

Esto era consistente con:

```text
Xorg → No devices detected
```

## 6.2. Estado funcional

En el equipo con pantalla funcional:

```bash
ls -l /sys/class/drm/
```

mostraba:

```text
card0
card1
card1-DSI-1
card1-HDMI-A-1
card1-HDMI-A-2
card1-Writeback-1
renderD128
```

La diferencia `card1` / `card1-DSI-1` fue uno de los indicadores más importantes de todo el diagnóstico.

---

# 7. Identificación del dispositivo VC4

En el equipo funcional:

```bash
readlink -f /sys/class/drm/card1/device/driver
```

devolvió:

```text
/sys/bus/platform/drivers/vc4-drm
```

y:

```bash
readlink -f /sys/class/drm/card1/device
```

devolvió:

```text
/sys/devices/platform/gpu
```

Posteriormente se comprobó que en ambos equipos existía:

```text
/sys/devices/platform/gpu
```

y que ambos tenían:

```text
/sys/devices/platform/gpu/driver
    -> ../../../bus/platform/drivers/vc4-drm
```

Esto permitió descartar:

- ausencia del dispositivo `gpu`;
- ausencia del driver `vc4`;
- ausencia del enlace entre `gpu` y `vc4-drm`.

La diferencia estaba más adelante, durante la inicialización del subsistema DRM.

---

# 8. Evidencia decisiva: `/sys/devices/platform/gpu/drm/`

## 8.1. Equipo funcional

```bash
ls -l /sys/devices/platform/gpu/drm/
```

mostró:

```text
card1
controlD65 -> card1
```

## 8.2. Equipo con pantalla negra

El mismo directorio no existía:

```text
ls: no se puede acceder a '/sys/devices/platform/gpu/drm/': No existe el fichero o el directorio
```

Por tanto:

```text
gpu
 |
 +-- vc4-drm                 OK
 |
 +-- DRM device/card1        FALLA
 |
 +-- DSI connector           NO DISPONIBLE
 |
 +-- framebuffer             NO DISPONIBLE
```

Este fue el punto técnico más importante del diagnóstico.

---

# 9. Verificación de módulos Waveshare

Se comprobó que los módulos Waveshare del equipo funcional y del equipo con pantalla negra eran exactamente iguales mediante SHA-256.

## 9.1. `WS_xinchDSI_Screen.ko`

Hash:

```text
8e2e3b2abade8a005410578ca27874c6b70812568606970d5c91722be0e9332c
```

## 9.2. `WS_xinchDSI_Screen_Interface.ko`

Hash:

```text
c3ffc6d3ac409f0ebeed7cc99512f7735badd81e4f7af5af7f85c21a9a0c0f3a
```

Por lo tanto, se descartó que la diferencia entre ambos equipos correspondiera simplemente a archivos `.ko` diferentes.

---

# 10. Prueba de recarga del módulo y kernel Oops

Durante el diagnóstico se ejecutó:

```bash
sudo modprobe -r WS_xinchDSI_Screen
sudo modprobe WS_xinchDSI_Screen
```

La recarga provocó:

```text
Internal error: Oops: 0000000096000005 [#1] PREEMPT SMP
```

y posteriormente:

```text
Violación de segmento
```

El registro también mostró:

```text
WS_xinchDSI_Screen(PO+)
```

Este resultado demostró que la recarga dinámica del módulo podía provocar una excepción del kernel.

## 10.1. Precaución

A partir de esta evidencia:

> **No se debe utilizar la descarga/carga dinámica de `WS_xinchDSI_Screen` como procedimiento rutinario de diagnóstico o recuperación en producción.**

No se debe ejecutar:

```bash
sudo modprobe -r WS_xinchDSI_Screen
```

ni forzar la descarga del módulo salvo que exista un procedimiento controlado, respaldo y justificación técnica.

Una recarga incorrecta puede dejar el subsistema gráfico en un estado inconsistente y producir un kernel Oops.

---

# 11. Recuperación mediante apagado completo

Después del kernel Oops y de un reinicio que no permitió recuperar inmediatamente SSH, se realizó un apagado forzado del CM4.

Posteriormente:

1. se esperó unos segundos;
2. se volvió a energizar el dispositivo;
3. la pantalla volvió a mostrar imagen;
4. SSH volvió a estar disponible.

Esto permitió comprobar que un arranque eléctrico limpio podía recuperar correctamente la inicialización gráfica.

---

# 12. Estado correcto después del arranque limpio

Después del arranque funcional:

```bash
ls -l /sys/class/drm/
```

mostró:

```text
card0
card1
card1-DSI-1
card1-HDMI-A-1
card1-HDMI-A-2
card1-Writeback-1
renderD128
```

Y:

```bash
ls -l /sys/devices/platform/gpu/drm/
```

mostró:

```text
card1
controlD65 -> card1
```

Esto confirmó que el subsistema DRM estaba correctamente inicializado.

---

# 13. Evidencia de inicialización correcta

El registro del arranque funcional mostró:

```text
[drm] Initialized v3d 1.0.0 20180419 for fec00000.v3d on minor 0
```

Posteriormente:

```text
vc4-drm gpu: bound fe400000.hvs (ops vc4_hvs_ops [vc4])
```

```text
vc4-drm gpu: bound fef00700.hdmi (ops vc4_hdmi_ops [vc4])
```

```text
vc4-drm gpu: bound fef05700.hdmi (ops vc4_hdmi_ops [vc4])
```

Y, especialmente:

```text
vc4-drm gpu: bound fe700000.dsi (ops vc4_dsi_ops [vc4])
```

Finalmente:

```text
[drm] Initialized vc4 0.0.0 20140616 for gpu on minor 1
```

y:

```text
vc4-drm gpu: [drm] fb0: vc4drmfb frame buffer device
```

Esta secuencia representa el estado esperado.

---

# 14. Criterio técnico de funcionamiento

Para considerar correctamente inicializada la pantalla, los siguientes elementos deberían estar presentes:

## 14.1. DRM

```bash
ls -l /sys/class/drm/
```

Debe existir como mínimo:

```text
card0
card1
card1-DSI-1
renderD128
```

Las salidas HDMI y Writeback también pueden aparecer.

## 14.2. VC4

Debe existir:

```text
/sys/devices/platform/gpu/drm/card1
```

## 14.3. DSI

Debe existir:

```text
/sys/class/drm/card1-DSI-1
```

## 14.4. Framebuffer

Puede comprobarse con:

```bash
cat /proc/fb
```

y debe existir una entrada asociada con el framebuffer de VC4, por ejemplo:

```text
0 vc4drmfb
```

## 14.5. Kernel log

Debe aparecer una secuencia equivalente a:

```text
vc4-drm gpu: bound ... dsi
[drm] Initialized vc4 ...
vc4-drm gpu: [drm] fb0: vc4drmfb frame buffer device
```

---

# 15. Diagnóstico recomendado para futuras incidencias

Cuando se presente nuevamente una pantalla negra, evitar comenzar directamente con Xorg, Chromium o TELSY.

Seguir el siguiente orden.

## Paso 1 – Comprobar si el sistema está accesible

Por SSH:

```bash
ssh pi@IP_DEL_CM4
```

Si SSH funciona, continuar con el diagnóstico.

Si no funciona, determinar primero si el sistema está arrancando correctamente.

---

## Paso 2 – Comprobar DRM

```bash
ls -l /sys/class/drm/
```

### Estado correcto

Debe aparecer:

```text
card0
card1
card1-DSI-1
renderD128
```

### Estado sospechoso

Si solamente aparece:

```text
card0
renderD128
```

la investigación debe dirigirse a:

```text
VC4 / DRM / DSI
```

y no a Chromium.

---

## Paso 3 – Comprobar VC4

```bash
ls -l /sys/devices/platform/gpu/
```

Debe existir:

```text
driver -> ../../../bus/platform/drivers/vc4-drm
```

Luego:

```bash
ls -l /sys/devices/platform/gpu/drm/
```

Debe existir:

```text
card1
```

Si `/sys/devices/platform/gpu/drm/` no existe, VC4 está enlazado al dispositivo `gpu`, pero no ha completado correctamente la creación del dispositivo DRM.

---

## Paso 4 – Comprobar DSI

```bash
ls -l /sys/class/drm/card1-DSI-1
```

Si no existe, revisar la inicialización del DSI.

---

## Paso 5 – Revisar kernel log

```bash
dmesg | grep -Ei "vc4|v3d|WS_xinchDSI|dsi|drm"
```

Buscar especialmente:

```text
Initialized vc4
bound ... dsi
fb0: vc4drmfb
```

Y prestar atención a:

```text
Oops
Internal error
failed
unable
timeout
bind
```

---

# 16. Diagnóstico de Xorg

Solo después de confirmar que DRM/VC4/DSI están correctamente inicializados, revisar Xorg.

```bash
grep -Ei "\(EE\)|no screens|No devices|fb0" /var/log/lightdm/x-0.log
```

Errores como:

```text
No devices detected
no screens found
open /dev/fb0: No such file or directory
```

deben interpretarse como posibles consecuencias de una falla anterior en DRM/VC4.

No asumir inmediatamente que Xorg es la causa raíz.

---

# 17. Diagnóstico de LightDM

Comprobar:

```bash
systemctl status lightdm
```

Y:

```bash
journalctl -u lightdm -b
```

Si se observan repetidamente:

```text
X server stopped
Display server stopped
Failed to start display server
```

y simultáneamente falta `card1`, la prioridad debe mantenerse en DRM/VC4/DSI.

---

# 18. Diagnóstico de Chromium / TELSY

Una vez confirmado:

```text
card1
card1-DSI-1
fb0
Xorg
LightDM
```

entonces sí tiene sentido revisar:

```text
startweb.sh
Chromium
DISPLAY=:0
LXDE autostart
```

y posteriormente los servicios de TELSY.

El hecho de que Chromium no aparezca en pantalla no demuestra que Chromium sea la causa.

---

# 19. Posibles causas

## 19.1. Inicialización incompleta de VC4/DRM

Síntoma:

```text
gpu → vc4-drm
```

pero no existe:

```text
gpu/drm/card1
```

Consecuencia:

```text
No card1
No DSI connector
No framebuffer
Xorg no puede iniciar
```

---

## 19.2. Problema de inicialización del driver Waveshare

Los módulos utilizados son:

```text
WS_xinchDSI_Screen
WS_xinchDSI_Screen_Interface
WS_xinchDSI_Touch
WS_xinchDSI_Touch_Interface
```

Son módulos externos al árbol estándar del kernel.

El kernel reporta que algunos módulos son out-of-tree/proprietary.

Una incompatibilidad entre módulo, Device Tree, kernel o estado del subsistema puede impedir la correcta inicialización del display.

---

## 19.3. Recarga dinámica del módulo

La prueba realizada demostró que:

```bash
sudo modprobe -r WS_xinchDSI_Screen
sudo modprobe WS_xinchDSI_Screen
```

puede producir:

```text
Internal error: Oops
```

Por tanto, una recarga dinámica debe considerarse una acción de alto riesgo para este sistema.

---

## 19.4. Device Tree / overlays

La configuración depende de:

```ini
dtoverlay=vc4-kms-v3d
dtoverlay=WS_xinchDSI_Screen,SCREEN_type=8,I2C_bus=10
dtoverlay=WS_xinchDSI_Touch,I2C_bus=10,invertedx,swappedxy
```

Cambios en estos overlays pueden afectar directamente:

```text
DSI
VC4
DRM
Touch
Framebuffer
```

No modificar parámetros sin comparar primero con una unidad funcional.

---

## 19.5. Kernel

En el caso analizado, ambos equipos funcional y problemático utilizaron:

```text
6.1.21-v8+ #1642
```

Por tanto, el kernel no explicó la diferencia observada entre ambos equipos.

No obstante, si en futuras unidades existen kernels diferentes, la compatibilidad entre kernel y módulos Waveshare debe comprobarse antes de reemplazar archivos.

---

# 20. Procedimientos que NO deben realizarse sin justificación

Evitar como primera acción:

```bash
sudo modprobe -r WS_xinchDSI_Screen
```

Evitar forzar:

```bash
sudo rmmod -f ...
```

Evitar modificar inmediatamente:

```text
/boot/config.txt
/etc/lightdm/lightdm.conf
LXDE autostart
startweb.sh
```

Evitar reinstalar aleatoriamente:

```text
kernel
drivers Waveshare
módulos DRM
```

Evitar atribuir el problema a Chromium cuando:

```text
card1
card1-DSI-1
fb0
```

todavía no existen.

---

# 21. Procedimiento de recuperación

Si se presenta una pantalla negra y se confirma:

```text
card0
renderD128
```

pero no:

```text
card1
card1-DSI-1
```

primero realizar una comprobación no destructiva:

```bash
dmesg | grep -Ei "vc4|v3d|WS_xinchDSI|dsi|drm"
```

Si el sistema se encuentra en un estado anómalo después de una intervención con módulos y no existe acceso gráfico, un **reinicio completo** puede ser suficiente para recuperar la inicialización.

Si el reinicio normal no recupera el sistema y no existe acceso SSH, puede requerirse un apagado/encendido físico controlado.

Después del arranque:

```bash
ls -l /sys/class/drm/
```

y:

```bash
ls -l /sys/devices/platform/gpu/drm/
```

confirmar nuevamente `card1` y `card1-DSI-1`.

---

# 22. Comparación contra una unidad funcional

Cuando exista una unidad funcional disponible, utilizarla como referencia.

Comparar, como mínimo:

### Kernel

```bash
uname -a
```

### Configuración

```bash
grep -vE '^\s*#|^\s*$' /boot/config.txt
```

### DRM

```bash
ls -l /sys/class/drm/
```

### VC4

```bash
ls -l /sys/devices/platform/gpu/
ls -l /sys/devices/platform/gpu/drm/
```

### Módulos

```bash
lsmod | grep -E 'WS_|vc4|v3d|drm'
```

### Archivos Waveshare

```bash
modinfo -n WS_xinchDSI_Screen
sha256sum "$(modinfo -n WS_xinchDSI_Screen)"
```

```bash
modinfo -n WS_xinchDSI_Screen_Interface
sha256sum "$(modinfo -n WS_xinchDSI_Screen_Interface)"
```

La comparación mediante SHA-256 permite determinar si los módulos realmente son idénticos.

---

# 23. Checklist rápido de diagnóstico

## Pantalla negra

- [ ] SSH disponible.
- [ ] `card0` presente.
- [ ] `card1` presente.
- [ ] `card1-DSI-1` presente.
- [ ] `/sys/devices/platform/gpu/drm/card1` presente.
- [ ] `vc4-drm` asociado a `gpu`.
- [ ] `fb0` presente.
- [ ] `vc4-drm gpu: bound ... dsi` presente en `dmesg`.
- [ ] `[drm] Initialized vc4` presente.
- [ ] Xorg sin `no screens found`.
- [ ] LightDM activo.
- [ ] Chromium/TELSY posteriormente.

### Interpretación rápida

| Resultado | Interpretación |
|---|---|
| Solo `card0` | DRM VC4 no completó inicialización |
| `card1` sin `card1-DSI-1` | VC4 existe, revisar DSI |
| `card1` + `card1-DSI-1` | Subsistema gráfico probablemente correcto |
| Sin `fb0` | Revisar framebuffer/DRM |
| Xorg `no screens found` + sin `card1` | Xorg probablemente es consecuencia, no causa |
| `vc4-drm ... bound ... dsi` | DSI enlazado correctamente |
| `Initialized vc4` | VC4 DRM inicializado |
| `Oops` al cargar Waveshare | No continuar recargando el módulo |

---

# 24. Estado final del caso analizado

Después de un apagado completo y nuevo arranque, el sistema recuperó correctamente la pantalla.

El estado final presentó:

```text
card0
card1
card1-DSI-1
card1-HDMI-A-1
card1-HDMI-A-2
card1-Writeback-1
renderD128
```

y:

```text
vc4-drm gpu: bound fe700000.dsi
```

seguido de:

```text
[drm] Initialized vc4 0.0.0 20140616 for gpu on minor 1
```

y:

```text
vc4-drm gpu: [drm] fb0: vc4drmfb frame buffer device
```

Por lo tanto, **la configuración instalada es capaz de inicializar correctamente la pantalla**.

No se identificó evidencia suficiente para justificar el reemplazo del kernel, los módulos Waveshare, los overlays o la configuración de TELSY.

La causa exacta del primer estado defectuoso permanece **no determinada**. Durante el diagnóstico se realizó una descarga/carga manual del módulo Waveshare que produjo un kernel Oops, por lo que esa intervención constituye un factor potencial del estado inconsistente observado posteriormente.

---

# 25. Recomendación para producción

Mientras el sistema se encuentre funcionando correctamente:

1. No modificar los módulos Waveshare.
2. No cambiar el kernel.
3. No modificar los overlays DSI.
4. No descargar manualmente `WS_xinchDSI_Screen`.
5. Mantener una copia de `/boot/config.txt`.
6. Mantener una copia/hash de los módulos `.ko`.
7. Documentar la versión exacta del kernel.
8. Conservar una unidad funcional como referencia.
9. Ante una pantalla negra, comprobar primero DRM/VC4/DSI.
10. Solo después revisar Xorg, LightDM y Chromium.
11. Registrar cualquier `Oops`, `panic` o error de `vc4-drm`.
12. Evitar modificaciones permanentes hasta tener evidencia reproducible.

---

# 26. Comandos de referencia

## Kernel

```bash
uname -a
```

## Configuración

```bash
grep -vE '^\s*#|^\s*$' /boot/config.txt
```

## DRM

```bash
ls -l /sys/class/drm/
```

## VC4

```bash
ls -l /sys/devices/platform/gpu/
ls -l /sys/devices/platform/gpu/drm/
```

## Framebuffer

```bash
cat /proc/fb
```

## Kernel log gráfico

```bash
dmesg | grep -Ei "vc4|v3d|WS_xinchDSI|dsi|drm"
```

## Módulos

```bash
lsmod | grep -E 'WS_|vc4|v3d|drm'
```

## Xorg

```bash
grep -Ei "\(EE\)|no screens|No devices|fb0" /var/log/lightdm/x-0.log
```

## LightDM

```bash
systemctl status lightdm
```

```bash
journalctl -u lightdm -b
```

## Comparación de módulos

```bash
modinfo -n WS_xinchDSI_Screen
sha256sum "$(modinfo -n WS_xinchDSI_Screen)"
```

```bash
modinfo -n WS_xinchDSI_Screen_Interface
sha256sum "$(modinfo -n WS_xinchDSI_Screen_Interface)"
```

---

# 27. Conclusión

La pantalla Waveshare del sistema TELSY-QM depende de una cadena de inicialización gráfica compuesta por Device Tree, overlays, módulos Waveshare, DSI, VC4 DRM, framebuffer, Xorg y finalmente el entorno gráfico.

El indicador más útil para determinar rápidamente el estado del sistema es la existencia de:

```text
card1
card1-DSI-1
```

en:

```text
/sys/class/drm/
```

Cuando estos dispositivos no existen, el problema debe investigarse inicialmente en **VC4/DRM/DSI**, antes de modificar Xorg, LightDM, Chromium o TELSY.

El caso analizado demostró que el mismo kernel, configuración y módulos pueden producir un arranque gráfico correcto después de un apagado completo. Por tanto, no existe evidencia suficiente para considerar incompatibles los componentes actualmente instalados.

La descarga y recarga manual del módulo `WS_xinchDSI_Screen` produjo un kernel Oops durante la investigación. En consecuencia, dicha operación no debe utilizarse como procedimiento rutinario de recuperación.

El procedimiento recomendado para futuras incidencias es:

```text
Pantalla negra
      |
      v
¿SSH disponible?
      |
      v
¿Existe card1?
      |
      +---- NO ----> revisar VC4/DRM/DSI
      |
      +---- SÍ ----> ¿existe card1-DSI-1?
                           |
                           +---- NO ----> revisar DSI
                           |
                           +---- SÍ ----> revisar framebuffer/Xorg
                                                |
                                                v
                                           LightDM/LXDE
                                                |
                                                v
                                           Chromium/TELSY
```

Este flujo reduce intervenciones innecesarias y permite localizar el nivel real de la falla antes de modificar componentes del sistema.
