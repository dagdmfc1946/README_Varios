# Cambios de arranque automático (autostart) de TELSY

## 1. Objetivo

Este documento registra la configuración utilizada para que la Raspberry Pi inicie TELSY automáticamente y muestre únicamente la aplicación en modo kiosco.

El arranque queda dividido en tres responsabilidades independientes:

1. `telsy-server.service`: inicia el servidor Django.
2. `telsy-driver.service`: inicia el driver PM6750.
3. `/home/pi/telsy-monitor/startweb.sh`: inicia Chromium en modo kiosco desde la sesión gráfica LXDE.

Esta separación evita que el servidor o el driver bloqueen el arranque de la sesión gráfica.

## 2. Configuración actual de LXDE

Archivo modificado:

```text
/etc/xdg/lxsession/LXDE-pi/autostart
```

La configuración relevante actual es:

```text
#@lxpanel --profile LXDE-pi
#@pcmanfm --desktop --profile LXDE-pi
@xscreensaver -no-splash

## El servidor y el driver se ejecutan como servicios systemd independientes.
## El navegador se inicia desde la sesión gráfica, cuando X ya está disponible.
@/home/pi/telsy-monitor/startweb.sh
```

### Motivo de mantener `lxpanel` comentado

```text
#@lxpanel --profile LXDE-pi
```

`lxpanel` inicia el panel, la barra de tareas y elementos del escritorio. Se mantiene comentado intencionalmente porque el dispositivo debe iniciar TELSY sin mostrar una sesión de escritorio navegable.

### Motivo de mantener `pcmanfm` comentado

```text
#@pcmanfm --desktop --profile LXDE-pi
```

`pcmanfm --desktop` muestra el escritorio y sus iconos. También se mantiene comentado intencionalmente para que el usuario final no pueda acceder al escritorio normal.

Que ambas líneas estén comentadas no es un error en esta instalación: forma parte del modo kiosco.

### Línea de `xscreensaver`

```text
@xscreensaver -no-splash
```

Inicia el salvapantallas sin mostrar su pantalla inicial. Su comportamiento depende de la configuración del salvapantallas del sistema.

### Línea que inicia la aplicación web

```text
@/home/pi/telsy-monitor/startweb.sh
```

Esta línea no inicia directamente el servidor ni el driver. Solo ejecuta el script que abre Chromium. El script espera a que Django responda en `127.0.0.1:8000/home/` antes de abrir el navegador.

La ejecución ocurre desde `autostart`, después de que la sesión gráfica de LXDE ya ha sido creada. Por eso `startweb.sh` puede utilizar:

```bash
DISPLAY=:0
XAUTHORITY=/home/pi/.Xauthority
```

Estas variables permiten que Chromium se conecte al servidor X de la sesión gráfica.

## 3. Por qué el servidor y el driver no están en `autostart`

La primera estrategia intentaba iniciar los tres scripts desde `autostart`:

```text
@sh /home/pi/telsy-monitor/startserver.sh
@sh /home/pi/telsy-monitor/startweb.sh
@sh /home/pi/telsy-monitor/run_driver.sh
```

También se probó una variante con procesos en segundo plano y logs en `/tmp`:

```text
@sh -c '/home/pi/telsy-monitor/startserver.sh >>/tmp/telsy-server.log 2>&1 &'
@sh -c '/home/pi/telsy-monitor/startweb.sh >>/tmp/telsy-web.log 2>&1 &'
@sh -c '/home/pi/telsy-monitor/run_driver.sh >>/tmp/telsy-driver.log 2>&1 &'
```

Esa estrategia tiene varios inconvenientes:

- `startserver.sh` es un proceso de larga duración: `manage.py runserver` no termina mientras el servidor está activo.
- El driver también es un proceso de larga duración y usa el puerto serie `/dev/ttyAMA1`.
- Si uno de los procesos falla, `autostart` no ofrece un estado claro ni un reinicio controlado.
- Los procesos iniciados desde LXSession pueden heredar un entorno distinto al esperado.
- Es más difícil saber si el problema es del servidor, del driver o del navegador.
- Un proceso en primer plano puede retrasar o bloquear otros componentes de la sesión.

Por esas razones, servidor y driver se trasladaron a unidades independientes de `systemd`.

