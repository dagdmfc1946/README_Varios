# Guía de diagnóstico de audio --- Telsy / Raspberry Pi CM4 / WM8904

**Proyecto:** Telsy\
**Plataforma evaluada:** Raspberry Pi Compute Module 4 (CM4) sobre
tarjeta principal del proyecto\
**Codec de audio:** Wolfson/Cirrus Logic WM8904\
**Sistema de audio:** ALSA + PulseAudio + Chromium/WebRTC\
**Aplicación Telsy:** `/home/pi/telsy-monitor`\
**Fecha de documentación:** 25 de septiembre de 2026

------------------------------------------------------------------------

## 1. Objetivo

Este documento consolida el procedimiento utilizado para diagnosticar un
fallo de audio en Telsy, especialmente el caso en el que:

-   el micrófono y la cámara funcionan;
-   la videollamada se ejecuta correctamente en Chromium;
-   el audio remoto no se escucha;
-   el hardware de audio WM8904 sí puede funcionar correctamente después
    de un reinicio.

La guía pretende servir tanto como **bitácora del caso investigado**
como procedimiento reutilizable para futuras unidades Telsy.

> **Importante:** algunos hallazgos son concluyentes y otros continúan
> en investigación. Las hipótesis pendientes se identifican
> explícitamente para evitar tratarlas como causas confirmadas.

------------------------------------------------------------------------

## 2. Arquitectura de audio relevante

La ruta simplificada de reproducción es:

``` text
Videollamada WebRTC
        │
        ▼
     Chromium
        │
        ▼
    PulseAudio
        │
        ▼
       ALSA
        │
        ▼
 BCM2835 I²S
        │
        ▼
      WM8904
        │
        ▼
Salida analógica / parlante
```

Esto permite dividir el diagnóstico por capas en lugar de asumir
inicialmente que el problema está en Chromium, el codec o el hardware.

------------------------------------------------------------------------

## 3. Hardware ALSA identificado

El comando:

``` bash
aplay -l
```

identificó el WM8904 como dispositivo de reproducción:

``` text
card 0: wm8904soundcard [wm8904-soundcard]
device 0: bcm2835-i2s-wm8904-hifi wm8904-hifi-0
```

También se encontraron dispositivos HDMI independientes.

La consulta:

``` bash
cat /proc/asound/cards
```

confirmó la presencia de la tarjeta WM8904.

### Comprobación I²C

Se utilizó:

``` bash
i2cdetect -y 1
```

La dirección correspondiente al codec apareció como `UU`.

En este contexto, `UU` es consistente con un dispositivo que ya está
siendo utilizado por un controlador del kernel; no debe interpretarse
automáticamente como una falla.

------------------------------------------------------------------------

## 4. Pruebas funcionales básicas de ALSA

### 4.1 Tono estéreo directo al hardware

Uno de los comandos más útiles para comprobar la cadena ALSA → I²S →
WM8904 es:

``` bash
speaker-test -D hw:0,0 -c 2 -r 48000 -t sine -f 1000
```

Parámetros:

-   `-D hw:0,0`: tarjeta 0, dispositivo 0, acceso directo al hardware.
-   `-c 2`: dos canales.
-   `-r 48000`: frecuencia de muestreo de 48 kHz.
-   `-t sine`: señal sinusoidal.
-   `-f 1000`: tono de 1 kHz.

Cuando el sistema se encuentra en un estado correcto, este comando
**produce sonido**.

### 4.2 Archivo WAV mediante ALSA

También funcionó:

``` bash
aplay /usr/share/sounds/alsa/Front_Center.wav
```

y:

``` bash
aplay -D plughw:0,0 /usr/share/sounds/alsa/Front_Center.wav
```

### 4.3 Diferencia entre `hw` y `plughw`

El siguiente comando falló:

``` bash
aplay -D hw:0,0 /usr/share/sounds/alsa/Front_Center.wav
```

con un error relacionado con el número de canales.

La razón es que `Front_Center.wav` es mono, mientras la interfaz
hardware utilizada requiere dos canales.

`plughw` permite que ALSA realice las conversiones necesarias:

