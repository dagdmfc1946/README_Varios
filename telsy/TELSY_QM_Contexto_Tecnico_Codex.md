# TELSY / TELSY-QM — Contexto técnico integral para desarrollo, diagnóstico y mantenimiento

> **Propósito:** este documento consolida el conocimiento técnico disponible sobre el proyecto TELSY/TELSY-QM para utilizarlo como contexto de trabajo con Codex, especialmente para análisis de código legado, mantenimiento, depuración, configuración de Raspberry Pi CM4, LTE, pantalla, servicios y arranque automático.
>
> **Regla importante para Codex:** distinguir siempre entre **HECHO / VALIDADO**, **INFERENCIA**, **RECOMENDACIÓN** y **REQUIERE VALIDACIÓN**. No asumir que una hipótesis es una solución aplicada.

---

## 1. Identificación del proyecto

**TELSY-QM** es un sistema embebido de telemonitorización desarrollado sobre Raspberry Pi. El sistema integra:

- Raspberry Pi 4 / Raspberry Pi Compute Module 4 (CM4).
- Django.
- Django Channels.
- Daphne.
- Frontend HTML/CSS/JavaScript.
- Python.
- Comunicación serial.
- I2C.
- GPIO.
- SysV IPC.
- Driver del módulo multiparamétrico PM6750.
- Módulo LTE Huawei ME906s.
- ModemManager.
- NetworkManager.
- Pantalla DSI.
- Entorno gráfico LXDE.
- Navegador para la WebApp.
- Scripts Bash de instalación, arranque, configuración y mantenimiento.

El proyecto se utiliza dentro del ecosistema TELSY, con componentes referidos como:

- TELSY-WEB.
- TELSY-HOGAR.
- TELSY-APP.
- TELSY-QM.
- Placa principal.
- Tarjeta multiparamétrica PM6750.

---

# 2. Hardware conocido

## 2.1 Raspberry Pi

Se ha trabajado con:

- Raspberry Pi 4.
- Raspberry Pi CM4.

Configuración de producción piloto validada:

- **RAM:** 2 GB.
- **Almacenamiento:** 8 GB eMMC.

También existieron prototipos con mayor capacidad de eMMC.

El proyecto TELSY fue validado funcionalmente sobre CM4 de 2 GB RAM / 8 GB eMMC.

### Consideración

Los problemas observados no deben atribuirse automáticamente a falta de RAM. En los problemas de pantalla/autostart, se debe investigar primero:

- secuencia de arranque;
- procesos bloqueantes;
- servicios;
- LXDE;
- DRM/KMS;
- módulos del kernel;
- drivers;
- scripts ejecutados durante el arranque.

---

# 3. Arquitectura software conocida

## 3.1 Estructura principal

Ruta principal del proyecto:

```text
/home/pi/telsy-monitor
```

Componentes/scripts relevantes:

```text
/home/pi/telsy-monitor/
├── telsy/
├── telsy_venv/
├── startserver.sh
├── startweb.sh
├── run_driver.sh
├── delete_VAR.sh
├── configAPN-LTE.sh
├── autoLTE.sh
└── install_dependencies_telsyAPP.sh
```

La estructura exacta puede variar entre versiones. Antes de modificar rutas, verificar siempre:

```bash
cd /home/pi/telsy-monitor
ls -lah
find . -maxdepth 2 -type f | sort
```

---

# 4. Entorno Python

El entorno virtual utilizado está en:

```text
/home/pi/telsy-monitor/telsy_venv
```

No debe asumirse que el entorno virtual está dentro de:

```text
/home/pi/telsy-monitor/telsy/
```

Activación:

```bash
source /home/pi/telsy-monitor/telsy_venv/bin/activate
```

Los scripts de arranque deben utilizar la ruta correcta del entorno.

---

# 5. Driver PM6750

El driver asociado al módulo multiparamétrico PM6750 es:

```text
vspm-pm6750
```

El script `run_driver.sh` ha utilizado:

```bash
./telsy/vspm-pm6750 --se0_path=/dev/ttyAMA1
```

Puerto serial conocido:

```text
/dev/ttyAMA1
```

## Diagnóstico básico

Verificar:

```bash
ls -l /dev/ttyAMA*
ls -l /dev/serial*
```

Procesos:

```bash
ps aux | grep vspm-pm6750
```

Permisos:

```bash
ls -l /home/pi/telsy-monitor/telsy/vspm-pm6750
```

Si aparece:

```text
Permission denied
```

verificar permisos de ejecución:

```bash
chmod +x /home/pi/telsy-monitor/telsy/vspm-pm6750
```

También verificar propietario:

```bash
ls -l /home/pi/telsy-monitor/telsy/vspm-pm6750
```

---

# 6. Scripts principales de TELSY

## 6.1 startserver.sh

Responsabilidad:

- activar el entorno virtual;
- iniciar el servidor Django/TELSY.

La ruta esperada es:

```text
/home/pi/telsy-monitor/startserver.sh
```

Ejecutar manualmente:

```bash
cd /home/pi/telsy-monitor
./startserver.sh
```

---

## 6.2 run_driver.sh

Responsabilidad:

- iniciar el driver PM6750;
- comunicarse mediante el puerto serial correspondiente.

Ejecutar:

```bash
cd /home/pi/telsy-monitor
./run_driver.sh
```

---

## 6.3 startweb.sh

Responsabilidad:

- iniciar el acceso a la WebApp/interfaz web;
- utilizar el navegador configurado en el sistema.

En el proyecto se decidió utilizar Chrome/Edge y no depender de Chromium/kiosco.

Ejecutar:

```bash
cd /home/pi/telsy-monitor
./startweb.sh
```

---

# 7. Autostart LXDE

Se utilizó:

```text
/etc/xdg/lxsession/LXDE-pi/autostart
```

Para iniciar componentes de TELSY.

Ejemplo de rutas utilizadas:

```text
@sh /home/pi/telsy-monitor/startserver.sh
@sh /home/pi/telsy-monitor/startweb.sh
@sh /home/pi/telsy-monitor/run_driver.sh
```

