# Recuperación de Navegador Chromium: diagnóstico y solución 💻🌐
> [!NOTE]
> Este documento resume los síntomas, diagnóstico y cambios aplicados para restablecer el funcionamiento de Chromium en modo kiosco y su apertura normal desde el icono del escritorio o barra de tareas, además de las optimizaciones realizadas para un mejor funcionamiento y rendimiento.

## Contexto y síntomas iniciales
- Chromium se abría y se cerraba automáticamente al iniciar el servidor web.
- Al abrir desde TTY/SSH aparecía: `Missing X server or $DISPLAY`.
- Desde el icono del menú, barra de tareas o escritorio no abría (únicamente se observaba el cursor de “cargando” y nada más).
- En modo kiosco a veces pantalla en blanco al inicio o igualmente se abría y cerraba instantáneamente.
- El Touch screen o respuesta táctil funcionaba en otras apps, pero no en Chromium.
- Al prender el monitor (raspberry), el autologin dejó de aplicarse y la pantalla DSI no mostraba imagen (solo por HDMI).

## Causas identificadas
> [!WARNIG]
> 1. Base de datos SQLite de Django con permisos de solo lectura (`db.sqlite3` pertenecía a root), provocando errores en el backend y cierre inesperado del flujo.
> 2. Lanzamiento de Chromium desde entornos sin sesión gráfica (TTY/SSH) sin `DISPLAY`/`XAUTHORITY` configurados.
> 3. Locks de perfil de Chromium (archivos `Singleton*`) que impedían relanzar después de un cierre.
> 4. Parámetros/flags no adecuados para la sesión actual (Wayland/X11) y para dispositivos táctiles.
> 5. Arranque de Chromium antes de que el servidor web respondiera (pantalla en blanco y/o cierres).
> 6. Configuración de autologin y overlays de DSI no alineados con el entorno actual.

## Acciones realizadas (en orden lógico)
1. Corregidos permisos de la BD y aplicadas migraciones de Django.
   - Dueño de `db.sqlite3` → `pi:pi` y `python manage.py migrate`.
2. Limpieza de locks de Chromium cuando no abría tras cerrarse:
   - `rm -f ~/.config/chromium/Singleton*`.
3. Asegurar variables de entorno gráfico al lanzar desde scripts/TTY:
   - `export DISPLAY=:0` y `export XAUTHORITY=/home/pi/.Xauthority`.
4. Ajuste de flags para touch screen e integración con el panel:
   - Consolidado a X11 para mayor compatibilidad: `--ozone-platform=x11`.
   - Touch: `--touch-events=enabled`, `--enable-pinch`, `--enable-features=TouchEvents,PointerEvents`.
5. Evitar pantalla en blanco y arranques “en frío”:
   - Espera activa al backend antes de abrir el navegador con `curl`.
   - Desactivar GPU en kiosco si la inicialización crea artefactos: `--disable-gpu`.
   - Optimizar uso de memoria compartida: `--disable-dev-shm-usage`.
6. Restaurar apertura desde icono (limpieza de overrides/flags globales) y fallback temporal con `.desktop` local cuando fue necesario.
7. Autologin: habilitado `pam-autologin-service=lightdm-autologin` en `lightdm.conf`.
8. DSI: comentado overlays previos y añadidos overlays estándar (`vc4-kms-dsi-generic`, `vc4-kms-dpi-generic`).

## Script original vs script actual (¿Por qué cambió?)

Original:
```bash
#!/bin/bash

sleep 15
chromium-browser http://localhost:8000 \
  --check-for-update-interval=31536000 \
  --start-fullscreen \
  --kiosk \
  --noerrdialogs \
  --disable-translate \
  --no-first-run \
  --no-context-menu \
  --disable-context-menu \
  --fast \
  --fast-start \
  --disable-infobars \
  --overscroll-history-navigation=0 \
  --disable-pinch \
  --disable-session-crashed-bubble \
  --disable-sync \
  --disable-features=TouchpadOverscrollHistoryNavigation
```