``` bash
aplay -D plughw:0,0 /usr/share/sounds/alsa/Front_Center.wav
```

Por tanto, un error de canales con `hw` **no implica una falla del
codec**.

------------------------------------------------------------------------

## 5. Prueba de PulseAudio

Para comprobar la ruta que utiliza Chromium se puede reproducir el mismo
archivo mediante PulseAudio:

``` bash
paplay /usr/share/sounds/alsa/Front_Center.wav
```

Después de un reinicio limpio, esta prueba produjo sonido correctamente.

Esto demuestra que, en un estado sano:

``` text
PulseAudio → ALSA → WM8904
```

también funciona.

------------------------------------------------------------------------

## 6. Verificación de los sinks de PulseAudio

Comando:

``` bash
pactl list short sinks
```

En el equipo se identificó:

``` text
alsa_output.platform-soc_sound.multichannel-output
```

Después de un reinicio y antes de reproducir la videollamada, el sink
funcionó a:

``` text
s16le 2ch 48000Hz
```

Durante la videollamada se observó posteriormente:

``` text
s16le 2ch 44100Hz RUNNING
```

Este cambio de frecuencia es un hallazgo importante, pero **todavía no
se considera la causa confirmada del problema**.

------------------------------------------------------------------------

## 7. Verificación del audio generado por Chromium/WebRTC

Durante una videollamada activa se ejecutó:

``` bash
pactl list short sink-inputs
```

y se observó un stream de reproducción:

``` text
float32le 2ch 44100Hz
```

Al mismo tiempo:

``` bash
pactl list short sinks
```

mostró el sink como:

``` text
RUNNING
```

### Interpretación

Esto permitió descartar varias hipótesis iniciales:

-   Chromium sí estaba generando audio.
-   PulseAudio sí estaba recibiendo el stream.
-   El stream estaba dirigido al sink disponible.
-   El sink estaba activo.
-   No parecía tratarse simplemente de autoplay bloqueado.
-   No parecía tratarse de que Chromium estuviera enviando el audio a
    HDMI.

A pesar de ello, físicamente no se escuchaba sonido.

------------------------------------------------------------------------

## 8. Error `Device or resource busy`

Al intentar:

``` bash
speaker-test -D hw:0,0 -c 2 -r 44100 -t sine -f 1000
```

durante la videollamada se obtuvo:

``` text
Playback open error: -16, Dispositivo o recurso ocupado
```

### Interpretación correcta

Este mensaje **no demuestra que 44,1 kHz sea incompatible**.

Significa que otro proceso tenía abierto el dispositivo hardware y
`speaker-test`, al utilizar `hw:0,0`, no podía acceder simultáneamente a
él.

Para identificar quién está utilizando los dispositivos de sonido:

``` bash
sudo fuser -v /dev/snd/*
```

En el caso investigado apareció PulseAudio.

------------------------------------------------------------------------

## 9. Detener temporalmente PulseAudio

Se utilizó:

``` bash
pulseaudio -k
sleep 2
sudo fuser -v /dev/snd/*
```

Si `fuser` ya no muestra PulseAudio, el dispositivo está disponible para
una prueba ALSA directa.

Sin embargo, durante el estado defectuoso, incluso después de detener
PulseAudio:

``` bash
speaker-test -D hw:0,0 -c 2 -r 48000 -t sine -f 1000
```

se ejecutó sin errores pero **no produjo sonido**.

Este resultado fue especialmente importante: el problema podía persistir
después de que PulseAudio dejara de utilizar el dispositivo.

------------------------------------------------------------------------

## 10. Verificación del mixer WM8904

### Headphone

``` bash
amixer -c 0 sget Headphone
```

Estado observado durante el fallo:

``` text
Front Left:  100% 6.00dB Playback [on]
Front Right: 100% 6.00dB Playback [on]
```

### Line Output

``` bash
amixer -c 0 sget 'Line Output'
```

También:

``` text
100% 6.00dB Playback [on]
```

Por tanto, el silencio no se explicó por un mute básico de Headphone o
Line Output.

> Los niveles utilizados durante el diagnóstico son muy altos. Para
> operación normal debe evaluarse una ganancia menor para evitar
> saturación o clipping.