## Problema importante

Se observó que determinados esquemas de autostart podían provocar:

- pantalla negra;
- bloqueo aparente;
- problemas de secuencia;
- interferencia con el entorno gráfico.

### Diagnóstico recomendado

No asumir que el problema es de hardware de pantalla.

Primero verificar:

```bash
systemctl --failed
ps aux
free -h
uptime
```

También comprobar:

```bash
journalctl -b -p err
journalctl -b
```

Y verificar el autostart:

```bash
cat /etc/xdg/lxsession/LXDE-pi/autostart
```

### Recomendación arquitectónica

Los procesos de backend/driver deben preferiblemente gestionarse mediante **systemd**, mientras que el navegador/interfaz gráfica puede mantenerse asociado al entorno LXDE.

Esto permite:

- controlar dependencias;
- controlar orden de arranque;
- reiniciar servicios;
- consultar logs;
- evitar procesos largos bloqueando el autostart gráfico.

---

# 8. Servicios systemd

Se han utilizado servicios systemd para componentes de TELSY.

Un servicio relevante es:

```text
delete_VAR.service
```

Su objetivo es ejecutar la limpieza de determinados logs.

Ver estado:

```bash
systemctl status delete_VAR.service
```

Ver configuración:

```bash
systemctl cat delete_VAR.service
```

Ver servicios TELSY:

```bash
systemctl list-units --type=service | grep -i telsy
```

o:

```bash
systemctl list-unit-files | grep -Ei 'telsy|delete|lte|modem'
```

---

# 9. delete_VAR.sh

Ruta:

```text
/home/pi/telsy-monitor/delete_VAR.sh
```

El script se creó para liberar espacio antes de iniciar TELSY, especialmente por crecimiento de logs.

Comandos utilizados:

```bash
sudo truncate -s 0 /var/log/syslog
sudo truncate -s 0 /var/log/daemon.log
```

También se utilizó:

```bash
sudo journalctl --vacuum-size=100M
```

Ejecución manual:

```bash
cd /home/pi/telsy-monitor
./delete_VAR.sh
```

## Importante

El servicio:

```text
delete_VAR.service
```

es preferible a ejecutar la limpieza desde el autostart gráfico cuando la limpieza debe ocurrir antes de los servicios TELSY.

La dependencia debe establecerse explícitamente mediante systemd cuando sea necesario:

```text
Requires=
After=
Before=
```

---

# 10. Problema de almacenamiento lleno

Uno de los problemas encontrados fue:

```text
/dev/root
6.9G usados de 6.9G
100%
```

Esto provocó problemas generales del sistema.

## Diagnóstico

Comando inicial:

```bash
df -h /
```

Después:

```bash
sudo du -xh --max-depth=1 /
```

Luego:

```bash
sudo du -xh --max-depth=1 /var
```

Y:

```bash
sudo du -xh --max-depth=1 /var/log
```

Se encontró aproximadamente:

```text
/var/log ≈ 3.3G
```

Además, `/usr` también tenía un tamaño considerable.

El journal llegó aproximadamente a:

```text
730 MB
```

## Limpieza aplicada

```bash
sudo truncate -s 0 /var/log/syslog
sudo truncate -s 0 /var/log/daemon.log
sudo journalctl --vacuum-size=100M
```

En una ejecución concreta, `journalctl` indicó:

```text
Freed 0B
```

Esto significa que no había journals archivados que pudieran eliminarse con esa operación concreta; no significa que `/var/log` estuviera vacío.

## Diagnóstico posterior

Siempre comprobar:

```bash
df -h
sudo du -xh --max-depth=1 /var
sudo du -xh --max-depth=1 /var/log
```

También:

```bash
journalctl --disk-usage
```

---

# 11. Problema de resolución del hostname

Durante las operaciones de mantenimiento apareció:

```text
sudo: unable to resolve host TVSM01:
Fallo temporal en la resolución del nombre
```

Esto indica una posible inconsistencia entre:

```text
/etc/hostname
```

y:

```text
/etc/hosts
```

Diagnóstico:

```bash
cat /etc/hostname
cat /etc/hosts
hostname
hostnamectl
```

El hostname observado fue:

```text
TVSM01
```

La configuración debe mantener coherencia entre `/etc/hostname` y la entrada local correspondiente de `/etc/hosts`.

---

# 12. Problema de GPS

El proyecto utiliza un script relacionado con el GPS:

```text
GPS_ME906s.py
```

Este script interactúa con ModemManager.

En versiones anteriores se observó que el script podía:

- detener ModemManager;
- ejecutar determinadas operaciones;
- volver a iniciar ModemManager.

Esto es importante porque el módem LTE también depende de ModemManager.

## Riesgo

Si el GPS y LTE utilizan el mismo módem:

```text
GPS_ME906s.py
        ↓
ModemManager
        ↓
Huawei ME906s
        ↓
LTE
```

entonces detener ModemManager puede interrumpir la conexión LTE.

### Regla para futuras modificaciones

Antes de modificar `GPS_ME906s.py`, verificar:

```bash
systemctl status ModemManager
```

y:

```bash
mmcli --list-modems
```

No detener ModemManager arbitrariamente durante una sesión LTE activa.

---

# 13. Módem LTE Huawei ME906s

Hardware identificado:

```text
HUAWEI Mobile
```

Modelo:

```text
Huawei ME906s
```

Firmware observado:

```text
11.617.04.00.00
```

Hardware:

```text
ML1ME906SM
```

Tecnologías:

```text
gsm-umts
lte
```

Puerto principal observado:

```text
cdc-wdm0
```

---

# 14. ModemManager

Estado esperado:

```bash
systemctl status ModemManager
```

En un momento del diagnóstico se encontró:

```text
couldn't find the ModemManager process in the bus
```

Posteriormente ModemManager quedó funcionando correctamente.

Comando:

```bash
sudo systemctl status ModemManager
```

Reiniciar:

```bash
sudo systemctl restart ModemManager
```

Ver módems:

```bash
mmcli --list-modems
```