## 3.1. Por qué se usan servicios `systemd` independientes

`systemd` es el administrador de servicios de Linux. Se encarga de iniciar procesos durante el arranque, controlar su estado, registrar sus mensajes y reiniciarlos cuando corresponde.

En TELSY se utilizan dos servicios separados porque el servidor Django y el driver PM6750 son procesos distintos, con responsabilidades, dependencias y posibles fallos diferentes.

### Servidor Django

El servidor Django entrega la aplicación web en `127.0.0.1:8000`. Debe permanecer ejecutándose para que Chromium pueda cargar la interfaz y para que la aplicación pueda atender sus peticiones HTTP y WebSocket.

Al administrarlo como `telsy-server.service`, `systemd` puede:

- Iniciarlo automáticamente al encender la Raspberry Pi.
- Esperar a que la red esté disponible.
- Ejecutarlo con el entorno virtual y el directorio correctos.
- Reiniciarlo si termina inesperadamente.
- Mostrar su estado como `active`, `failed` o `inactive`.
- Guardar sus mensajes en el journal del sistema.

### Driver PM6750

El driver ejecuta `vspm-pm6750` y mantiene la comunicación con el equipo multiparamétrico mediante `/dev/ttyAMA1`. No necesita una pantalla gráfica ni Chromium para funcionar.

Al administrarlo como `telsy-driver.service`, `systemd` puede:

- Iniciarlo automáticamente al encender la Raspberry Pi.
- Esperar a que exista el dispositivo `/dev/ttyAMA1`.
- Mantenerlo separado del servidor web.
- Reiniciarlo si el binario termina por un error.
- Mostrar su estado y sus mensajes de forma independiente.

### Ventajas de mantenerlos separados

La independencia entre ambos servicios permite localizar rápidamente el origen de un fallo:

```text
telsy-server.service  -> aplicación web, Django, HTTP y WebSocket
telsy-driver.service  -> comunicación serie y equipo PM6750
startweb.sh           -> sesión gráfica y Chromium
```

Por ejemplo:

- Si el servidor falla, el driver puede seguir comunicándose con el PM6750.
- Si el driver falla, Django puede seguir mostrando la aplicación y reportar el problema.
- Si Chromium no inicia, servidor y driver pueden continuar activos.
- Un reinicio del servidor no necesita reiniciar el driver ni la sesión gráfica.
- Los logs de cada proceso no se mezclan.

Esto también evita una condición importante: ejecutar dos veces el mismo proceso. No se debe iniciar manualmente `startserver.sh` mientras `telsy-server.service` está activo, ni `run_driver.sh` mientras `telsy-driver.service` está activo. En el primer caso habría dos servidores intentando usar el mismo puerto; en el segundo, dos procesos intentarían usar el mismo puerto serie.

### Por qué no se ejecutan desde `autostart`

`autostart` pertenece a la sesión gráfica del usuario `pi`. Su función en esta configuración es iniciar únicamente lo que necesita X11: `startweb.sh` y, a través de este, Chromium.

El servidor y el driver deben comenzar aunque todavía no exista una sesión gráfica. Por eso pertenecen a `systemd`, que inicia servicios del sistema antes de LightDM y LXDE. De esta forma Chromium puede esperar a que Django esté disponible sin que LXDE tenga que controlar procesos de larga duración.

En resumen:

```text
systemd  -> inicia y supervisa servidor y driver desde el arranque del sistema
LXDE     -> crea la sesión gráfica
autostart -> ejecuta startweb.sh cuando X11 ya está disponible
Chromium -> muestra TELSY en modo kiosco
```

## 4. Servicio del servidor Django

Archivo instalado fuera del repositorio:

```text
/etc/systemd/system/telsy-server.service
```

Contenido utilizado:

```ini
[Unit]
Description=TELSY Django server
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/telsy-monitor/telsy
ExecStart=/home/pi/telsy-monitor/telsy_venv/bin/python manage.py runserver 127.0.0.1:8000
Restart=on-failure
RestartSec=5
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target
```

### Funcionamiento

- Se ejecuta con el usuario `pi`, no con `sudo`.
- Usa directamente el Python del entorno virtual `telsy_venv`.
- Trabaja desde `/home/pi/telsy-monitor/telsy`, donde está `manage.py`.
- Escucha solo en `127.0.0.1:8000`, porque Chromium se ejecuta en la misma Raspberry Pi.
- Espera a que la red esté disponible.
- Se reinicia automáticamente si termina con error.