------------------------------------------------------------------------

## 11. Verificación de los MUX de salida

Se utilizó:

``` bash
amixer -c 0 | grep -A3 -E "HPL Mux|HPR Mux|LINEL Mux|LINER Mux"
```

Resultado:

``` text
HPL Mux   → DAC
HPR Mux   → DAC
LINEL Mux → DAC
LINER Mux → DAC
```

Esto confirmó que las salidas analógicas continuaban recibiendo la ruta
del DAC.

------------------------------------------------------------------------

## 12. Controles ALSA disponibles

Para evitar asumir nombres de controles inexistentes:

``` bash
amixer -c 0 scontrols
```

Entre los controles relevantes encontrados estuvieron:

``` text
Headphone
Line Output
DACL Mux
DACR Mux
Digital
Digital Playback Boost
HPL Mux
HPR Mux
LINEL Mux
LINER Mux
```

No existe un control denominado literalmente:

``` text
Digital Playback
```

en esta implementación.

------------------------------------------------------------------------

## 13. Nivel digital y MUX del DAC

### Digital

``` bash
amixer -c 0 sget 'Digital'
```

Durante el fallo:

``` text
Playback 100% [0.00dB]
```

### DACL

``` bash
amixer -c 0 sget 'DACL Mux'
```

Resultado:

``` text
Left
```

### DACR

``` bash
amixer -c 0 sget 'DACR Mux'
```

Resultado:

``` text
Right
```

Así, la ruta visible mediante ALSA permanecía coherente:

``` text
Digital L/R
    ↓
DACL / DACR
    ↓
HPL/HPR o LINEL/LINER
    ↓
Salida analógica
```

A pesar de ello, no había sonido.

------------------------------------------------------------------------

## 14. Prueba decisiva mediante reinicio

Se realizó:

``` bash
sudo reboot
```

Después del reinicio, antes de iniciar Telsy/Chromium, se ejecutó:

``` bash
speaker-test -D hw:0,0 -c 2 -r 48000 -t sine -f 1000
```

y **el sonido volvió a funcionar**.

Posteriormente:

``` bash
paplay /usr/share/sounds/alsa/Front_Center.wav
```

también produjo sonido y PulseAudio mostraba el sink a:

``` text
48000Hz
```

### Conclusión confirmada

El hardware de reproducción puede funcionar correctamente.

El fallo observado no corresponde, por sí solo, a evidencia de:

-   parlante averiado;
-   WM8904 permanentemente averiado;
-   I²S permanentemente averiado;
-   ALSA completamente defectuoso;
-   PulseAudio completamente defectuoso.

El problema aparece asociado a una **transición de estado durante el
funcionamiento del sistema**, y el reinicio restaura el funcionamiento.

------------------------------------------------------------------------

## 15. Hipótesis actual: 48 kHz frente a 44,1 kHz

Se observó esta secuencia:

### Estado sano

``` text
PulseAudio sink → 48000 Hz
paplay          → sonido correcto
```

### Durante la videollamada problemática

``` text
Chromium/WebRTC input → 44100 Hz
PulseAudio sink       → 44100 Hz RUNNING
Salida física         → silencio
```

Por esta razón existe una hipótesis de trabajo:

> El cambio de frecuencia de 48 kHz a 44,1 kHz podría provocar una
> configuración incorrecta o un estado anómalo en la ruta I²S/WM8904.

**Esta hipótesis todavía debe confirmarse.**

No debe documentarse todavía como causa raíz.

------------------------------------------------------------------------

## 16. Prueba pendiente para confirmar o descartar la hipótesis

Es necesario iniciar el equipo sin que Telsy/Chromium abra
automáticamente y, partiendo de un estado conocido bueno, probar
directamente:

### 44,1 kHz

``` bash
pulseaudio -k
sleep 2
speaker-test -D hw:0,0 -c 2 -r 44100 -t sine -f 1000
```

### 48 kHz

``` bash
speaker-test -D hw:0,0 -c 2 -r 48000 -t sine -f 1000
```

Los resultados deben registrarse.

Especialmente importante:

-   ¿44,1 kHz produce sonido?
-   ¿48 kHz continúa produciendo sonido después de probar 44,1 kHz?
-   ¿probar 44,1 kHz deja posteriormente el codec en silencio?

Esto permitiría comprobar si el cambio de frecuencia reproduce el mismo
fallo sin necesidad de Chromium.

------------------------------------------------------------------------

## 17. Inicio automático de Telsy

El script identificado es:

``` text
/home/pi/telsy-monitor/startweb.sh
```

Su función incluye esperar al servidor local:

``` text
http://127.0.0.1:8000/home/
```

y posteriormente iniciar Chromium en modo kiosk.

Para localizar qué mecanismo ejecuta automáticamente el script:

``` bash
grep -R "startweb.sh" ~/.config/autostart ~/.config/lxsession /etc/xdg/autostart /etc/systemd/system /etc/rc.local 2>/dev/null
```

Antes de modificar el autoinicio se recomienda identificar exactamente
el archivo o servicio responsable.

**No eliminar `startweb.sh`.** El objetivo del diagnóstico es desactivar
temporalmente el autoinicio para poder realizar pruebas controladas y
luego restaurarlo.

------------------------------------------------------------------------

## 18. Secuencia recomendada de diagnóstico

Cuando una unidad Telsy presente "no se escucha", seguir este orden:

### Paso 1 --- Confirmar que ALSA reconoce el hardware

``` bash
aplay -l
```

Debe aparecer el WM8904.

### Paso 2 --- Identificar tarjetas

``` bash
cat /proc/asound/cards
```

### Paso 3 --- Probar ALSA directo

``` bash
speaker-test -D hw:0,0 -c 2 -r 48000 -t sine -f 1000
```

Si suena, la cadena hardware básica funciona.

### Paso 4 --- Probar conversión ALSA

``` bash
aplay -D plughw:0,0 /usr/share/sounds/alsa/Front_Center.wav
```

### Paso 5 --- Probar PulseAudio

``` bash
paplay /usr/share/sounds/alsa/Front_Center.wav
```

### Paso 6 --- Revisar sink

``` bash
pactl list short sinks
```

Registrar:

-   nombre;
-   estado (`SUSPENDED`, `IDLE`, `RUNNING`);
-   canales;
-   frecuencia de muestreo.

### Paso 7 --- Durante una videollamada, revisar stream

``` bash
pactl list short sink-inputs
```

Si existe un sink-input mientras la persona remota habla, Chromium está
entregando audio a PulseAudio.

### Paso 8 --- Revisar quién utiliza ALSA

``` bash
sudo fuser -v /dev/snd/*
```

### Paso 9 --- Revisar mixer

``` bash
amixer -c 0 sget Headphone
amixer -c 0 sget 'Line Output'
amixer -c 0 sget 'Digital'
amixer -c 0 sget 'DACL Mux'
amixer -c 0 sget 'DACR Mux'
```

### Paso 10 --- Revisar MUX analógicos

``` bash
amixer -c 0 | grep -A3 -E "HPL Mux|HPR Mux|LINEL Mux|LINER Mux"
```

### Paso 11 --- Si el estado parece correcto pero continúa en silencio

Reiniciar:

``` bash
sudo reboot
```

y repetir inmediatamente:

``` bash
speaker-test -D hw:0,0 -c 2 -r 48000 -t sine -f 1000
```

Si después del reinicio vuelve el audio, existe evidencia de un problema
de **estado/configuración durante la ejecución**, no necesariamente de
una falla física.

------------------------------------------------------------------------