Consultar un módem:

```bash
mmcli --modem=0
```

El número puede cambiar.

---

# 15. Numeración del módem

Se observó que el módem podía aparecer como:

```text
modem=0
```

y posteriormente:

```text
modem=3
```

o:

```text
modem=4
```

No debe asumirse que el número de módem es permanente.

La numeración puede variar por:

- reinicio;
- enumeración USB;
- reconexión;
- inserción/remoción de SIM;
- reinicio de ModemManager;
- cambios en el orden de detección.

## Regla crítica

No construir lógica de producción basada únicamente en:

```text
--modem=0
```

Es preferible detectar dinámicamente el módem disponible.

---

# 16. Diagnóstico básico LTE

Lista de módems:

```bash
mmcli --list-modems
```

Información detallada:

```bash
mmcli --modem=0
```

Estado de ModemManager:

```bash
systemctl status ModemManager
```

Estado NetworkManager:

```bash
systemctl status NetworkManager
```

Conexiones:

```bash
nmcli connection show
```

Dispositivos:

```bash
nmcli device status
```

Consultar el módem mediante ModemManager:

```bash
mmcli --modem=0
```

Conexión directa de prueba:

```bash
sudo mmcli --modem=0 --simple-connect="apn=internet.comcel.com.co"
```

Deshabilitar:

```bash
sudo mmcli --modem=0 --disable
```

Habilitar mediante NetworkManager:

```bash
sudo nmcli --modem=0 --enable
```

> El identificador del módem debe determinarse dinámicamente antes de ejecutar comandos de producción.

---

# 17. Perfiles LTE mediante NetworkManager

Se utilizó NetworkManager para crear perfiles GSM.

Ejemplo de perfiles observados:

```text
lte-CLARO
lte-ETB
lte-EXITO
lte-FLASH
...
```

También existió:

```text
LTE-TELSY
```

asociado inicialmente con WOM.

Posteriormente se decidió eliminar los perfiles `LTE-TELSY` y utilizar:

```text
lte-WOM
```

---

# 18. Listar todas las conexiones

Para listar todos los perfiles configurados:

```bash
nmcli connection show
```

Este comando muestra:

- WiFi;
- Ethernet;
- GSM/LTE;
- otros perfiles de NetworkManager.

Para dispositivos:

```bash
nmcli device status
```

---

# 19. Problema de cambio de SIM

El requisito funcional principal es:

> El equipo debe poder cambiar de SIM y conectarse automáticamente al operador correspondiente sin que el usuario tenga que seleccionar manualmente un perfil.

Inicialmente se probó:

- Movistar → conexión exitosa.
- Claro → conexión posterior.
- Movistar nuevamente → conexión exitosa.

También se probó WOM.

## Requisito final

La Raspberry debe:

1. detectar el módem;
2. identificar la SIM/operador disponible;
3. determinar el perfil/APN correspondiente;
4. conectar automáticamente;
5. funcionar después de retirar/insertar una SIM diferente;
6. no depender del número fijo de módem;
7. evitar perfiles duplicados o ambiguos.

---

# 20. Configuración de APN

Se planteó separar la lógica en dos scripts:

```text
configAPN-LTE.sh
```

y:

```text
autoLTE.sh
```

## configAPN-LTE.sh

Responsabilidad:

- crear perfiles;
- actualizar perfiles;
- garantizar que existan todos los APN necesarios;
- no requerir selección interactiva.

La idea es ejecutar este script durante la preparación del equipo para dejar configurados todos los perfiles.

No debe instalar dependencias si el dispositivo ya dispone de los paquetes necesarios.

---

# 21. autoLTE.sh

Responsabilidad:

- detectar dinámicamente el módem;
- detectar la SIM/operador;
- seleccionar el perfil correspondiente;
- activar la conexión;
- validar conectividad.

Debe evitar:

```bash
MODEM=0
```

como valor fijo.

La detección debe basarse en el estado real del sistema.

---

# 22. Autoconexión LTE recomendada

Flujo conceptual:

```text
BOOT
 │
 ├── ModemManager
 │
 ├── NetworkManager
 │
 ├── Detectar módem
 │
 ├── Detectar SIM
 │
 ├── Identificar operador
 │
 ├── Seleccionar APN
 │
 ├── Activar perfil
 │
 └── Validar conectividad
```

El script no debe asumir que la SIM está disponible inmediatamente después del arranque.

Por tanto, puede requerirse:

- esperar a que aparezca el módem;
- esperar a que la SIM esté disponible;
- consultar estado varias veces;
- aplicar timeout;
- generar logs.

---

# 23. Validación de conexión LTE

Se solicitó una validación sencilla al final del proceso.

Ejemplo:

```bash
ping -c 3 8.8.8.8
```

También puede utilizarse:

```bash
ping -c 3 1.1.1.1
```

La prueba debe interpretarse como validación de conectividad IP, no como prueba completa del estado del operador.

## Diagnóstico por capas

### Capa 1 — Hardware

```bash
lsusb
```

### Capa 2 — ModemManager

```bash
mmcli --list-modems
mmcli --modem=<ID>
```

### Capa 3 — SIM

Consultar el estado de SIM mediante `mmcli`.

### Capa 4 — Registro de red

Consultar estado del módem.

### Capa 5 — NetworkManager

```bash
nmcli device status
nmcli connection show
```

### Capa 6 — IP

```bash
ip addr
ip route
```

### Capa 7 — Conectividad

```bash
ping -c 3 8.8.8.8
```

---

# 24. Problema: módem en estado "connecting"

Se observó un caso donde:

```text
mmcli
```

mostraba el módem en estado:

```text
connecting
```

Una hipótesis inicial fue falta de señal.

Esto puede deberse a múltiples causas:

- señal insuficiente;
- SIM no registrada;
- APN incorrecto;
- operador no disponible;
- autenticación;
- módem ocupado;
- estado anterior de ModemManager;
- perfil NetworkManager incorrecto;
- interfaz WWAN no activa.

## Diagnóstico correcto

No concluir inmediatamente "no hay señal".