Actual (`startweb.sh`):
```bash
#!/bin/bash
# Asegurar entorno gráfico cuando se ejecuta desde TTY/SSH
export DISPLAY=:0
export XAUTHORITY=/home/pi/.Xauthority

sleep 2
# Esperar a que el servidor responda
until curl -s http://127.0.0.1:8000/home/ >/dev/null; do
  sleep 2
done

chromium-browser http://localhost:8000/home/ \
  --check-for-update-interval=31536000 \
  --start-fullscreen \
  --kiosk \
  --noerrdialogs \
  --disable-translate \
  --no-first-run \
  --no-context-menu \
  --disable-context-menu \
  --fast \
  --fast-start \
  --disable-infobars \
  --overscroll-history-navigation=0 \
  --disable-session-crashed-bubble \
  --disable-sync \
  --disable-features=TouchpadOverscrollHistoryNavigation \
  --enable-features=TouchEvents,PointerEvents \
  --ozone-platform=x11 \
  --touch-events=enabled \
  --enable-pinch \
  --disable-gpu \
  --disable-software-rasterizer \
  --disable-dev-shm-usage
```

> [!IMPORTANT]
> Justificación de los cambios clave:
> - `export DISPLAY`/`XAUTHORITY`: garantiza que Chromium se conecte al servidor gráfico de la sesión cuando se llama desde servicios/TTY/SSH.
> - Espera con `curl`: evita abrir Chromium antes de que la app esté disponible, reduciendo pantallas en blanco y reintentos.
> - `--ozone-platform=x11`: en Raspberry Pi OS (LXDE/Wayfire con Xwayland), X11 ofrece mejor integración con la barra de tareas y estabilidad del input.
> - Flags táctiles: reactivan gestos y eventos de toque necesarios para kiosco táctil.
> - `--disable-gpu`: mitiga problemas de inicialización de GPU que causan pantalla en blanco o cierres en algunos drivers.
> - `--disable-dev-shm-usage`: reduce bloqueos por límites de memoria compartida en entornos con `/dev/shm` pequeño.

## Recomendaciones de uso
> [!TIP]
> - Lanzar kiosco: `./startweb.sh` (espera al backend y abre estable).
> - Si no abre tras un cierre forzado: `pkill -f chromium && rm -f ~/.config/chromium/Singleton*` y volver a lanzar.
> - Para depurar: revisar `/tmp/*.log` si se ejecuta el script redirigiendo salida.

## Autologin
> [!TIP]
> - Configurado en `/etc/lightdm/lightdm.conf`:
>   - `autologin-user=pi`
>   - `autologin-session=LXDE-pi-wayland`
>   - `pam-autologin-service=lightdm-autologin`
> - Si sigue pidiendo clave tras reboot: `sudo systemctl restart lightdm` o reiniciar.

## Pantalla o Display DSI
> [!TIP]
> - En `/boot/config.txt` se comentaron overlays específicos y se añadieron genéricos:
>   - `#dtoverlay=vc4-dsi`, `#dtoverlay=vc4-dsi-ts`
>   - `dtoverlay=vc4-kms-dsi-generic`
>   - `dtoverlay=vc4-kms-dpi-generic`
> - Requiere reinicio para aplicar. Si no hay imagen, probar uno a uno o volver a `display_auto_detect=1` + overlay del proveedor.

## Notas y troubleshooting (Proceso sitemático para diagnosticar y resolver problemas)
> [!IMPORTANT]
> - “Missing X server or $DISPLAY”: falta sesión gráfica o variables; usar las export del script.
> - Touch no responde: probar X11 (actual) y verificar pertenencia a grupo `input` (`id pi`).
> - Icono que no aparece en barra (Wayland): habilitar decoraciones en chrome://flags; en X11 activar “usar barra de título y bordes del sistema”.
> - Si el icono del menú no abre: limpiar `Singleton*` y verificar que no existan overrides en `~/.local/share/applications`.

## Estado final
- Chromium funciona modo en kiosco y desde el icono en el menú en barra de tareas.
- Touch scrren habilitado y rendimiento mejorado.
- Autologin y pantalla o display DSI ajustados; se requiere **reinicio** tras cambios en `/boot/config.txt`.

---

> [!NOTE]
> **Fecha de creación**: Agosto 2025  
> **Sistema**: Problemas al iniciar el Navegador Chromium en modo normal y en modo kiosko, también fallaba el 'Autologin'.
> **Problema resuelto**:  Inicio del Navegador Chromium y 'Autologin' corregidos
> **Estado**: ✅ FUNCIONANDO
> 
> **Made by:** @dagdmfc
> 