## 19. Matriz rápida de interpretación

  -----------------------------------------------------------------------------------
  Síntoma                     Posible explicación        Acción
  --------------------------- -------------------------- ----------------------------
  WM8904 no aparece en        Driver, Device Tree, I²S o Revisar kernel/DT antes de
  `aplay -l`                  inicialización             investigar Chromium

  `speaker-test` directo      Hardware + ALSA + I²S +    Investigar capas superiores
  funciona                    codec operativos           

  `paplay` funciona           PulseAudio y salida al     Investigar
                              codec operativos           aplicación/Chromium/WebRTC

  `sink-input` de Chromium no Chromium/WebRTC no está    Revisar aplicación/browser
  aparece                     entregando reproducción    

  `sink-input` aparece y sink PulseAudio recibe audio    Revisar salida/configuración
  está `RUNNING`                                         posterior

  `Device or resource busy`   Otro proceso tiene abierto Usar `fuser`; no concluir
  con `hw:0,0`                el dispositivo             falla de frecuencia

  Mixer muestra salida        Mute ALSA                  Activar el control
  `[off]`                                                correspondiente

  HPL/HPR no apuntan a DAC    Enrutamiento incorrecto    Corregir MUX

  Todo parece correcto y      Estado anómalo de          Investigar transición que
  reboot recupera audio       software/codec/driver      dispara el fallo

  Sink cambia 48 kHz → 44,1   Posible problema de        Confirmar experimentalmente
  kHz y aparece silencio      clock/rate/configuración   antes de modificar
                                                         configuración
  -----------------------------------------------------------------------------------

------------------------------------------------------------------------

## 20. Posibles soluciones según la causa

### A. Dispositivo ocupado

Identificar el proceso:

``` bash
sudo fuser -v /dev/snd/*
```

No matar procesos indiscriminadamente; determinar primero si corresponde
a PulseAudio, Chromium u otro servicio.

### B. Mixer silenciado

Verificar:

``` bash
amixer -c 0 sget Headphone
amixer -c 0 sget 'Line Output'
```

La solución depende del control encontrado en `off`.

### C. MUX incorrecto

Comprobar que las salidas esperadas estén conectadas al DAC.

### D. Frecuencia de muestreo

**Solo si se confirma experimentalmente** que 44,1 kHz provoca el fallo,
una posible solución será mantener PulseAudio a 48 kHz y permitir el
resampling de streams que lleguen a 44,1 kHz.

Antes de aplicar esa solución debe comprobarse la configuración actual
de PulseAudio y documentar el resultado de las pruebas 44,1/48 kHz.

### E. Estado anómalo persistente del codec

Si el codec queda silencioso incluso después de cerrar Chromium y
detener PulseAudio, pero un reboot lo recupera, puede ser necesario
investigar:

-   gestión de energía del codec;
-   secuencia de clocking;
-   comportamiento del machine driver;
-   reconfiguración del I²S;
-   transición de sample rate;
-   registros internos del WM8904;
-   secuencia de suspend/resume.

El reinicio es una medida de recuperación, pero **no constituye la
solución raíz**.

------------------------------------------------------------------------

## 21. Errores de diagnóstico que deben evitarse

1.  **Concluir que el parlante o codec están dañados únicamente porque
    Telsy no suena.**\
    Primero realizar `speaker-test`.

2.  **Interpretar `Device or resource busy` como incompatibilidad con
    44,1 kHz.**\
    El error indica que el dispositivo está ocupado.

3.  **Usar `hw:0,0` con un WAV mono y concluir que ALSA está roto.**\
    Utilizar `plughw` cuando sea necesaria conversión.

4.  **Culpar a Chromium cuando existe el mismo silencio con `paplay`.**\
    `paplay` permite separar Chromium de PulseAudio.

5.  **Culpar a PulseAudio únicamente porque está ejecutándose.**\
    Después de un reboot se comprobó que PulseAudio a 48 kHz reproduce
    correctamente.

6.  **Modificar Device Tree, I²C o drivers antes de probar el hardware
    con ALSA.**\
    En este caso el hardware ya demostró funcionar.

7.  **Cambiar múltiples parámetros simultáneamente.**\
    Se pierde la posibilidad de identificar qué cambio corrige o
    reproduce el fallo.

------------------------------------------------------------------------

## 22. Estado actual del caso

### Confirmado

-   El WM8904 es reconocido por ALSA.
-   El audio por I²S puede funcionar.
-   `speaker-test` a 48 kHz produce sonido después de un reinicio.
-   `aplay`/`plughw` pueden producir sonido.
-   PulseAudio puede reproducir correctamente después de un reinicio.
-   Chromium genera un stream durante la videollamada.
-   El sink de PulseAudio entra en estado `RUNNING`.
-   Durante el fallo, los controles visibles de volumen y MUX
    permanecieron correctamente configurados.