Consultar:

```bash
mmcli --modem=<ID>
```

y:

```bash
nmcli device status
```

También:

```bash
nmcli connection show
```

y:

```bash
ip addr
ip route
```

---

# 25. WWAN

Se trabajó con la interfaz WWAN mediante NetworkManager.

Ver:

```bash
nmcli device status
```

Buscar:

```text
wwan0
```

o el dispositivo equivalente.

La conexión debe estar asociada al perfil GSM correspondiente.

---

# 26. Perfiles y autoconnect

Para consultar un perfil:

```bash
nmcli connection show "lte-CLARO"
```

Debe revisarse:

```text
connection.autoconnect
```

El objetivo es evitar que múltiples perfiles compitan por el mismo módem.

Por eso, para cambio automático de SIM, es preferible que `autoLTE.sh` controle explícitamente el perfil seleccionado según la SIM.

---

# 27. Pantalla DSI

Uno de los problemas más complejos fue una pantalla negra después del arranque.

La pantalla funciona mediante:

- DSI;
- DRM/KMS;
- VC4;
- módulo específico:

```text
WS_xinchDSI_Screen
```

y:

```text
WS_xinchDSI_Screen_Interface
```

---

# 28. Configuración de pantalla conocida

En el equipo funcional se encontró una configuración de `/boot/config.txt` que incluía:

```text
dtparam=i2c_arm=on
dtparam=audio=on
camera_auto_detect=1
display_auto_detect=1
dtoverlay=vc4-kms-v3d
max_framebuffers=2
arm_64bit=1
disable_overscan=1

[cm4]
otg_mode=1

dtoverlay=wm8904
dtoverlay=i2c-rtc,ds1307
ignore_lcd=1
dtparam=i2c_vc=on

dtoverlay=WS_xinchDSI_Screen
SCREEN_type=8
I2C_bus=10
```

No modificar estos parámetros sin comparar primero con un equipo funcional.

---

# 29. Kernel conocido del equipo funcional

Se observó:

```text
Linux TVSM01 6.1.21-v8+ #1642 SMP PREEMPT
Mon Apr 3 17:24:16 BST 2023
aarch64 GNU/Linux
```

Comando:

```bash
uname -a
```

---

# 30. Diagnóstico DRM

Comando:

```bash
ls -l /sys/class/drm/
```

En un estado observado aparecieron:

```text
card0
card1
card1-DSI-1
card1-HDMI-A-1
card1-HDMI-A-2
```

También se observó:

```bash
readlink -f /sys/class/drm/card1/device/driver
```

resultado:

```text
/sys/bus/platform/drivers/vc4-drm
```

Esto confirma asociación con:

```text
vc4-drm
```

---

# 31. Diagnóstico GPU/VC4

Se observó:

```bash
ls -l /sys/devices/platform/gpu
```

con:

```text
driver -> ../../../bus/platform/drivers/vc4-drm
```

y otros archivos del dispositivo.

También:

```bash
dmesg | grep -Ei 'vc4|drm|dsi|screen'
```

---

# 32. Módulo WS_xinchDSI_Screen

Se verificó:

```bash
modinfo -n WS_xinchDSI_Screen
```

Ruta:

```text
/lib/modules/6.1.21-v8+/WS_xinchDSI_Screen.ko
```

Se realizó una carga manual:

```bash
modprobe WS_xinchDSI_Screen
```

y:

```bash
modprobe WS_xinchDSI_Screen_Interface
```

---

# 33. Fallo grave al cargar módulo de pantalla

Durante la investigación, la carga manual del módulo provocó:

```text
Internal error: Oops: 0000000096000005 [#1]
PREEMPT SMP
```

y:

```text
Segmentation fault
```

En `dmesg` aparecieron los módulos:

```text
WS_xinchDSI_Screen(PO+)
WS_xinchDSI_Screen_Interface(O)
vc4-drm
```

## Interpretación

Esto es evidencia de un problema a nivel kernel/módulo, no simplemente un problema de configuración de escritorio.

Posibles causas:

- incompatibilidad entre módulo y kernel;
- módulo compilado para otra versión;
- incompatibilidad con VC4/KMS;
- orden de carga;
- incompatibilidad del driver DSI;
- corrupción o problema del módulo;
- conflicto entre módulos.

No asumir que el módulo es seguro de cargar manualmente.

---

# 34. /usr/share/dispsetup.sh

Se encontró:

```text
/usr/share/dispsetup.sh
```

Era ejecutable y su contenido esencial era:

```bash
#!/bin/sh
exit 0
```

No se encontraron referencias relevantes a:

```text
dsi
fb0
fbdev
drm
vc4
xrandr
```

Esto indica que este script no era responsable de la configuración activa de pantalla en ese estado.

---

# 35. Diagnóstico de framebuffer

Se revisaron:

```bash
ls -l /dev/fb*
```

y:

```bash
ls -l /dev/dri/
```

También:

```bash
ls -l /sys/class/drm/
```

La ausencia/presencia de estos nodos es útil para separar:

- problema del kernel;
- problema DRM;
- problema framebuffer;
- problema LXDE;
- problema del navegador.

---

# 36. Problema de pantalla negra: metodología

Cuando la pantalla esté negra, seguir este orden.

## Paso 1 — Comprobar que el sistema está vivo

Desde SSH:

```bash
uptime
```

```bash
free -h
```

```bash
top
```

```bash
systemctl --failed
```

## Paso 2 — Verificar DRM

```bash
ls -l /sys/class/drm/
```

## Paso 3 — Verificar driver VC4

```bash
readlink -f /sys/class/drm/card1/device/driver
```

## Paso 4 — Verificar dispositivos gráficos

```bash
ls -l /dev/dri/
ls -l /dev/fb*
```

## Paso 5 — Ver logs del kernel

```bash
dmesg | grep -Ei 'drm|vc4|dsi|xinch|screen'
```

## Paso 6 — Revisar errores de boot

```bash
journalctl -b -p err
```

## Paso 7 — Revisar autostart