Comandos de administración:

```bash
sudo systemctl enable telsy-server.service
sudo systemctl start telsy-server.service
sudo systemctl stop telsy-server.service
sudo systemctl restart telsy-server.service
sudo systemctl status telsy-server.service
```

Logs:

```bash
sudo journalctl -u telsy-server.service -n 100 --no-pager
sudo journalctl -u telsy-server.service -f
```

Prueba HTTP:

```bash
curl --fail http://127.0.0.1:8000/home/
```

## 5. Servicio del driver PM6750

Archivo instalado fuera del repositorio:

```text
/etc/systemd/system/telsy-driver.service
```

Contenido utilizado:

```ini
[Unit]
Description=TELSY PM6750 driver
After=dev-ttyAMA1.device
Wants=dev-ttyAMA1.device

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/telsy-monitor/telsy
ExecStart=/home/pi/telsy-monitor/telsy/vspm-pm6750 --se0_path=/dev/ttyAMA1
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Funcionamiento

- Se ejecuta con el usuario `pi`.
- Usa `/home/pi/telsy-monitor/telsy/vspm-pm6750`.
- Usa el puerto serie `/dev/ttyAMA1`.
- Espera la disponibilidad del dispositivo serie.
- Se reinicia automáticamente si termina con error.

Comandos de administración:

```bash
sudo systemctl enable telsy-driver.service
sudo systemctl start telsy-driver.service
sudo systemctl stop telsy-driver.service
sudo systemctl restart telsy-driver.service
sudo systemctl status telsy-driver.service
```

Logs:

```bash
sudo journalctl -u telsy-driver.service -n 100 --no-pager
sudo journalctl -u telsy-driver.service -f
```

## 6. Cambios realizados en los scripts

### `startserver.sh`

Archivo:

```text
/home/pi/telsy-monitor/startserver.sh
```

Cambios relevantes:

- Se obtiene la ruta real del script con `readlink -f`.
- Se valida el cambio al directorio del proyecto.
- Se activa `/home/pi/telsy-monitor/telsy_venv`.
- Se entra en `/home/pi/telsy-monitor/telsy`.
- Se elimina la necesidad de `sudo` para ejecutar Django.
- Se usa el Python del entorno virtual.
- Se utiliza `exec` para que el proceso del servicio sea directamente Django.

El script sigue siendo útil para ejecución manual:

```bash
/home/pi/telsy-monitor/startserver.sh
```

Sin embargo, el arranque automático actual utiliza `telsy-server.service`, no este script desde LXDE.

### `run_driver.sh`

Archivo:

```text
/home/pi/telsy-monitor/run_driver.sh
```

Cambios relevantes:

- Se calcula la ruta absoluta del proyecto.
- Se entra en `/home/pi/telsy-monitor/telsy`.
- Se ejecuta el binario con `exec`.
- Se conserva el argumento fijo `--se0_path=/dev/ttyAMA1`.
- Se mantienen argumentos adicionales recibidos mediante `"$@"`.

El script sigue siendo útil para ejecución manual:

```bash
/home/pi/telsy-monitor/run_driver.sh
```

El arranque automático actual utiliza `telsy-driver.service`, no este script desde LXDE.

### `startweb.sh`

Archivo:

```text
/home/pi/telsy-monitor/startweb.sh
```

Cambios relevantes:

- Define `DISPLAY=:0` para la sesión gráfica.
- Define `XAUTHORITY` usando `/home/pi/.Xauthority` como valor predeterminado.
- Espera a que el servidor responda antes de iniciar Chromium.
- Comprueba `http://127.0.0.1:8000/home/` mediante `curl`.
- Usa Chromium en modo `--kiosk` y pantalla completa.
- Mantiene las opciones necesarias para pantalla táctil.
- Fuerza `--ozone-platform=x11` para la sesión gráfica X11.
- Desactiva GPU y usa `--disable-dev-shm-usage` para mejorar compatibilidad en este equipo.

El flujo es:

```text
Inicio de LXDE
    -> autostart ejecuta startweb.sh
        -> startweb.sh espera a Django
            -> Chromium abre /home/ en modo kiosco
```

