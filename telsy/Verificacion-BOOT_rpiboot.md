# Etapa A --- Verificación inicial

## 1. Estado inicial

Antes de comenzar:

  ElementoEstado          
  ----------------------- ------------------
  CM4                     Apagada
  Banco de baterías       **DESCONECTADO**
  USB PC → Main Board     Desconectado
  `BOOT_OPTION_SW`        Liberado
  Otros USB/periféricos   Desconectados

El hecho de que la Main Board se encienda al conectar USB significa que
**el USB está suministrando alimentación a la placa**. Eso es
precisamente lo que debemos aprovechar inicialmente.

------------------------------------------------------------------------

## 2. Conecta el USB al PC

Conecta:

    PC
     │
     │ USB
     ▼
    Main Board
     │
     ▼
    CM4

La Main Board y la CM4 deberían encenderse.

**No conectes las baterías.**

------------------------------------------------------------------------

## 3. Ahora presiona `BOOT_OPTION_SW`

Como verificaste físicamente que:

    CM4 Pin 93
         │
         │ nRPIBOOT
         │
         ├──── BOOT_OPTION_SW ──── GND

al presionarlo obtenemos:

    nRPIBOOT = LOW

Esto es justamente lo que requiere la ROM de la CM4 para entrar en
`usbboot`.

### Pero hay una cuestión práctica

Como el USB **ya encendió la CM4 antes de presionar el botón**, no
quiero que hagamos simplemente:

    USB → CM4 arranca → presionar botón

porque eso puede ser demasiado tarde.

Lo que necesitamos es que el botón esté presionado **en el momento en
que la CM4 recibe alimentación y comienza el boot**.

Por tanto, la primera prueba debe hacerse así:

------------------------------------------------------------------------

# 4. Secuencia correcta con tu Main Board

Como el USB es el que alimenta la placa, necesitamos utilizar la
conexión USB como evento de encendido.

### Secuencia:

**1.** Desconecta el USB del PC.

La CM4 debe quedar sin alimentación USB.

**2.** Asegúrate de que la batería continúa desconectada.

**3.** Presiona y mantén presionado:

    BOOT_OPTION_SW

**4.** Sin soltar el botón, conecta el USB al PC.

Ahora tenemos:

                     PRESIONADO
                         │
                         ▼
    PC USB ───────► Main Board ─────► CM4
                         │
                         │
                    nRPIBOOT = LOW
                         │
                         ▼
                  ROM USB BOOT

**5.** Mantén el botón presionado.

**6.** En el PC ejecuta `rpiboot` con verbose:

    & "C:\Program Files (x86)\Raspberry Pipiboot.exe" -v

Idealmente, incluso mejor:

### Primero ejecutamos `rpiboot` y después hacemos la conexión USB.

Como `rpiboot` necesita estar esperando al dispositivo, recomiendo esta
secuencia final:

    1. USB desconectado
    2. Batería desconectada
    3. BOOT_OPTION_SW liberado

    4. Ejecutar rpiboot -v

    5. Esperar:
       Waiting for BCM2835/6/7/2711/2712...

    6. Presionar BOOT_OPTION_SW
    7. Conectar USB al PC
    8. Mantener BOOT_OPTION_SW
    9. Observar rpiboot
    10. Liberar BOOT_OPTION_SW cuando confirmemos la detección

Esta secuencia es coherente con la indicación de Raspberry Pi de que
para CM4 `nRPIBOOT` debe estar aplicado para cambiar la ROM a `usbboot`.

------------------------------------------------------------------------

# 5. ¿Cuándo suelto el botón?

Aquí quiero hacer una pequeña modificación respecto a lo que te indiqué
antes.

**En la primera prueba no lo sueltes inmediatamente.**

Quiero que me muestres primero la salida de `rpiboot`.

La razón es que estamos trabajando con una Main Board de terceros y
quiero comprobar exactamente en qué etapa está la CM4.

Si vemos algo como:

    Waiting for BCM2835/6/7/2711/2712...

y posteriormente:

    Device found

o mensajes relacionados con:

    BCM2711
    bootcode4.bin

entonces sabemos que la ROM respondió.

A partir de ahí podemos liberar el pulsador.

------------------------------------------------------------------------

# 6. ¿Qué debería ocurrir?

Si todo está correcto, `rpiboot` debería cargar el software de
`mass-storage-gadget64`.

La función de este software es:

    CM4
     │
     ▼
    ROM USB Boot
     │
     ▼
    rpiboot
     │
     ▼
    mass-storage-gadget64
     │
     ▼
    Linux initramfs
     │
     ▼
    Detecta eMMC
     │
     ▼
    USB Mass Storage
     │
     ▼
    Windows

Raspberry Pi indica que `mass-storage-gadget64` escanea dispositivos de
almacenamiento como eMMC y los expone mediante USB Mass Storage.

------------------------------------------------------------------------

# 7. En Windows verificaremos el eMMC

Una vez que `rpiboot` haya terminado su proceso, abre **PowerShell como
administrador** y ejecuta:

    Get-Disk | Format-Table Number, FriendlyName, SerialNumber, BusType, OperationalStatus, Size -AutoSize

Después:

    Get-PhysicalDisk | Format-Table DeviceId, FriendlyName, SerialNumber, BusType, Size, HealthStatus -AutoSize

Y:

    Get-Volume | Format-Table DriveLetter, FileSystemLabel, FileSystem, Size, SizeRemaining -AutoSize

No vamos a modificar absolutamente nada.

------------------------------------------------------------------------

# 8. Si Windows muestra "Formatear"

Puede ocurrir que Windows detecte el eMMC pero no pueda interpretar
alguna de sus particiones.

Si aparece:

> "Debe formatear el disco para poder utilizarlo."

selecciona:

**Cancelar.**

No:

-   Formatear.
-   Inicializar disco.
-   Crear partición.
-   Reparar.
-   Convertir GPT/MBR.
-   `chkdsk`.

El objetivo ahora es solamente **leer/detectar**.

------------------------------------------------------------------------

# Una comprobación adicional muy importante

También quiero que ejecutes:

    Get-Disk | Where-Object BusType -eq "USB" |
        Format-Table Number, FriendlyName, SerialNumber, BusType, OperationalStatus, Size -AutoSize

Esto nos permitirá identificar los discos USB que Windows ve.

Como tu PC tiene múltiples dispositivos USB, **no debemos asumir que el
primer disco que aparezca es el eMMC**.