```bash
cat /etc/xdg/lxsession/LXDE-pi/autostart
```

## Paso 8 — Comparar contra equipo funcional

Comparar:

```bash
uname -a
```

```bash
cat /boot/config.txt
```

```bash
lsmod
```

```bash
ls -l /sys/class/drm/
```

```bash
dmesg | grep -Ei 'drm|vc4|dsi|xinch'
```

---

# 37. Diferenciar pantalla negra de sistema congelado

No asumir que:

```text
pantalla negra = sistema apagado
```

Si SSH responde:

```bash
ssh pi@<IP>
```

el sistema está operativo y el problema probablemente está en:

- entorno gráfico;
- display server;
- DRM;
- driver;
- navegador;
- autostart.

Si SSH no responde, revisar además:

- CPU;
- memoria;
- almacenamiento;
- kernel panic;
- bloqueo de servicios;
- red.

---

# 38. Recuperación después de congelamiento

En una ocasión el equipo pareció quedar congelado y se recuperó después de:

- apagado forzado;
- reinicio.

Posteriormente volvió la imagen y el acceso SSH.

Esto no debe considerarse una solución definitiva.

Ante un congelamiento:

1. revisar logs del boot anterior;
2. revisar `dmesg`;
3. revisar `journalctl`;
4. comprobar almacenamiento;
5. comprobar módulos;
6. revisar autostart;
7. revisar servicios systemd.

Comando útil:

```bash
journalctl -b -1
```

para revisar el boot anterior, si el journal está disponible.

---

# 39. Problema de autostart y pantalla negra

Una hipótesis/observación importante fue que el autostart podía estar relacionado con la pantalla negra.

Por eso se recomienda aislar componentes.

## Prueba

Deshabilitar temporalmente:

```text
startserver.sh
startweb.sh
run_driver.sh
```

del autostart.

Reiniciar y comprobar si la pantalla funciona.

Luego activar un componente por vez.

Esto permite identificar cuál proceso modifica el comportamiento.

---

# 40. Orden recomendado de arranque

Arquitectura recomendada:

```text
BOOT
 │
 ├── systemd
 │    ├── ModemManager
 │    ├── NetworkManager
 │    ├── delete_VAR.service
 │    ├── TELSY backend
 │    └── TELSY driver
 │
 └── LXDE
      └── navegador / WebApp
```

Evitar que procesos de larga duración sean ejecutados secuencialmente desde un único script de autostart.

---

# 41. Problemas de rutas relativas

Los scripts TELSY contienen o pueden contener rutas relativas.

Por eso es recomendable:

```bash
cd /home/pi/telsy-monitor
./startserver.sh
```

en lugar de ejecutar el script desde un directorio arbitrario.

Para scripts ejecutados por systemd, utilizar rutas absolutas siempre que sea posible.

Ejemplo:

```text
WorkingDirectory=/home/pi/telsy-monitor
```

---

# 42. Diagnóstico general del sistema

## Información del sistema

```bash
uname -a
```

```bash
hostname
```

```bash
hostnamectl
```

```bash
lsb_release -a
```

## CPU/RAM

```bash
free -h
```

```bash
lscpu
```

```bash
uptime
```

## Almacenamiento

```bash
df -h
```

```bash
df -i
```

```bash
sudo du -xh --max-depth=1 /
```

## Servicios

```bash
systemctl --failed
```

```bash
systemctl list-units --type=service
```

## Logs

```bash
journalctl -b
```

```bash
journalctl -b -p err
```

```bash
dmesg -T
```

---

# 43. Diagnóstico de procesos TELSY

```bash
ps aux | grep -Ei 'python|django|daphne|telsy|vspm'
```

También:

```bash
pgrep -af vspm-pm6750
```

y:

```bash
pgrep -af daphne
```

---

# 44. Diagnóstico de puertos

Si TELSY utiliza un puerto TCP:

```bash
sudo ss -lntp
```

Buscar específicamente:

```bash
sudo ss -lntp | grep -E '8000|8080|80'
```

El puerto real debe verificarse en la configuración actual.

---

# 45. Diagnóstico de red

Interfaces:

```bash
ip addr
```

Rutas:

```bash
ip route
```

DNS:

```bash
resolvectl status
```

Conectividad local:

```bash
ping -c 3 <gateway>
```

Conectividad IP externa:

```bash
ping -c 3 8.8.8.8
```

DNS:

```bash
ping -c 3 google.com
```

Interpretación:

- IP externa funciona + dominio falla → posible problema DNS.
- Gateway funciona + IP externa falla → posible problema de ruta/operador.
- WWAN sin IP → problema de conexión LTE.
- Módem no aparece → problema anterior a NetworkManager.

---

# 46. Diagnóstico de NetworkManager

```bash
systemctl status NetworkManager
```

```bash
nmcli general status
```

```bash
nmcli device status
```

```bash
nmcli connection show
```

Para una conexión concreta:

```bash
nmcli connection show "lte-CLARO"
```

Logs:

```bash
journalctl -u NetworkManager -b
```

---

# 47. Diagnóstico de ModemManager

```bash
systemctl status ModemManager
```

```bash
mmcli --list-modems
```

```bash
mmcli --modem=<ID>
```

Logs:

```bash
journalctl -u ModemManager -b
```

También:

```bash
dmesg -T | grep -Ei 'usb|cdc|qmi|mbim|wwan|modem'
```

---

# 48. USB y módem

Diagnóstico:

```bash
lsusb
```

```bash
lsusb -t
```

Buscar:

```text
Huawei
```

y dispositivos:

```text
cdc-wdm
```

Comprobar:

```bash
ls -l /dev/cdc-wdm*
```

El dispositivo observado fue:

```text
/dev/cdc-wdm0
```

---

# 49. Dependencias LTE

La arquitectura utiliza componentes del stack Linux para módem:

- ModemManager.
- NetworkManager.
- libmbim/libqmi, según el modo utilizado por el módem.
- `mmcli`.
- `nmcli`.

El equipo objetivo ya disponía de las dependencias necesarias, por lo que `configAPN-LTE.sh` no debe asumir que necesita instalar todo nuevamente.