## 7. NetworkManager y `views.py`

En `telsy/monitor/views.py` existía una comprobación durante la carga de la aplicación que hacía lo siguiente:

```python
network_manager = getoutput('systemctl status NetworkManager | grep "Active:"')
if network_manager.find('running') == -1:
    system('systemctl restart NetworkManager')
```

Esto era problemático porque la aplicación podía intentar reiniciar un servicio del sistema desde el proceso Django. El reinicio podía provocar una solicitud de autenticación o interrumpir temporalmente la red.

El comportamiento actual es comprobar el estado sin reiniciar el servicio:

```python
network_manager = getoutput('systemctl is-active NetworkManager')
if network_manager != 'active':
    print('NetworkManager is not active:', network_manager)
```

La responsabilidad de iniciar NetworkManager queda en `systemd`, no en Django.

Configuración recomendada:

```bash
sudo systemctl enable --now NetworkManager
```

Comprobación:

```bash
systemctl is-enabled NetworkManager
systemctl is-active NetworkManager
nmcli device status
```

Las acciones de conexión Wi-Fi de la aplicación siguen usando `nmcli` a través de `wifi_management.py`. Eso es diferente de reiniciar NetworkManager durante el arranque.

## 8. Orden real de inicio

El arranque esperado es:

```text
1. systemd inicia NetworkManager.
2. systemd inicia telsy-driver.service.
3. systemd inicia telsy-server.service.
4. LightDM inicia sesión automáticamente con el usuario pi.
5. LXDE lee /etc/xdg/lxsession/LXDE-pi/autostart.
6. LXDE ejecuta startweb.sh.
7. startweb.sh espera hasta que http://127.0.0.1:8000/home/ responda.
8. Chromium abre TELSY en modo kiosco.
9. lxpanel y pcmanfm permanecen desactivados para ocultar el escritorio.
```

El driver y el servidor pueden iniciar antes que la sesión gráfica. Esto es intencional: no necesitan `DISPLAY`. Chromium sí necesita la sesión gráfica, por eso se inicia desde `autostart`.

## 9. Diagnóstico

### Verificar ambos servicios

```bash
systemctl is-active telsy-server.service
systemctl is-active telsy-driver.service
systemctl is-enabled telsy-server.service
systemctl is-enabled telsy-driver.service
```

El resultado esperado es:

```text
active
active
enabled
enabled
```

### Verificar el servidor

```bash
curl --fail -I http://127.0.0.1:8000/home/
sudo systemctl status telsy-server.service --no-pager
sudo journalctl -u telsy-server.service -n 100 --no-pager
```

### Verificar el driver

```bash
sudo systemctl status telsy-driver.service --no-pager
sudo journalctl -u telsy-driver.service -n 100 --no-pager
ls -l /dev/ttyAMA1
```

### Verificar el navegador

Desde la sesión gráfica, comprobar:

```bash
printf 'DISPLAY=%s\n' "$DISPLAY"
printf 'XAUTHORITY=%s\n' "$XAUTHORITY"
pgrep -a chromium
```

El script puede ejecutarse manualmente para ver errores directamente:

```bash
/home/pi/telsy-monitor/startweb.sh
```

### Verificar el autostart

```bash
sed -n '1,160p' /etc/xdg/lxsession/LXDE-pi/autostart
```

Debe existir esta línea activa:

```text
@/home/pi/telsy-monitor/startweb.sh
```

Y estas líneas deben permanecer comentadas en modo kiosco:

```text
#@lxpanel --profile LXDE-pi
#@pcmanfm --desktop --profile LXDE-pi
```

## 10. Recuperación temporal del escritorio

Si Chromium no abre y se necesita recuperar el escritorio para diagnosticar, editar temporalmente:

```bash
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```

Cambiar:

```text
#@lxpanel --profile LXDE-pi
#@pcmanfm --desktop --profile LXDE-pi
```

Por:

```text
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi
```

Después de reiniciar o reiniciar la sesión gráfica aparecerán el panel y el escritorio. Una vez terminado el diagnóstico, volver a comentar ambas líneas para restaurar el modo kiosco.

También se puede detener Chromium desde una terminal:

```bash
pkill -f chromium
```

