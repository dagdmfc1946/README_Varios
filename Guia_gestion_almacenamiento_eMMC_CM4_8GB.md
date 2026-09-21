# Gestión de almacenamiento eMMC — Raspberry Pi CM4 (8 GB)

## 1. Objetivo

Guía práctica para administrar el almacenamiento de un **Raspberry Pi Compute Module 4 (CM4) con eMMC de 8 GB**, incluyendo:

- Verificar capacidad y espacio disponible.
- Identificar qué directorios y archivos ocupan más espacio.
- Limpiar archivos temporales del sistema y del usuario.
- Limpiar cachés y paquetes innecesarios.
- Gestionar logs del sistema y de los servicios.
- Inspeccionar qué procesos/servicios están generando logs.
- Determinar qué puede eliminarse de forma razonablemente segura.
- Evitar eliminar archivos necesarios para el funcionamiento del sistema.
- Solucionar la advertencia `sudo: unable to resolve host TVSM01`.

> **Advertencia:** En un eMMC de 8 GB cada GB es importante, pero no se debe eliminar contenido de `/usr`, `/etc`, `/var/lib` u otras rutas del sistema sin identificar previamente qué contiene. La regla general es **medir → identificar → revisar → limpiar → verificar**.

---

# 2. Verificar el almacenamiento

## 2.1. Espacio disponible en la partición raíz

```bash
df -h /
```

Ejemplo:

```text
S.ficheros     Tamaño Usados  Disp Uso% Montado en
/dev/root        6,9G   5,0G   1,9G  73% /
```

Los campos importantes son:

- `Tamaño`: capacidad de la partición.
- `Usados`: espacio utilizado.
- `Disp`: espacio disponible.
- `Uso%`: porcentaje utilizado.
- `/`: sistema de archivos raíz.

### Estado recomendado

En un CM4 de 8 GB conviene evitar operar permanentemente cerca del 100 %.

Como referencia práctica:

- **< 70 %:** normalmente hay margen suficiente.
- **70–85 %:** conviene revisar periódicamente.
- **85–95 %:** recomendable realizar limpieza.
- **> 95 %:** situación de riesgo; primero liberar espacio.
- **100 %:** pueden aparecer fallos de servicios, logs, actualizaciones, bases de datos y aplicaciones.

---

## 2.2. Ver discos, particiones y eMMC

```bash
lsblk
```

Para obtener más información:

```bash
lsblk -f
```

También:

```bash
sudo fdisk -l
```

`lsblk` permite comprobar qué dispositivo corresponde al almacenamiento utilizado por el sistema.

---

# 3. Identificar qué ocupa más espacio

## 3.1. Revisar los directorios principales de `/`

```bash
sudo du -xh --max-depth=1 / 2>/dev/null | sort -h
```

Una salida típica puede mostrar:

```text
6,0M    /etc
350M    /home
3,0G    /usr
3,2G    /var
6,9G    /
```

Esto permite determinar dónde concentrar la investigación.

### Importante

No significa que todos los directorios grandes deban limpiarse.

Por ejemplo:

- `/usr`: contiene gran parte del software instalado. **No debe limpiarse manualmente.**
- `/etc`: contiene configuración del sistema. **No eliminar manualmente.**
- `/home`: contiene archivos de usuarios y aplicaciones.
- `/var`: contiene datos variables, cachés, bases de datos y logs. **Debe investigarse antes de borrar.**

---

## 3.2. Revisar `/var`

```bash
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h
```

Prestar especial atención a:

```text
/var/log
/var/cache
/var/lib
```

No eliminar `/var/*` de forma indiscriminada.

---

## 3.3. Revisar `/home`

```bash
du -xh --max-depth=2 /home/pi 2>/dev/null | sort -h
```

Para encontrar los elementos más grandes:

```bash
du -xh /home/pi 2>/dev/null | sort -h | tail -30
```

Esto puede detectar:

- archivos de respaldo;
- imágenes `.img`;
- proyectos;
- entornos virtuales;
- cachés;
- archivos descargados;
- logs de aplicaciones;
- archivos temporales creados por el usuario.

---

## 3.4. Buscar archivos individuales grandes

Para archivos mayores de 100 MB:

```bash
sudo find / -xdev -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

Para mayores de 500 MB:

```bash
sudo find / -xdev -type f -size +500M -exec ls -lh {} \; 2>/dev/null
```

Antes de eliminar un archivo encontrado con `find`, comprobar:

1. qué aplicación lo utiliza;
2. si es un archivo de configuración;
3. si es un backup;
4. si es un log;
5. si puede regenerarse;
6. si pertenece a un paquete del sistema.

---

# 4. Archivos temporales del sistema

## 4.1. `/tmp`

`/tmp` contiene archivos temporales creados por aplicaciones y procesos del sistema.

Revisar:

```bash
sudo du -xh --max-depth=1 /tmp 2>/dev/null | sort -h
```

Limpiar el contenido:

```bash
sudo rm -rf /tmp/*
```

### ¿Por qué se crean?

Los programas pueden utilizar `/tmp` para:

- archivos temporales;
- sockets;
- archivos intermedios;
- procesos de instalación;
- extracción de archivos;
- operaciones de actualización;
- procesamiento temporal de datos.

En muchos sistemas, `/tmp` puede limpiarse durante el arranque o mediante mecanismos propios del sistema, pero esto depende de la configuración.

### Precaución

No ejecutar:

```bash
sudo rm -rf /tmp
```

Se debe eliminar el **contenido**, no el directorio `/tmp`.

---

## 4.2. `/var/tmp`

Revisar:

```bash
sudo du -xh --max-depth=1 /var/tmp 2>/dev/null | sort -h
```

A diferencia de `/tmp`, `/var/tmp` está destinado a datos temporales que pueden necesitar sobrevivir a un reinicio.

Por eso, antes de eliminar su contenido se debe comprobar que ningún proceso lo esté utilizando.

Si se confirma que no contiene información necesaria:

```bash
sudo rm -rf /var/tmp/*
```

---

# 5. Archivos temporales y caché del usuario

Para el usuario `pi`:

```bash
du -xh --max-depth=1 ~/.cache 2>/dev/null | sort -h
```

La caché normalmente puede regenerarse.

Para limpiar el contenido:

```bash
rm -rf ~/.cache/*
```

Esto no elimina los programas instalados; elimina datos de caché del usuario.

### ¿Por qué se genera?

Las aplicaciones utilizan cachés para:

- acelerar el arranque;
- evitar descargar nuevamente información;
- almacenar miniaturas;
- guardar archivos intermedios;
- almacenar datos temporales de navegadores y otras aplicaciones.

La limpieza puede provocar que algunas aplicaciones tarden más inicialmente mientras reconstruyen sus cachés.

---

# 6. Caché de APT

Consultar cuánto ocupa:

```bash
sudo du -sh /var/cache/apt
```

Limpiar paquetes descargados que ya no son necesarios:

```bash
sudo apt clean
```

También puede utilizarse:

```bash
sudo apt autoclean
```

### Diferencia

`apt clean` elimina los archivos `.deb` almacenados en la caché de APT.

`apt autoclean` elimina paquetes descargados que ya no pueden descargarse desde los repositorios configurados.

Estas operaciones no desinstalan los programas actualmente instalados.

---

# 7. Paquetes que ya no son necesarios

Consultar:

```bash
sudo apt autoremove
```

APT mostrará qué paquetes considera que ya no son necesarios.

**Revisar la lista antes de confirmar.**

No utilizar `autoremove` automáticamente sin revisar la propuesta en un equipo de producción.

---

# 8. Logs del sistema: `/var/log`

## 8.1. ¿Por qué existen?

Los logs registran eventos producidos por:

- kernel;
- systemd;
- servicios;
- demonios;
- NetworkManager;
- ModemManager;
- SSH;
- aplicaciones;
- procesos de usuario;
- errores;
- advertencias;
- eventos de red;
- arranque y apagado.

Los logs son fundamentales para diagnóstico y mantenimiento.

El problema es que, si un proceso genera mensajes continuamente, los archivos pueden crecer hasta consumir una parte importante del eMMC.

En un CM4 de 8 GB esto es especialmente relevante.

---

## 8.2. Revisar cuánto ocupa `/var/log`

```bash
sudo du -xh --max-depth=1 /var/log 2>/dev/null | sort -h
```

Para encontrar los archivos más grandes:

```bash
sudo du -xh /var/log/* 2>/dev/null | sort -h | tail -30
```

Ejemplo:

```text
730M    /var/log/journal
1,2G    /var/log/daemon.log
1,2G    /var/log/syslog
```

En este caso, el problema no es un archivo temporal: son logs que han crecido considerablemente.

---

# 9. Inspeccionar logs de texto

## 9.1. Ver las últimas líneas de `syslog`

```bash
sudo tail -100 /var/log/syslog
```

En tiempo real:

```bash
sudo tail -f /var/log/syslog
```

Salir con:

```text
Ctrl+C
```

---

## 9.2. Ver `daemon.log`

Últimas 100 líneas:

```bash
sudo tail -100 /var/log/daemon.log
```

En tiempo real:

```bash
sudo tail -f /var/log/daemon.log
```

---

## 9.3. Buscar errores

```bash
sudo grep -i "error" /var/log/syslog | tail -50
```

Buscar advertencias:

```bash
sudo grep -i "warning" /var/log/syslog | tail -50
```

Buscar un servicio específico:

```bash
sudo grep -i "ModemManager" /var/log/syslog | tail -100
```

```bash
sudo grep -i "NetworkManager" /var/log/syslog | tail -100
```

```bash
sudo grep -i "TELSY" /var/log/syslog | tail -100
```

---

# 10. Journal de systemd

Los sistemas modernos basados en systemd utilizan `journald`.

Consultar tamaño:

```bash
journalctl --disk-usage
```

Ver los últimos eventos:

```bash
sudo journalctl -n 100
```

Seguir eventos en tiempo real:

```bash
sudo journalctl -f
```

Ver eventos del arranque actual:

```bash
sudo journalctl -b
```

Ver errores del arranque actual:

```bash
sudo journalctl -b -p err
```

Ver advertencias y errores:

```bash
sudo journalctl -b -p warning
```

---

# 11. Inspeccionar logs de un servicio específico

## ModemManager

```bash
sudo journalctl -u ModemManager
```

Últimas 100 líneas:

```bash
sudo journalctl -u ModemManager -n 100
```

En tiempo real:

```bash
sudo journalctl -u ModemManager -f
```

Desde el arranque actual:

```bash
sudo journalctl -b -u ModemManager
```

## NetworkManager

```bash
sudo journalctl -u NetworkManager -n 100
```

En tiempo real:

```bash
sudo journalctl -u NetworkManager -f
```

## Un servicio TELSY

Si el servicio se llama, por ejemplo, `telsy-server.service`:

```bash
sudo journalctl -u telsy-server.service -n 100
```

En tiempo real:

```bash
sudo journalctl -u telsy-server.service -f
```

---

# 12. Saber qué servicios están activos

Lista general:

```bash
systemctl --type=service --state=running
```

Buscar servicios relacionados con TELSY:

```bash
systemctl --type=service | grep -Ei "telsy|django|driver"
```

Buscar servicios de red:

```bash
systemctl --type=service | grep -Ei "network|modem|dhcp"
```

Estado de un servicio:

```bash
systemctl status ModemManager --no-pager
```

```bash
systemctl status NetworkManager --no-pager
```

---

# 13. Identificar qué proceso genera un log

Una estrategia práctica es observar el log en tiempo real:

```bash
sudo tail -f /var/log/syslog
```

Si aparecen repetidamente mensajes del mismo servicio, investigar ese servicio.

Por ejemplo:

```bash
sudo journalctl -u ModemManager -f
```

o:

```bash
sudo journalctl -u NetworkManager -f
```

Para conocer procesos relacionados:

```bash
ps aux | grep -Ei "telsy|modem|network"
```

También:

```bash
systemctl list-units --type=service --state=running
```

### Regla importante

**No solucionar un log grande simplemente borrándolo.**

Primero determinar:

```text
¿Qué servicio escribe?
        ↓
¿Por qué escribe tanto?
        ↓
¿Es comportamiento normal?
        ↓
¿Existe un error repetitivo?
        ↓
¿Debe corregirse la causa?
        ↓
¿Se debe ajustar la política de retención?
```

---

# 14. Rotación de logs

Los sistemas Linux utilizan mecanismos de rotación para evitar que los logs crezcan indefinidamente.

Revisar configuración de `logrotate`:

```bash
ls -lh /etc/logrotate.d/
```

Configuración principal:

```bash
cat /etc/logrotate.conf
```

Buscar reglas relacionadas con `syslog`:

```bash
grep -Rni "syslog\|daemon.log" /etc/logrotate.conf /etc/logrotate.d/ 2>/dev/null
```

No modificar la configuración de rotación sin comprender primero qué servicio la utiliza.

---

# 15. Vaciar un log sin eliminar el archivo

Si se identifica un log excesivamente grande y se confirma que se puede vaciar:

```bash
sudo truncate -s 0 /var/log/syslog
```

Por ejemplo:

```bash
sudo truncate -s 0 /var/log/daemon.log
```

### ¿Por qué `truncate`?

El archivo permanece disponible para el proceso que lo utiliza, pero su tamaño se reduce a cero.

Es preferible a hacer:

```bash
sudo rm /var/log/syslog
```

sobre todo cuando un proceso puede tener el archivo abierto.

> **No vaciar un log sin investigar primero por qué creció.** Si la causa continúa, volverá a crecer.

---

# 16. Limpiar journals de systemd

Comprobar primero:

```bash
sudo journalctl --disk-usage
```

Eliminar journals archivados hasta dejar aproximadamente 100 MB:

```bash
sudo journalctl --vacuum-size=100M
```

Otra alternativa es conservar solamente un periodo:

```bash
sudo journalctl --vacuum-time=7d
```

Esto elimina journals archivados que sean anteriores al periodo indicado.

### Importante

Si `journalctl --vacuum-size=100M` informa:

```text
freed 0B
```

no necesariamente significa que haya un problema. Puede significar que no existen archivos de journal archivados que puedan eliminarse o que el journal actual no pueda reducirse mediante esa operación.

---

# 17. Logs rotados

Los logs antiguos suelen aparecer con nombres como:

```text
syslog.1
syslog.2.gz
daemon.log.1
daemon.log.2.gz
```

Consultar su tamaño:

```bash
sudo du -xh /var/log/* 2>/dev/null | sort -h
```

Los archivos `.gz` suelen ser logs comprimidos y forman parte de la rotación.

No borrar indiscriminadamente todos los archivos de `/var/log`.

---

# 18. ¿Qué se puede borrar normalmente?

## Generalmente seguro, después de comprobar

### Caché de APT

```bash
sudo apt clean
```

### Caché del usuario

```bash
rm -rf ~/.cache/*
```

### Temporales de `/tmp`

```bash
sudo rm -rf /tmp/*
```

### Temporales antiguos de `/var/tmp`

Solo después de comprobar que no estén siendo utilizados:

```bash
sudo rm -rf /var/tmp/*
```

### Logs excesivamente grandes

Después de identificar el servicio y decidir que es apropiado vaciarlos:

```bash
sudo truncate -s 0 /var/log/archivo.log
```

### Journals antiguos

```bash
sudo journalctl --vacuum-size=100M
```

---

# 19. ¿Qué NO se debe borrar manualmente?

Evitar comandos como:

```bash
sudo rm -rf /usr/*
sudo rm -rf /etc/*
sudo rm -rf /var/*
sudo rm -rf /lib/*
sudo rm -rf /bin/*
sudo rm -rf /sbin/*
```

También evitar eliminar manualmente:

- `/usr/bin`
- `/usr/lib`
- `/usr/share`
- `/etc`
- `/var/lib`
- `/lib`
- `/boot`

Estos directorios contienen componentes esenciales del sistema.

Si un paquete instalado ya no es necesario, utilizar el gestor de paquetes:

```bash
sudo apt remove paquete
```

o:

```bash
sudo apt purge paquete
```

y posteriormente, si corresponde:

```bash
sudo apt autoremove
```

---

# 20. Encontrar archivos antiguos

Para archivos modificados hace más de 30 días:

```bash
sudo find /home/pi -xdev -type f -mtime +30 -ls
```

Para archivos mayores de 100 MB:

```bash
sudo find /home/pi -xdev -type f -size +100M -ls
```

No eliminar automáticamente los resultados. La antigüedad o el tamaño no significan que el archivo sea innecesario.

---

# 21. Comprobar espacio después de una limpieza

Siempre verificar:

```bash
df -h /
```

También:

```bash
sudo du -xh --max-depth=1 / 2>/dev/null | sort -h
```

Si se necesita revisar `/var` nuevamente:

```bash
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h
```

---

# 22. Procedimiento recomendado para un CM4 de 8 GB

Cuando el almacenamiento esté bajo:

### Paso 1 — Medir

```bash
df -h /
```

### Paso 2 — Identificar directorios grandes

```bash
sudo du -xh --max-depth=1 / 2>/dev/null | sort -h
```

### Paso 3 — Si `/var` es grande

```bash
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h
```

### Paso 4 — Si `/var/log` es grande

```bash
sudo du -xh --max-depth=1 /var/log 2>/dev/null | sort -h
```

### Paso 5 — Identificar archivos grandes

```bash
sudo du -xh /var/log/* 2>/dev/null | sort -h | tail -30
```

### Paso 6 — Revisar el contenido

```bash
sudo tail -100 /var/log/syslog
```

```bash
sudo tail -100 /var/log/daemon.log
```

### Paso 7 — Revisar servicios

```bash
systemctl --type=service --state=running
```

### Paso 8 — Limpiar de forma controlada

```bash
sudo apt clean
rm -rf ~/.cache/*
sudo journalctl --vacuum-size=100M
```

Y, cuando corresponda, limpiar temporales:

```bash
sudo rm -rf /tmp/*
```

### Paso 9 — Verificar

```bash
df -h /
```

---

# 23. Comandos de diagnóstico rápido

## Espacio general

```bash
df -h /
```

## Directorios principales

```bash
sudo du -xh --max-depth=1 / 2>/dev/null | sort -h
```

## `/var`

```bash
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h
```

## `/var/log`

```bash
sudo du -xh --max-depth=1 /var/log 2>/dev/null | sort -h
```

## Archivos grandes

```bash
sudo find / -xdev -type f -size +100M -exec ls -lh {} \; 2>/dev/null
```

## Journal

```bash
sudo journalctl --disk-usage
```

## Servicios activos

```bash
systemctl --type=service --state=running
```

## Logs en tiempo real

```bash
sudo journalctl -f
```

```bash
sudo tail -f /var/log/syslog
```

---

# 24. Advertencia: `sudo: unable to resolve host TVSM01`

Si aparece:

```text
sudo: unable to resolve host TVSM01: Temporary failure in name resolution
```

normalmente significa que el nombre de host configurado en el sistema no coincide con la información disponible en `/etc/hosts`.

## 24.1. Comprobar el hostname

```bash
hostname
```

```bash
cat /etc/hostname
```

```bash
grep -n "127.0.1.1\|TVSM01" /etc/hosts
```

También:

```bash
hostnamectl
```

Si el hostname es:

```text
TVSM01
```

debe existir una resolución local coherente para ese nombre.

---

## 24.2. Revisar `/etc/hosts`

Abrir:

```bash
sudo nano /etc/hosts
```

Una configuración típica puede contener:

```text
127.0.0.1       localhost
127.0.1.1       TVSM01
```

Las demás entradas existentes deben conservarse.

Guardar en `nano`:

```text
Ctrl+O
Enter
Ctrl+X
```

---

## 24.3. Verificar

```bash
hostname
```

Después:

```bash
getent hosts TVSM01
```

Debería devolver una dirección local, por ejemplo:

```text
127.0.1.1       TVSM01
```

Finalmente:

```bash
sudo -v
```

Si la advertencia desapareció, la resolución local del hostname quedó corregida.

---

# 25. Recomendación específica para TELSY

En un CM4 que ejecuta TELSY, no conviene considerar los logs únicamente como archivos para borrar.

Si `syslog`, `daemon.log` o `journal` crecen rápidamente, primero investigar los servicios relacionados con:

```text
ModemManager
NetworkManager
TELSY
Django
GPS
systemd
```

Por ejemplo:

```bash
sudo journalctl -u ModemManager -n 100
```

```bash
sudo journalctl -u NetworkManager -n 100
```

Si TELSY tiene servicios systemd:

```bash
systemctl --type=service | grep -i telsy
```

y luego:

```bash
sudo journalctl -u NOMBRE_DEL_SERVICIO -n 100
```

Para observar el comportamiento mientras ocurre:

```bash
sudo journalctl -f
```

La prioridad debe ser:

**identificar la causa del crecimiento del log → corregir el proceso que genera mensajes repetitivos → controlar la retención/rotación → limpiar los archivos antiguos.**

Esto evita que el eMMC de 8 GB vuelva a alcanzar el 100 % de utilización.