---

# 50. Instalación inicial

Existió:

```text
install_dependencies_telsyAPP.sh
```

Este script fue utilizado como referencia durante la preparación del sistema.

Sin embargo, para el proceso de configuración APN no se requiere reinstalar las dependencias LTE si ya están presentes.

---

# 51. Filosofía para modificar scripts TELSY

Antes de modificar cualquier script:

1. leer el archivo completo;
2. identificar dependencias;
3. identificar rutas;
4. identificar servicios externos;
5. identificar archivos modificados;
6. identificar efectos secundarios;
7. preservar comportamiento existente;
8. hacer cambios mínimos;
9. probar manualmente;
10. después automatizar.

No reescribir un script completo si el requerimiento puede resolverse con un cambio localizado.

---

# 52. Regla para cambios en servicios

Antes de modificar un servicio:

```bash
systemctl cat <servicio>
```

Después:

```bash
sudo systemctl daemon-reload
```

Si corresponde:

```bash
sudo systemctl restart <servicio>
```

Verificar:

```bash
systemctl status <servicio>
```

Y logs:

```bash
journalctl -u <servicio> -b
```

---

# 53. Diagnóstico antes de reiniciar

No reiniciar inmediatamente.

Primero recopilar:

```bash
date
uname -a
uptime
free -h
df -h
systemctl --failed
ip addr
ip route
nmcli device status
nmcli connection show
mmcli --list-modems
```

Después:

```bash
journalctl -b -p err
```

y:

```bash
dmesg -T | tail -n 200
```

Esto permite conservar evidencia del fallo.

---

# 54. Diagnóstico después de reiniciar

Después del reinicio:

```bash
uptime
```

```bash
systemctl --failed
```

```bash
journalctl -b -p err
```

```bash
journalctl -b | tail -n 200
```

Para revisar el boot anterior:

```bash
journalctl -b -1
```

---

# 55. Problemas conocidos y estado

| Problema | Evidencia | Solución/acción | Estado |
|---|---|---|---|
| Disco lleno | `/` al 100% | Limpieza de logs + `journalctl` | Resuelto operacionalmente |
| `/var/log` excesivo | ~3.3 GB | `truncate` y limpieza | Requiere prevención |
| Hostname no resolvía | `unable to resolve host TVSM01` | Revisar `/etc/hostname` y `/etc/hosts` | Requiere verificación |
| ModemManager no disponible | error DBus | Reinicio/configuración de servicio | Resuelto en pruebas |
| Número de módem cambia | `modem=0`, `3`, `4` | Detección dinámica | Requiere implementación robusta |
| LTE queda `connecting` | observado con SIM | Diagnóstico por capas | Depende de causa |
| Cambio de SIM | distintos operadores | `configAPN-LTE.sh` + `autoLTE.sh` | Arquitectura definida |
| Perfiles LTE duplicados | `LTE-TELSY` | Eliminación y creación de `lte-WOM` | Corregido |
| Pantalla negra | después de arranque | Aislar autostart/DRM/driver | Investigación |
| Oops del kernel | `WS_xinchDSI_Screen` | No cargar módulo indiscriminadamente | Riesgo identificado |
| Congelamiento | pantalla/SSH | reinicio forzado recuperó equipo | Causa raíz pendiente |
| Autostart conflictivo | procesos de TELSY | migrar backend/driver a systemd | Arquitectura recomendada |
| GPS/ModemManager | GPS detiene MM | coordinar ambos componentes | Requiere revisión |

---

# 56. Checklist de diagnóstico LTE

## Hardware

```bash
lsusb
lsusb -t
ls -l /dev/cdc-wdm*
```

## ModemManager

```bash
systemctl status ModemManager
mmcli --list-modems
```

## Módem

```bash
mmcli --modem=<ID>
```

## NetworkManager

```bash
systemctl status NetworkManager
nmcli device status
nmcli connection show
```

## SIM

Verificar mediante `mmcli` que la SIM esté detectada y registrada.

## Perfil

```bash
nmcli connection show "<perfil>"
```

## IP

```bash
ip addr
ip route
```

## Prueba

```bash
ping -c 3 8.8.8.8
```

---

# 57. Checklist de diagnóstico de pantalla

```bash
uname -a
```

```bash
ls -l /dev/dri/
```

```bash
ls -l /dev/fb*
```

```bash
ls -l /sys/class/drm/
```

```bash
readlink -f /sys/class/drm/card1/device/driver
```

```bash
lsmod | grep -Ei 'vc4|xinch|dsi'
```

```bash
dmesg -T | grep -Ei 'drm|vc4|dsi|xinch|screen'
```

```bash
journalctl -b -p err
```

```bash
cat /boot/config.txt
```

```bash
cat /etc/xdg/lxsession/LXDE-pi/autostart
```

---

# 58. Checklist de diagnóstico de arranque TELSY

```bash
systemctl --failed
```

```bash
systemctl list-units --type=service | grep -Ei 'telsy|delete'
```

```bash
ps aux | grep -Ei 'telsy|django|daphne|vspm'
```

```bash
cat /etc/xdg/lxsession/LXDE-pi/autostart
```

```bash
systemctl status delete_VAR.service
```

```bash
journalctl -u delete_VAR.service -b
```

---

# 59. Estrategia recomendada para systemd

Para producción, estructurar TELSY aproximadamente así:

```text
delete_VAR.service
        │
        ▼
TELSY backend service
        │
        ├── Django/Daphne
        │
        └── dependencias Python

TELSY driver service
        │
        └── vspm-pm6750

LTE auto-connect service
        │
        ├── ModemManager
        └── NetworkManager

LXDE
        │
        └── navegador / WebApp
```

Las dependencias deben definirse explícitamente.

No ejecutar todos los procesos como comandos independientes dentro del autostart de LXDE.

---

# 60. Reglas para autoLTE.sh en producción

El script debe:

- usar Bash robusto;
- registrar eventos;
- utilizar rutas absolutas;
- detectar el módem dinámicamente;
- no depender del número `modem=0`;
- esperar disponibilidad del módem;
- verificar SIM;
- identificar operador;
- seleccionar APN;
- activar el perfil;
- esperar la conexión;
- verificar IP;
- realizar ping;
- devolver código de error apropiado;
- no modificar innecesariamente perfiles existentes;
- evitar conexiones simultáneas de múltiples perfiles;
- poder ejecutarse varias veces sin crear duplicados.

Idealmente debe ser **idempotente**.

---

# 61. Ejemplo de estructura conceptual de autoLTE.sh

```bash
#!/bin/bash

set -u

# 1. Esperar ModemManager
# 2. Detectar módem
# 3. Verificar SIM
# 4. Obtener operador
# 5. Determinar perfil
# 6. Desconectar perfiles incompatibles
# 7. Activar perfil
# 8. Esperar IP
# 9. Hacer ping
# 10. Registrar resultado
```

No implementar valores de operador/APN sin comprobar los valores definitivos del proyecto.

---

# 62. Idempotencia

Ejecutar dos veces:

```bash
sudo ./configAPN-LTE.sh
```

no debería producir:

```text
lte-CLARO
lte-CLARO-1
lte-CLARO-2
lte-CLARO-3
```

El script debe:

1. comprobar si existe;
2. crear si no existe;
3. modificar si necesita actualizarse;
4. dejarlo intacto si ya está correcto.

---

# 63. Seguridad y robustez de scripts

Preferir:

```bash
set -euo pipefail
```

cuando el script haya sido diseñado para manejar correctamente estas condiciones.

Validar comandos críticos:

```bash
command -v nmcli
command -v mmcli
command -v ping
```

Ejemplo:

```bash
if ! command -v nmcli >/dev/null 2>&1; then
    echo "ERROR: nmcli no está disponible"
    exit 1
fi
```

---

# 64. Logs de diagnóstico

Para scripts de producción:

```text
/var/log/
```

debe utilizarse con cuidado porque el dispositivo tiene eMMC limitada.

Evitar logs infinitos.

Preferir:

- `journalctl`;
- rotación;
- límites de tamaño;
- mensajes relevantes;
- no registrar datos innecesarios continuamente.

---

# 65. Diagnóstico de consumo de almacenamiento

Comandos:

```bash
df -h
```

```bash
sudo du -xh --max-depth=1 /
```

```bash
sudo du -xh --max-depth=1 /var
```

```bash
sudo du -xh --max-depth=1 /var/log
```

```bash
journalctl --disk-usage
```

Archivos grandes:

```bash
sudo find /var/log -type f -size +100M -ls
```

---

# 66. Diagnóstico de espacio antes de instalar/modificar

Siempre comprobar:

```bash
df -h /
```

y:

```bash
df -i /
```

Esto es especialmente importante en CM4 con eMMC de 8 GB.

---

# 67. Diferencias entre prototipo y producción

No asumir que una configuración válida en un prototipo será idéntica en producción.

Deben compararse:

- kernel;
- `/boot/config.txt`;
- módulos;
- overlays;
- paquetes;
- servicios;
- autostart;
- perfiles NetworkManager;
- reglas udev;
- puertos seriales;
- hardware;
- eMMC;
- pantalla.

Comando útil:

```bash
uname -a
```

y:

```bash
lsmod
```

---

# 68. Principio de comparación con equipo funcional

Cuando exista:

- un equipo funcional;
- un equipo con fallo;

comparar exactamente:

```bash
uname -a
```

```bash
cat /boot/config.txt
```

```bash
lsmod
```

```bash
systemctl --failed
```

```bash
ls -l /sys/class/drm/
```

```bash
nmcli connection show
```

```bash
mmcli --list-modems
```

Esto es preferible a modificar múltiples parámetros simultáneamente.

---

# 69. Flujo general de troubleshooting

```text
PROBLEMA
   │
   ▼
¿Sistema responde por SSH?
   │
   ├── NO
   │    └── hardware / kernel / red / freeze
   │
   └── SÍ
        │
        ▼
   ¿Servicio correspondiente está activo?
        │
        ├── NO → systemd/logs
        │
        └── SÍ
             │
             ▼
        ¿Dispositivo hardware existe?
             │
             ├── NO → USB/kernel/driver
             │
             └── SÍ
                  │
                  ▼
             ¿Configuración correcta?
                  │
                  ├── NO → corregir
                  │
                  └── SÍ
                       │
                       ▼
                  ¿Comunicación funciona?
                       │
                       ├── NO → capa inferior
                       │
                       └── SÍ → aplicación
```

---

# 70. Reglas para Codex

Cuando Codex analice TELSY:

## Debe

- respetar la arquitectura existente;
- analizar código antes de modificarlo;
- identificar dependencias;
- conservar compatibilidad con Raspberry Pi CM4;
- considerar Raspberry Pi 4 cuando corresponda;
- considerar Python/Django/Channels/Daphne;
- considerar serial/I2C/GPIO;
- considerar ModemManager/NetworkManager;
- considerar el módem Huawei ME906s;
- considerar el sistema de pantalla DSI/VC4/DRM;
- considerar los scripts Bash;
- considerar systemd;
- considerar LXDE;
- evitar introducir dependencias innecesarias;
- documentar cualquier cambio;
- indicar exactamente qué archivos modifica;
- indicar comandos de validación;
- diferenciar hechos de hipótesis.

## No debe

- asumir que `modem=0` siempre existe;
- asumir que `cdc-wdm0` siempre es permanente;
- reinstalar dependencias sin necesidad;
- reemplazar componentes funcionales sin justificarlo;
- eliminar perfiles NetworkManager indiscriminadamente;
- modificar `/boot/config.txt` sin comparar contra un equipo funcional;
- cargar módulos de kernel de pantalla manualmente sin verificar compatibilidad;
- mover el virtualenv dentro de `telsy/`;
- introducir Chromium/kiosco si no es requerido;
- considerar una pantalla negra automáticamente como fallo de hardware;
- considerar un módem `connecting` automáticamente como falta de señal;
- considerar un reinicio como solución definitiva;
- generar cambios masivos sin pruebas intermedias.