No se debe desactivar ni borrar los servicios del servidor o del driver solo porque Chromium no aparezca: primero hay que comprobar sus estados y sus logs.

## 11. Reinstalar o actualizar las unidades systemd

Si se modifica alguno de los archivos de servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl restart telsy-server.service
sudo systemctl restart telsy-driver.service
```

Para habilitarlos durante cada arranque:

```bash
sudo systemctl enable telsy-server.service
sudo systemctl enable telsy-driver.service
```

Para iniciar ambos inmediatamente:

```bash
sudo systemctl start telsy-server.service
sudo systemctl start telsy-driver.service
```

## 12. Desactivar el arranque automático de servidor o driver

Para detener temporalmente el servidor:

```bash
sudo systemctl disable --now telsy-server.service
```

Para detener temporalmente el driver:

```bash
sudo systemctl disable --now telsy-driver.service
```

Para volver a habilitarlos:

```bash
sudo systemctl enable --now telsy-server.service
sudo systemctl enable --now telsy-driver.service
```

## 13. Reversión completa de esta estrategia

La reversión no es necesaria mientras la configuración actual funcione. Si se necesita volver al esquema anterior, el procedimiento es:

1. Detener y deshabilitar los dos servicios:

   ```bash
   sudo systemctl disable --now telsy-server.service
   sudo systemctl disable --now telsy-driver.service
   ```

2. Editar `autostart` y sustituir el arranque de Chromium por el esquema deseado.

3. No ejecutar simultáneamente el servicio y el script equivalente: eso produciría dos servidores Django o dos procesos del driver.

4. Si se elimina una unidad, conservar una copia antes:

   ```bash
   sudo cp /etc/systemd/system/telsy-server.service /home/pi/telsy-monitor/
   sudo cp /etc/systemd/system/telsy-driver.service /home/pi/telsy-monitor/
   ```

5. Después de eliminar unidades:

   ```bash
   sudo systemctl daemon-reload
   ```

## 14. Riesgos y precauciones

- No ejecutar `startserver.sh` manualmente mientras `telsy-server.service` esté activo.
- No ejecutar `run_driver.sh` manualmente mientras `telsy-driver.service` esté activo.
- No añadir de nuevo las tres ejecuciones directas a `autostart` mientras existan los servicios habilitados.
- No usar `sudo python manage.py runserver` para este arranque: cambia permisos y usa otro intérprete Python.
- No cambiar `127.0.0.1:8000` por una dirección pública sin una necesidad específica.
- El usuario `pi` debe tener permisos para ejecutar el binario y acceder a `/dev/ttyAMA1`.
- Si se cambia el usuario de un servicio, revisar permisos del puerto serie, de la base de datos y del proyecto.
- `autostart` es una configuración del sistema y requiere privilegios para editarse.

## 15. Validaciones realizadas

Se validó la sintaxis de los tres scripts:

```bash
bash -n /home/pi/telsy-monitor/startserver.sh
bash -n /home/pi/telsy-monitor/startweb.sh
bash -n /home/pi/telsy-monitor/run_driver.sh
```

También se validó Django:

```bash
cd /home/pi/telsy-monitor/telsy
/home/pi/telsy-monitor/telsy_venv/bin/python manage.py check
```

Resultado registrado:

```text
System check identified no issues (0 silenced).
```

En la configuración actual se comprobó además que:

- `telsy-server.service` está habilitado y activo.
- `telsy-driver.service` está habilitado y activo.
- El servidor responde por HTTP en `127.0.0.1:8000`.
- El driver está ejecutando `vspm-pm6750` con `/dev/ttyAMA1`.
- `lxpanel` y `pcmanfm` están comentados deliberadamente.

## 16. Resumen final

La línea:

```text
@/home/pi/telsy-monitor/startweb.sh
```

se añadió a `autostart` para iniciar únicamente la parte que necesita el entorno gráfico: Chromium.

El servidor y el driver no se lanzan desde `autostart`; funcionan como servicios `systemd` independientes, con arranque automático, reinicio ante fallo y logs consultables por separado.

Mantener comentadas estas líneas:

```text
#@lxpanel --profile LXDE-pi
#@pcmanfm --desktop --profile LXDE-pi
```

es correcto para el objetivo del dispositivo: iniciar TELSY y ocultar el escritorio normal.