-   El fallo puede persistir después de cerrar Chromium y detener
    PulseAudio.
-   Reiniciar la Raspberry restaura el audio.

### Observado pero no confirmado como causa

-   En estado sano se observó PulseAudio a 48 kHz.
-   Durante la videollamada problemática se observó un stream/sink a
    44,1 kHz.
-   Existe una correlación entre el cambio de frecuencia y la aparición
    del silencio.

### Pendiente

-   Probar 44,1 kHz directamente desde un estado sano y con el hardware
    libre.
-   Determinar si 44,1 kHz por sí solo reproduce el fallo.
-   Determinar si volver de 44,1 a 48 kHz recupera el audio sin reboot.
-   Identificar el mecanismo exacto de autoinicio de `startweb.sh`.
-   Si se confirma el problema de frecuencia, probar PulseAudio
    bloqueado a 48 kHz.
-   Si la frecuencia no es la causa, investigar estado interno/power
    management/clocking del WM8904.

------------------------------------------------------------------------

## 23. Registro recomendado para futuras pruebas

Para cada prueba registrar:

``` text
Fecha/hora:
Estado inicial: recién iniciado / Telsy activo / llamada activa
Chromium: sí/no
PulseAudio: sí/no
Frecuencia sink:
Estado sink:
Comando ejecutado:
Resultado del comando:
¿Se escuchó audio?: sí/no
¿El audio siguió funcionando después?: sí/no
Observaciones:
```

Esto permite identificar exactamente qué transición provoca el fallo.

------------------------------------------------------------------------

## 24. Comandos de referencia rápida

``` bash
# Listar hardware ALSA
aplay -l

# Listar tarjetas
cat /proc/asound/cards

# Tono directo WM8904 a 48 kHz
speaker-test -D hw:0,0 -c 2 -r 48000 -t sine -f 1000

# Tono directo a 44,1 kHz
speaker-test -D hw:0,0 -c 2 -r 44100 -t sine -f 1000

# WAV mediante conversión ALSA
aplay -D plughw:0,0 /usr/share/sounds/alsa/Front_Center.wav

# WAV mediante PulseAudio
paplay /usr/share/sounds/alsa/Front_Center.wav

# Sinks PulseAudio
pactl list short sinks

# Streams de reproducción PulseAudio
pactl list short sink-inputs

# Procesos utilizando dispositivos de audio
sudo fuser -v /dev/snd/*

# Controles disponibles del mixer
amixer -c 0 scontrols

# Controles principales
amixer -c 0 sget Headphone
amixer -c 0 sget 'Line Output'
amixer -c 0 sget 'Digital'
amixer -c 0 sget 'DACL Mux'
amixer -c 0 sget 'DACR Mux'

# MUX de salidas
amixer -c 0 | grep -A3 -E "HPL Mux|HPR Mux|LINEL Mux|LINER Mux"

# Detener PulseAudio temporalmente
pulseaudio -k

# Reiniciar
sudo reboot
```

------------------------------------------------------------------------

## 25. Principio general de diagnóstico

La estrategia más útil en este caso ha sido **aislar las capas una por
una**:

``` text
¿ALSA directo suena?
       │
       ├── NO → investigar hardware / codec / driver / estado
       │
       └── SÍ
            │
            ▼
      ¿PulseAudio suena?
            │
            ├── NO → investigar PulseAudio / configuración / estado
            │
            └── SÍ
                 │
                 ▼
        ¿Chromium genera stream?
                 │
                 ├── NO → investigar aplicación/WebRTC
                 │
                 └── SÍ → investigar transición/routing/formato
```

Este enfoque evita modificar simultáneamente hardware, ALSA, PulseAudio
y Chromium y permite obtener evidencia reproducible antes de aplicar una
solución.

------------------------------------------------------------------------

**Estado del documento:** diagnóstico en curso.\
**Último estado conocido bueno:** después de reiniciar, ALSA directo y
`paplay` funcionan a 48 kHz.\
**Próxima prueba crítica:** reproducción directa a 44,1 kHz desde un
arranque limpio, evitando el autoinicio de Telsy/Chromium.