---

# 71. Criterio de análisis de cambios

Todo cambio propuesto por Codex debe presentar:

### 1. Problema

Qué comportamiento se quiere corregir.

### 2. Causa

Qué evidencia demuestra la causa.

### 3. Cambio

Qué archivo/componente se modificará.

### 4. Impacto

Qué otros componentes podrían verse afectados.

### 5. Prueba

Qué comando o procedimiento valida el cambio.

### 6. Rollback

Cómo volver al estado anterior.

---

# 72. Formato recomendado para diagnóstico

Codex debe responder preferentemente con:

```text
HECHO
- Evidencia observada.

INFERENCIA
- Explicación probable.

REQUIERE VALIDACIÓN
- Prueba necesaria.

SOLUCIÓN PROPUESTA
- Cambio concreto.

VALIDACIÓN
- Comandos/procedimiento.

RIESGO
- Posibles efectos secundarios.
```

---

# 73. Comandos de referencia rápida

## Sistema

```bash
uname -a
hostname
uptime
free -h
df -h
```

## Procesos

```bash
ps aux
top
pgrep -af <proceso>
```

## Servicios

```bash
systemctl --failed
systemctl status <servicio>
systemctl cat <servicio>
journalctl -u <servicio> -b
```

## Logs

```bash
journalctl -b
journalctl -b -p err
dmesg -T
```

## Red

```bash
ip addr
ip route
nmcli device status
nmcli connection show
```

## LTE

```bash
mmcli --list-modems
mmcli --modem=<ID>
systemctl status ModemManager
systemctl status NetworkManager
```

## USB

```bash
lsusb
lsusb -t
```

## Pantalla

```bash
ls -l /dev/dri/
ls -l /dev/fb*
ls -l /sys/class/drm/
lsmod
```

---

# 74. Estado funcional conocido

Se logró validar:

- TELSY sobre Raspberry Pi CM4 de 2 GB RAM / 8 GB eMMC.
- Conexión LTE con Movistar.
- Conexión LTE con Claro.
- Posteriormente conexión nuevamente con Movistar.
- Uso de Huawei ME906s.
- Detección mediante ModemManager.
- Gestión de perfiles mediante NetworkManager.
- Configuración de perfiles APN.
- Identificación de problemas relacionados con cambio de SIM.
- Identificación de variación del identificador del módem.
- Identificación de problemas de autostart/pantalla.
- Identificación de un Oops relacionado con el módulo DSI.
- Identificación de crecimiento excesivo de `/var/log`.

---

# 75. Pendientes técnicos relevantes

## LTE

- Implementar definitivamente `configAPN-LTE.sh`.
- Implementar definitivamente `autoLTE.sh`.
- Detección robusta de SIM.
- Detección robusta de operador.
- Asociación operador → APN.
- Evitar dependencia de `modem=0`.
- Servicio systemd para autoconexión.
- Validación de conectividad.
- Manejo de timeout.
- Manejo de ausencia de SIM.
- Manejo de señal insuficiente.
- Manejo de pérdida de conexión.
- Integración correcta con GPS.

## Pantalla

- Determinar causa raíz de pantalla negra.
- Determinar compatibilidad exacta entre kernel y `WS_xinchDSI_Screen`.
- Comparar equipo funcional vs equipo con fallo.
- Evitar carga manual insegura del módulo.
- Revisar dependencia entre autostart y pantalla.
- Determinar si el congelamiento está relacionado con DRM/driver.

## Arranque

- Consolidar servicios TELSY en systemd.
- Mantener navegador en LXDE.
- Definir dependencias de servicios.
- Evitar scripts largos bloqueando autostart.

## Almacenamiento

- Implementar límites de logs.
- Verificar journald.
- Evitar crecimiento de `/var/log`.
- Mantener espacio libre suficiente en eMMC.

---

# 76. Filosofía general de mantenimiento

TELSY debe tratarse como un sistema embebido de producción, no como un computador de propósito general.

Por tanto:

- los cambios deben ser reproducibles;
- las configuraciones deben estar documentadas;
- los servicios deben ser recuperables;
- los scripts deben ser idempotentes;
- los logs deben estar controlados;
- el hardware debe diagnosticarse por capas;
- las modificaciones de kernel/DT/DRM deben realizarse con especial precaución;
- el cambio de SIM debe ser transparente para el usuario;
- los fallos deben poder diagnosticarse remotamente mediante SSH;
- cada solución debe dejar trazabilidad.

---

# 77. Información que Codex debe solicitar si no está disponible

Antes de modificar componentes críticos, Codex debe solicitar o inspeccionar:

### Sistema

```bash
uname -a
cat /etc/os-release
```

### Hardware

```bash
lsusb
lsusb -t
```

### LTE

```bash
mmcli --list-modems
mmcli --modem=<ID>
nmcli device status
nmcli connection show
```

### Pantalla

```bash
ls -l /sys/class/drm/
ls -l /dev/dri/
ls -l /dev/fb*
lsmod
```

### Servicios

```bash
systemctl --failed
systemctl list-units --type=service
```

### TELSY

```bash
find /home/pi/telsy-monitor -maxdepth 2 -type f | sort
```

### Autostart

```bash
cat /etc/xdg/lxsession/LXDE-pi/autostart
```

### Boot

```bash
cat /boot/config.txt
```

---

# 78. Regla final para futuras sesiones con Codex

Este documento debe considerarse **contexto técnico inicial**, no una especificación inmutable.

Cuando Codex encuentre discrepancias entre este documento y el sistema real:

1. debe priorizar la evidencia actual del dispositivo;
2. debe indicar la discrepancia;
3. debe evitar asumir que una configuración histórica sigue vigente;
4. debe pedir los comandos necesarios para verificar;
5. debe actualizar la documentación cuando se confirme una nueva configuración.

**Nunca asumir que una hipótesis histórica es un hecho actual.**

---

## Fin del contexto técnico TELSY / TELSY-QM
