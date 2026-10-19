# TELSY · Historial técnico de modificaciones y mejoras

**Fecha de elaboración:** 17 de octubre de 2026, America/Bogota.  
**Periodo reconstruido:** trabajo registrado entre el 5 y el 16 de octubre de 2026.  
**Proyecto:** TELSY / Telsy-Hogar, Raspberry Pi y módulo multiparamétrico Berry PM6750.  
**Responsable del proyecto y pruebas de usuario:** Diego Andrés García Díaz.  
**Asistencia de diagnóstico, implementación y documentación:** Codex.  
**Estado del documento:** consolidación histórica; no constituye una nueva versión instalada del aplicativo.

> Este archivo se elaboró sin conectarse por SSH, sin consultar el dispositivo en vivo, sin reiniciar servicios y sin modificar el código de ejecución. Se contrastaron la conversación, los informes, las copias originales, los archivos de implementación, los instaladores y los registros locales. «Instalado» significa instalado según la última evidencia conservada; no significa comprobado nuevamente el 17 de octubre.

## Índice

1. [Objetivo y alcance](#1-objetivo-y-alcance)
2. [Resumen de lo obtenido](#2-resumen-de-lo-obtenido)
3. [Entorno y arquitectura](#3-entorno-y-arquitectura)
4. [Cronología de las intervenciones](#4-cronología-de-las-intervenciones)
5. [Adquisición y transporte ECG](#5-adquisición-y-transporte-ecg)
6. [Ciclo común de medición, PANI y alimentación](#6-ciclo-común-de-medición-pani-y-alimentación)
7. [Interfaz de medición y envío a Telsy Web](#7-interfaz-de-medición-y-envío-a-telsy-web)
8. [Navegación de instrucciones y apagado del equipo](#8-navegación-de-instrucciones-y-apagado-del-equipo)
9. [Batería](#9-batería)
10. [Teleconsulta](#10-teleconsulta)
11. [Metas Diarias y vídeos YouTube](#11-metas-diarias-y-vídeos-youtube)
12. [Inventario de archivos instalados](#12-inventario-de-archivos-instalados)
13. [Fuentes C, recursos preexistentes y elementos no instalados](#13-fuentes-c-recursos-preexistentes-y-elementos-no-instalados)
14. [Verificación y resultados](#14-verificación-y-resultados)
15. [Respaldos y reversión](#15-respaldos-y-reversión)
16. [Pendientes y límites](#16-pendientes-y-límites)
17. [Propuesta de versionado](#17-propuesta-de-versionado)
18. [Fuentes y trazabilidad](#18-fuentes-y-trazabilidad)
19. Anexo A: mapa de las últimas copias locales y sus huellas.
20. Anexo B: inventario local de entregables y herramientas.

## 1. Objetivo y alcance

Documentar qué se cambió, qué problema motivó cada intervención, cómo quedó resuelto, qué archivos participan y qué evidencia existe para defender los resultados. El objetivo es permitir mantenimiento, argumentación técnica, preparación de una entrega y continuidad del trabajo.

Se distinguen cuatro situaciones:

| Identificación | Significado |
|---|---|
| **Aplicado** | Existe registro de instalación o de comprobación posterior en el equipo. |
| **Verificado automáticamente** | Hay pruebas con datos simulados, compilación, comprobaciones HTTP/IPC o pruebas de navegador. Su alcance se precisa en cada apartado. |
| **Confirmado por el usuario** | Diego comunicó un resultado tras utilizar la pantalla o el banco. No implica una campaña completa de validación. |
| **Propuesto o pendiente** | Se analizó o preparó, pero no se presenta como parte de la solución instalada o como validación terminada. |

La aplicación original y buena parte de sus fuentes son anteriores a este trabajo. No se atribuye su creación a Codex. Las referencias históricas C2/D5 describen la base encontrada; no son versiones SemVer ni cambios nuevos atribuidos a esta intervención.

La carpeta local contiene paquetes parciales, copias y herramientas. No es una réplica completa del sistema desplegado ni existe aquí un historial Git completo que sustituya la reconstrucción documental. Los nombres de carpetas y fechas de modificación de Windows tampoco son prueba suficiente de una instalación remota.

## 2. Resumen de lo obtenido

| Área | Situación inicial o problema informado | Resultado documentado |
|---|---|---|
| ECG | Pérdida de historial al transportar snapshots; conteo por longitud de texto. | Buffer circular desde recepción UART, secuencias, tiempos y conteo real; se observaron aproximadamente 15.000 muestras en 60 s. |
| Sesiones | Cancelaciones y bucles síncronos dificultaban finalizar y repetir mediciones. | Trabajador cancelable y ciclo común para SpO₂, ECG, NIBP y Monitor. |
| PM6750 | Encendido permanente y enclavamientos de sesiones anteriores. | Encendido después de Empezar; apagado condicionado y limpieza al terminar/cancelar. |
| PANI | Estados terminales no manejados, bloqueos y ausencia de confirmación de cero. | Telemetría con novedad, manejo de estados, supervisión de descarga y recuperación explícita desde pantalla. |
| Resultados completos | Una medición podía conservar valores en pantalla pero perder el envío por una advertencia de descarga. | Conservación de los resultados válidos y reanudación después de recuperar; cancelaciones y fallos no se envían. |
| Diálogos | Mensajes nativos de Chromium y referencias a hardware poco claras. | SweetAlert, textos aprobados y distinción entre cancelación, finalización y presión pendiente. |
| Envío | Espera automática de 10 s. | Espera final de **5 s**, a partir de Aceptar y de la habilitación de Enviar. |
| Instrucciones | Recorrido completo obligatorio antes del monitoreo combinado. | Omitir instrucciones lleva a Empezar monitoreo sin iniciar la medición. |
| Apagado del equipo | Faltaba un control de apagado desde Información. | Botón, confirmación Sí/No y apagado del SO con controles de sesión/presión. |
| Batería | Porcentaje generalmente oculto e iconos poco informativos. | Porcentaje dentro del icono, nivel y estado; lectura ausente representada con `--%`. |
| Teleconsulta | Pestaña adicional sin control táctil para volver después de colgar. | Misma pestaña y extensión local con **Volver a Telsy-Hogar**, abajo a la izquierda. |
| YouTube | Error 153 y extracción frágil del identificador de vídeo. | Referer habilitado en vistas de vídeo y análisis de diferentes formatos de enlaces. |

**Estado al cierre del 16 de octubre:** el usuario comunicó que las pruebas realizadas hasta ese momento estaban bien. Quedaron pruebas adicionales y la reproducción de los vídeos asignados en una red que no impida YouTube.

## 3. Entorno y arquitectura

### 3.1 Ubicaciones y componentes

| Elemento | Valor registrado |
|---|---|
| Workspace de documentación y copias | `C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5` |
| Raspberry de banco | `TVSM01`, `172.16.70.32` |
| Alias SSH utilizado históricamente | `telsy`, usuario `pi`; verificado el 16 de octubre contra esa IP. |
| Proyecto remoto | `/home/pi/telsy-monitor/telsy` |
| Entorno virtual | `/home/pi/telsy-monitor/telsy_venv` |
| Ejecutable del driver | `/home/pi/telsy-monitor/telsy/vspm-pm6750` |
| UART registrada | `/dev/ttyAMA1` |
| Servicios | `telsy-driver.service` y `telsy-server.service` |
| Interfaz local | Chromium en modo kiosco, `http://127.0.0.1:8000/home/` |
| Aplicación HTTP/WebSocket | Django/Channels; el journal registró Django 4.2.23 y ASGI/Daphne 4.2.1. |
| Comando servidor observado | `python manage.py runserver 127.0.0.1:8000`, ejecutado desde el virtualenv. |
| Chromium registrado el 16 de octubre | 122.0.6261.89, Debian 11. |
| Plataforma de registros | Endpoint preexistente `vitalsignrecords` del backend de Telsy Web/Telecentro. |
| Teleconsulta externa | `https://videoconferencia.fcv.org/` |
| Alimentación del PM6750 | GPIO27: HIGH durante operación; LOW después del cierre autorizado. |

Estos datos son evidencia histórica del entorno, no una actualización de sus dependencias realizada en este trabajo. Las credenciales y enlaces privados de pacientes no forman parte de este documento.

### 3.2 Flujo funcional

```text
Chromium / pantalla táctil
  ├─ HTTP: pantallas, batería y apagado
  ├─ WebSocket: SpO₂, ECG, NIBP y Monitor
  └─ HTTPS: Telsy Web, Teleconsulta FCV y YouTube
          |
Django / Channels
  ├─ measure.py: configuración y adquisición
  ├─ lifecycle.py: sesión, cancelación y descarga
  ├─ ecg_ring.py: lectura ECG por lotes
  └─ driver_KR.py: comandos IPC
          |
Driver C / vspm-pm6750
  ├─ UART ↔ PM6750
  ├─ snapshots heredados
  ├─ ring ECG 5000
  └─ telemetría PANI 5001
```

### 3.3 Recursos IPC

| Key | Uso |
|---|---|
| 1000 | Semáforo de datos. |
| 1001 | Semáforo de comandos. |
| 2000 | Snapshot heredado de onda ECG. |
| 3000 | Parámetros heredados ECG/SpO₂/PANI/temperatura. |
| 4000 | Ondas heredadas SpO₂/respiración. |
| 2001 y 3001 | Buzones heredados; no son el destino del envío corregido de comandos. |
| 4001 | Buzón de comandos utilizado por Python y consumido por C. |
| **5000** | Nuevo buffer circular ECG, ABI versión 1. |
| **5001** | Nuevos metadatos PANI, ABI versión 1, 64 bytes. |

## 4. Cronología de las intervenciones

Las fechas se reconstruyen de informes y conversación. Las correcciones sucesivas dentro de una etapa se describen juntas; no se inventa una fecha distinta para cada edición intermedia.

| Etapa | Fecha o periodo | Trabajo |
|---|---|---|
| 1 | 5–6 de octubre | Conexión, identificación del equipo, servicios, base C2/D5 y copia local de fuentes seleccionadas. |
| 2 | 6 de octubre | Análisis del protocolo PM6750 y diagnóstico del transporte ECG. |
| 3 | 6 de octubre | Ring ECG, integración Python/frontend, compilación, pruebas e instalación de banco. |
| 4 | 6 de octubre y ajustes posteriores | Ciclo común, telemetría PANI, encendido al Empezar, cancelación y descarga. |
| 5 | 6–8 de octubre | Correcciones por silencio PANI, conservación de prueba de cero y recuperación desde pantalla. |
| 6 | 8 de octubre | Conservación/envío de resultados completos tras descarga pendiente; avisos de apagado al cancelar. |
| 7 | 8 de octubre | Botón de apagado del SO en Información. |
| 8 | 12 de octubre | Documento de contexto y análisis de despliegue/distribución; sin instalación de servidor de producción. |
| 9 | 16 de octubre | Confirmación del alias `telsy`; usuario confirmó apagado funcionando. |
| 10 | 16 de octubre | Omitir instrucciones, ajuste amarillo y traslado del apagado abajo a la derecha. |
| 11 | 16 de octubre | Nuevos textos de diálogos, distinción cancelada/completada y corrección de caché. |
| 12 | 16 de octubre | Aviso de finalización en las cuatro mediciones, estado detenido inmediato, espera de presión de 9 s. |
| 13 | 16 de octubre | Envío automático de 5 s; icono SVG detenido y flecha izquierda en medición. |
| 14 | 16 de octubre | Análisis y mejora del indicador de batería; propuesta inicial de Teleconsulta en iframe. |
| 15 | 16 de octubre | Confirmación del bloqueo externo, solicitud formal para FCV y solución local de regreso. |
| 16 | 16 de octubre | Corrección de YouTube en Metas Diarias y vistas de vídeo. |
| 17 | 16 de octubre | Último ajuste: Volver a Telsy-Hogar abajo a la izquierda, conservando 166 × 70 px. |
| 18 | 17 de octubre | Este inventario consolidado, sin SSH ni cambios de ejecución. |

## 5. Adquisición y transporte ECG

Se revisó el adjunto `Manual VSPM HW.pdf`, que contenía cinco páginas del protocolo Berry PM6750 V3.2 (2016), no un manual eléctrico completo. Se extrajo su texto y se revisaron las cinco páginas renderizadas. El documento describe comunicación a 115200, 8N1, siete amplitudes ECG por paquete y 250 paquetes/s. Su referencia a RS232 no permitió certificar niveles eléctricos, conexión directa con UART ni esquema de alimentación. Esa revisión fundamentó el transporte nuevo; no certifica por sí sola el diseño de hardware.

### 5.1 Problema identificado

El receptor UART actualizaba una estructura global y otro hilo publicaba su último contenido en memoria compartida. Python leía snapshots, sin historial ni secuencia. El tipo del último paquete recibido condicionaba además si se aceptaba una muestra ECG. Aumentar la velocidad de lectura Python no recuperaba datos ya sobrescritos.

También se registraba la longitud de la cadena ECG como si equivaliera a número de muestras. Una curva con más caracteres no demuestra una adquisición de más muestras.

### 5.2 Solución aplicada

- Publicación de cada paquete ECG válido directamente desde el hilo receptor UART a un ring separado, antes de que el snapshot global pueda sobrescribirlo.
- Definición C del ABI en `ecg_ring.h`: identificador, versión, capacidad, tamaño de registro, época de arranque y secuencia.
- Cada registro conserva secuencia, tiempo monotónico de recepción y siete derivaciones.
- Lectores Python con cursor independiente; una lectura no consume destructivamente los datos de otro lector.
- Lectura por lotes y procesamiento fuera del semáforo.
- Inicio de cada adquisición después del historial previo, evitando incorporar muestras de otra medición.
- Detección de sobreescritura, reinicio del driver, secuencias inconsistentes y tiempos no monotónicos.
- Acumulación en lista y unión final, conservando el formato de la onda utilizado por el frontend.
- Limitación de lecturas/envíos redundantes de parámetros a 5 Hz en la integración ECG/Monitor; no altera la tasa del ring.
- Rehabilitación explícita del flujo ECG al encender el módulo.

El ring tiene **4.096 registros de 24 bytes y una cabecera de 32 bytes**: 98.336 bytes, equivalentes a unos 16,384 s de retención a 250 paquetes/s. El límite evita un crecimiento ilimitado de memoria.

### 5.3 Integración y aceptación de la adquisición

`EcgConsumer` y `MonitorConsumer` usan `Acquisition` de `ecg_ring.py`. La duración se controla con reloj monotónico y se cuentan muestras reales. Se preserva la escala de la derivación II utilizada por la representación existente; conservar siete derivaciones en el transporte no significa que se hayan añadido siete gráficas.

El mensaje final incorpora `quality`, `sample_count`, `duration_s`, `effective_hz` y `max_gap_ms`. El frontend exige un resultado completo antes de componer el registro ECG.

Se establecieron límites **de banco** de 237,5–262,5 muestras/s y pausas máximas de 100 ms, incluidos los bordes de la adquisición. No son tolerancias oficiales del fabricante ni una validación clínica. No se interpolan ni se rellenan huecos para aparentar una adquisición válida.

La secuencia permite detectar pérdidas después de la recepción en Raspberry. El protocolo recibido no aporta una secuencia del módulo que permita contar con exactitud paquetes que nunca alcanzaron el receptor UART.

### 5.4 Construcción e instalación

Se modificó `pm6750_main.c`, se crearon cabeceras y se compiló un ejecutable candidato en la Raspberry. Los instaladores sustituyeron el ejecutable `vspm-pm6750` y los archivos Python/JS correspondientes. Se prepararon parches, manifiestos SHA256, instaladores y rollback.

El primer intento de instalación ECG revirtió a la base anterior porque el chequeo HTTP no esperó suficiente al arranque de Django. Se amplió esa espera y el siguiente intento terminó correctamente. No debe interpretarse el primer intento fallido como una instalación parcial definitiva.

## 6. Ciclo común de medición, PANI y alimentación

### 6.1 Gestión de las cuatro sesiones

Se creó `monitor/raspberry/lifecycle.py` y se integraron `Spo2Consumer`, `EcgConsumer`, `NibpConsumer` y `MonitorConsumer` mediante `ManagedMixin`.

Antes, los bucles síncronos y las banderas heredadas dificultaban atender cancelaciones y repetir mediciones. Ahora:

1. Conectar el WebSocket no energiza el PM6750.
2. Recibir el mensaje de duración posterior a Empezar inicia un trabajador cancelable.
3. El dispatcher de Channels queda disponible para Cancelar y desconexiones.
4. Una exclusión mutua impide dos sesiones principales simultáneas dentro del proceso.
5. La sesión conserva el control de inicialización, adquisición, descarga y limpieza.
6. Al completar el apagado autorizado se restablecen `measure_flarg`, `nibp_flarg`, `nibp_power_hold` y `buffer_data`.

El bloqueo usa `threading.Lock` y variables en memoria. No coordina varios procesos ASGI ni proporciona persistencia de sesión ante una caída del servidor. Esta es una restricción actual de arquitectura.

### 6.2 Alimentación del PM6750

La configuración anterior forzaba GPIO27 HIGH al iniciar el driver. Se creó el drop-in `11-session-power.conf`, que sustituye ese prearranque por GPIO27 LOW. El driver continúa ejecutándose y escuchando UART entre sesiones.

El módulo se enciende después de Empezar. ECG y SpO₂ finalizan mediante la limpieza común. NIBP y Monitor añaden la condición de descarga PANI antes de autorizar un corte de alimentación.

Se corrigió la persistencia de `nibp_power_hold`, que dejaba encendido el módulo y afectaba mediciones siguientes. La limpieza compartida permite iniciar nuevas sesiones después de finalizar o cancelar, salvo que exista una descarga pendiente real.

### 6.3 Telemetría PANI y estados terminales

Se añadió `nibp_meta.h` y un bloque IPC de 64 bytes en key 5001. Solo se actualiza con una trama PANI válida. Incluye época del driver, secuencia, tiempo monotónico y los campos de estado, presión y resultado.

El estado se extrae de los bits correspondientes y la presión CUFF se interpreta con la resolución de 2 mmHg del protocolo. Se evita tomar como nueva una lectura de la sesión anterior.

Los estados terminales 2 a 8 se contemplan explícitamente: detenida, sobrepresión, manga floja/desconectada, tiempo agotado, error de medición, interferencia y resultado fuera de rango. El estado 8 deja de mantener al consumidor esperando innecesariamente hasta 150 s.

La prueba 30/15 en adulto con arritmia devolvió estado 8 del propio PM6750. El software acepta un resultado válido `30S15S20` si el módulo lo entrega como normal; no transforma errores en resultados, no cambia automáticamente el modo de paciente y no modifica el algoritmo del fabricante.

### 6.4 Arranque: correcciones por comportamiento real del firmware

Una primera versión esperaba CUFF=0 antes de iniciar PANI. Se comprobó que este firmware no publica PANI en reposo, por lo que esa espera bloqueaba el comienzo de la medición.

La disponibilidad se pasó a comprobar mediante flujo ECG nuevo y estable antes de START. La duración ECG se cuenta después de confirmar ese flujo, no durante toda la inicialización del módulo.

### 6.5 Comandos y latencia

`driver_KR.py` escribe en el buzón 4001, que es el que consume el driver C. Se eliminaron escrituras redundantes en otros buzones y retardos heredados de 1,8 s. Se espera a que el buzón esté disponible y a que C consuma el comando, con plazo acotado y errores explícitos.

Se activó `SEM_UNDO` en los semáforos utilizados para reducir el riesgo de bloqueos retenidos al morir un proceso. La confirmación de consumo del buzón no demuestra que la válvula haya actuado ni que la presión física sea cero.

### 6.6 Descarga y prueba de cero

Al completar o cancelar PANI se envía STOP. El apagado requiere una trama terminal nueva con CUFF=0 y **un segundo de estabilización sin nuevas lecturas incompatibles**.

Se sustituyó la exigencia inicial de varios paquetes cero: el módulo podía emitir una única trama cero y dejar de publicar PANI. También se conserva la evidencia de cero observada en la sesión cuando PANI termina antes que ECG. Una presión posterior, estado activo o cambio de época invalida esa evidencia.

Un cero antiguo nunca observado como nuevo en la sesión no autoriza apagar. Tampoco se interpreta una trama detenida con CUFF=4 mmHg como si fuera cero.

El plazo final antes de solicitar comprobación física pasó de **12 a 9 segundos**. Se modificó la llamada de cierre a `wait_zero(..., timeout=9)`; no se retiraron el segundo de estabilización ni los requisitos de novedad y presión cero. Si la prueba automática llega antes, no es necesario agotar los nueve segundos. No es una promesa de tiempo total exacto desde la pulsación, porque existen comandos, procesamiento y comunicación adicionales.

### 6.7 Recuperación desde pantalla

Cuando falta prueba de descarga, se mantiene alimentación y bloqueo, y el backend informa `requires_zero_confirmation`. El usuario comprueba físicamente 0 mmHg y ausencia de medición activa antes de pulsar Confirmar.

Una conexión de recuperación envía `confirm_zero`. El backend toma el mismo bloqueo de sesiones, impide recuperar mientras exista una medición activa y, si procede, apaga y libera el estado. La respuesta `recovered` permite continuar el flujo de pantalla.

Todavía no conserva la condición pendiente y permite reintentar. El cambio evita depender de SSH para recuperar una sesión, pero no convierte el caso sin trama cero en una descarga verificada automáticamente.

### 6.8 Confirmación de apagado de una sesión

Se añadió `powered_off` después de una limpieza/apagado exitosos en cancelación. No se emite como confirmación de éxito si falla el apagado. El frontend evita mostrar el mismo aviso varias veces.

Durante el diagnóstico se utilizó temporalmente un driver instrumentado para observar UART. Los registros indican su retirada y la restauración del binario definitivo; ese driver temporal no forma parte de la solución final.

## 7. Interfaz de medición y envío a Telsy Web

### 7.1 Resultados completos, fallos y descarga pendiente

Se introdujeron indicadores de resultado completo, error, cancelación y descarga pendiente. Antes de enviar se comprueban los requisitos de cada medición: NIBP válido, ECG completo donde corresponda y datos válidos para considerar finalizada SpO₂.

Una advertencia de descarga pendiente no marca por sí sola una adquisición completa como fallida. Se conservan presión y ECG, y después de recuperar se habilita el envío si el registro sigue siendo válido. Esta corrección atiende el caso 200/120 informado por el usuario.

Cancelación, adquisición incompleta, error real, cierre anormal o recuperación fallida siguen impidiendo el envío automático. No se generan resultados ficticios ni registros de ceros para suplir errores.

La coordinación evita programar varias veces el envío por eventos repetidos de cierre/recuperación. Esa comprobación de interfaz no equivale a una garantía general de idempotencia del backend remoto, que no se modificó.

### 7.2 Diálogos y textos finales aprobados

Se reemplazaron los avisos nativos añadidos en la medición por SweetAlert, evitando el encabezado de Chromium con `127.0.0.1:8000`.

| Situación | Título final | Mensaje o botones |
|---|---|---|
| Descarga pendiente | **Confirmar liberación de presión** | «Comprobar que la manga esté completamente desinflada (0 mmHg) y que no haya una medición en curso. Confirmar solo después de verificar.» |
| Acciones de presión | — | **Confirmar** / **Todavía no**. |
| Cancelación terminada | **Medición cancelada con éxito** | «Puede volver a la pantalla principal para iniciar una nueva medición.» |
| Resultado completo | **Medición completada con éxito** | Mismo mensaje de regreso; botón **Aceptar**. |

Los textos históricos Módulo apagado, Confirmar descarga de la manga y Confirmo 0 mmHg quedaron reemplazados en esos diálogos. Existe un título neutral **Equipo listo** para una recuperación que no permite afirmar que haya una medición completa; evita anunciar como completa una adquisición inválida.

### 7.3 Orden de cierre correcto en las cuatro mediciones

El primer cambio de textos reutilizaba cuadros existentes; no añadía el cuadro a todos los cierres normales. Después de la observación del usuario se unificó el flujo de **Monitor, NIBP, ECG y SpO₂**:

1. Finaliza la adquisición y el backend completa su cierre autorizado.
2. Si hay presión pendiente, primero se resuelve su confirmación.
3. Se presenta Medición completada con éxito cuando el resultado es válido.
4. Se espera a Aceptar.
5. Se habilita Enviar y comienza el temporizador de envío automático.

La promesa del diálogo se comparte para evitar habilitar Enviar mientras otro evento aún mantiene abierto el cuadro. No se anuncia éxito de una adquisición que haya fallado.

### 7.4 Estado visible después de Detener

La pulsación cambia inmediatamente el texto a **Medición detenida**, conserva ese estado durante la descarga y evita enviar varias cancelaciones por pulsaciones repetidas.

La primera implementación usó el emoji de prohibición. En la pantalla no se renderizaba y el texto desbordaba. Se reemplazó por el archivo SVG `measurement-stopped.svg`, se ajustaron ancho, tipografía y distribución del botón y se mantuvo el texto completo visible.

Al detener, el botón derecho de tres líneas cambia al icono `arrow-left.png`, con descripción accesible de regreso. Se conserva el comportamiento existente hacia el menú. Si queda pendiente la presión, el control permite volver a abrir su confirmación.

### 7.5 Temporizador de envío: valor final

**Valor vigente documentado: 5 segundos.** Inicialmente eran 10 segundos y se redujeron por petición del usuario.

El conteo empieza después de Aceptar en el cuadro de éxito, cuando se habilita Enviar. No comienza durante la medición ni mientras se espera la comprobación de presión. El envío manual sigue disponible antes de agotar el plazo.

Se conservó la composición de registros y el endpoint preexistente `vitalsignrecords`. Las pruebas automáticas utilizan `postMethod` simulado para no crear registros reales. No se modificó el procesamiento del backend de Telsy Web ni se verificó exhaustivamente su almacenamiento.

### 7.6 Corrección de caché

El usuario informó que seguía viendo los textos antiguos pese a que el servidor entregaba el JavaScript nuevo. Se añadió un identificador de versión a la referencia de `telsy.js` en la plantilla de medición y se actualizaron las referencias correspondientes en otras vistas intervenidas.

Los parámetros `?v=...` sirven para renovar recursos de navegador. No representan por sí solos la versión SemVer de TELSY ni sustituyen un manifiesto de entrega. El usuario confirmó que después del ajuste sí aparecieron los cambios.

## 8. Navegación de instrucciones y apagado del equipo

### 8.1 Omitir instrucciones de Monitor

Se revisaron las capturas de `Proceso_MonitorConsumer.pdf`. Se añadió un enlace en la pantalla inicial Monitoreo, a la izquierda de Iniciar:

```text
Omitir instrucciones → /monitoring/?s=3&u=0 → Empezar monitoreo
Iniciar              → /monitoring/?s=0&u=0 → recorrido de instrucciones
```

El nuevo enlace no envía el formulario de medición. El usuario conserva la decisión de pulsar Empezar después de llegar a la pantalla final.

Primero se instaló con el estilo claro existente. Después se redujo su altura y tipografía y se reutilizó la clase `yellow`, del mismo amarillo de Metas Diarias, para que predominara Iniciar. Este cambio se limitó a `monitor.html`; no se cambiaron las rutas ni la secuencia existente.

### 8.2 Apagado del sistema operativo

Se creó un botón con icono rojo de encendido en Información. Al principio estaba debajo del menú superior derecho; después se trasladó a la **esquina inferior derecha**, que es su posición final.

El flujo implementado es:

1. Pulsar el botón de apagado.
2. SweetAlert pregunta si se desea apagar TELSY, con **Sí / No**.
3. Sí envía POST `/shutdown/` con CSRF y confirmación JSON explícita.
4. El servidor toma el bloqueo de sesión y rechaza medición activa, descarga pendiente o GPIO27 alto.
5. Si procede, ejecuta `sudo -n shutdown -h now`.
6. Mantiene el bloqueo después de aceptar el apagado para impedir una nueva medición durante la salida del SO.

Se añadieron `monitor/shutdown.py`, `monitor/static/js/shutdown.js` y la ruta en `monitor/urls.py`. `information.html` incorpora el botón, el token y los scripts de diálogo.

Las pruebas automáticas simularon el comando de apagado, comprobaron los rechazos y no apagaron el equipo. Posteriormente Diego confirmó que el botón se visualizaba y funcionaba físicamente. No se modificó `sudoers`; se utilizó el permiso ya disponible en el equipo.

## 9. Batería

### 9.1 Análisis de la lectura heredada

El script llamado `geekworm_x728.py` utiliza `/dev/i2c-5`, dirección 0x48, e interpreta señales GPIO23, GPIO24 y GPIO25. El nombre del archivo no demuestra por sí solo cuál es el circuito de batería montado.

El porcentaje se obtiene de `voltage // 111`, limitado a 0–100. Por eso el ejemplo de 9767 mV produce 87 %. Es una estimación de la implementación heredada, no una curva calibrada ni una medición directa del estado de carga.

Se mantuvieron la combinación de bytes del ADC y la escala de voltaje. Antes de corregirlas es necesario identificar referencia, divisor resistivo, química, número de celdas y tensión real medida. No se deducen esos datos únicamente de la salida de consola.

### 9.2 Correcciones de código

- Apertura I²C con `with`, garantizando cierre del archivo y evitando una referencia a `i2c_fd` no inicializado si falla la apertura.
- Una única muestra de voltaje por instantánea; el porcentaje se calcula con esa misma muestra.
- Lectura inválida representada explícitamente, sin convertirla en un 0 % válido ni ocultarla tras un porcentaje anterior.
- Diagnóstico de uso de batería como no cargando, conservando el estado bruto de carga por separado.
- `GPIO.cleanup()` limitado a GPIO23, GPIO24 y GPIO25. Ejecutar la prueba de batería por consola no debe limpiar GPIO27 ni otros pines del proyecto.
- Ampliación de `/battery/` para informar validez, estimación, voltaje y estado, manteniendo `capacity` y `power_ac`.

Los estados de interfaz distinguen uso de batería, cargando, carga completa, conectado a corriente y lectura no disponible. Estar conectado a corriente no se presenta automáticamente como carga en curso si las señales no lo indican.

### 9.3 Indicador final

Se modificó Inicio y se añadieron `battery-indicator.js` y `battery-indicator.css`. El indicador horizontal muestra el porcentaje dentro, relleno proporcional, color rojo en nivel bajo, verde en nivel alto/carga completa y rayo durante carga en curso. La lectura ausente se presenta con `--%` y la descripción accesible indica que el porcentaje es estimado.

Se conserva la consulta cada cinco segundos y el comportamiento de alerta de batería baja. La señal de carga completa puede coexistir con 92 % estimado; no se fuerza 100 % para ocultar una diferencia de calibración.

La verificación real registrada mostró lectura válida de 92 %, alimentación externa y estado de carga completa. No se realizó un ciclo completo de carga/descarga ni se calibró la batería. El usuario confirmó que el nuevo icono funcionaba en sus pruebas.

## 10. Teleconsulta

### 10.1 Problema y propuesta inicial

La confirmación Sí abría la llamada con `target="_blank"`. Chromium en modo kiosco dejaba al usuario en esa pestaña y sin una salida táctil visible después de colgar. Se preparó una vista con iframe y flecha de regreso.

Al conocer el dominio real se comprobó que la página pública de `videoconferencia.fcv.org` respondía con **`X-Frame-Options: DENY`**. Se conservó evidencia de las cabeceras y se preparó una solicitud formal de autorización de incrustación para Infraestructura/Desarrollo de FCV.

**La propuesta de iframe no se activó.** Sus archivos locales son una propuesta histórica, no la implementación instalada. No se modificó el servidor de FCV ni se retiraron sus cabeceras de protección.

### 10.2 Solución instalada para recuperar la navegación

Para resolver la necesidad inmediata de regreso:

- La llamada abre en **la misma pestaña**.
- Se añadió una extensión local Chromium Manifest V3 que coloca un control de regreso en las páginas FCV autorizadas.
- El botón regresa a `http://127.0.0.1:8000/menu/`.
- Se comprueba la disponibilidad de ese control antes de navegar desde la confirmación.
- Se valida que el enlace de consulta use HTTPS y el dominio esperado, sin credenciales incrustadas.
- Se evitan dobles pulsaciones y se eliminó el retorno automático de 30 s de la confirmación.
- El lanzador `startweb.sh` carga la extensión en cada inicio de Chromium.

Esto es una solución de navegación en la misma pestaña, **no una incrustación iframe**. Permite volver al aplicativo sin depender del cambio externo pendiente.

### 10.3 Archivos y alcance de la extensión

La extensión se añadió fuera del árbol Django:

```text
/home/pi/telsy-monitor/chromium-extensions/telsy-return/
├── manifest.json
└── return-button.js
```

Sus coincidencias se limitan a:

- La página local `http://127.0.0.1:8000/videocall/*`, para informar que el regreso está disponible.
- `https://videoconferencia.fcv.org/*`.
- `https://www.fcv.org/*`.
- `https://fcv.org/*`.

No se autorizó el acceso genérico a todos los sitios. No se añadieron captura de datos de pacientes, envío de información, permisos de cámara de extensión ni retirada de cabeceras. El botón usa Shadow DOM cerrado para aislar su estilo y se conserva ante determinados cambios del documento/pantalla completa.

Los cambios en permisos de red, dominio o mecanismo de inicio del kiosco requieren revisar esta integración. Si FCV redirige en el futuro a otro dominio, este no recibe automáticamente el control de regreso.

### 10.4 Última posición y texto

Primero se instaló arriba a la derecha con Volver a TELSY. Por petición posterior se cambió a:

| Propiedad | Valor final |
|---|---|
| Posición | Inferior izquierda, `left:16px`, `bottom:12px`. |
| Dimensiones | **166 × 70 píxeles**, sin aumento. |
| Texto | **Volver a Telsy-Hogar**, en dos líneas. |
| Icono | Flecha izquierda SVG. |
| Tipografía | Ajustada a 16 px para conservar el tamaño y el texto completo. |

La versión declarada del manifiesto continúa en `1.0.0`; el ajuste visual no recibió una nueva versión del manifiesto. Se documenta esta situación para corregir el procedimiento de versionado futuro, no para afirmar un cambio que no se hizo.

Se validaron posición y medidas en Chromium real, y se probó el regreso desde la página pública de videoconferencia y desde el portal FCV. El usuario confirmó el funcionamiento del botón. No se presenta como terminada una campaña de llamadas clínicas con audio y cámara.

Chromium se cerró normalmente y se reabrió para activar la extensión, con autorización del usuario. Se corrigió un primer intento que reutilizó la sesión existente y, por tanto, no cargaba todavía la nueva extensión. Los navegadores aislados y túneles temporales de prueba se cerraron al finalizar.

## 11. Metas Diarias y vídeos YouTube

### 11.1 Causa comprobada del error 153

Las capturas mostraban Video player configuration error / Error 153. Las respuestas de las vistas tenían `Referrer-Policy: same-origin`, que evitaba enviar Referer a YouTube. La documentación de YouTube exige esa referencia para identificar el contexto de inserción.

Se aplicó `strict-origin-when-cross-origin` en la respuesta de **las vistas de vídeo** `goalsv` y `monitoringinfov` y en sus iframes. No se cambió globalmente la política de todas las páginas ni se añadió una clave de API.

### 11.2 Enlaces y estados de contenido

Se creó `goals-media.js` para analizar identificadores y enlaces `watch?v=`, `youtu.be`, `/embed/`, `/shorts/` y `/live/`. El iframe recibe la URL de inserción y el origen real del aplicativo.

La extracción anterior dependía de dividir por `=` y podía fallar con otros formatos. Los enlaces ausentes o inválidos ahora muestran un mensaje; se evita reproducir un vídeo predeterminado ajeno a la meta. Los detalles vacíos de una meta también muestran una explicación, en lugar de quedar silenciosamente en blanco.

Las categorías son **Autocuidado, Ejercicio, Dietas y Calidad de vida**. No se editaron las metas, sus prescripciones ni los enlaces asignados al paciente en Telsy Web.

### 11.3 Resultado y pendiente de red

En Chromium aislado se cargaron cuatro categorías con formatos distintos de un vídeo público de prueba de Google. Se comprobaron las solicitudes reales con `Referer: http://127.0.0.1:8000/` y una captura mostró el reproductor cargado sin error 153.

Esto no prueba que todos los vídeos asignados permitan inserción, estén disponibles ni puedan reproducirse en una red empresarial restrictiva. El usuario informó que cambiaría de red antes de realizar esa verificación. No se cambiaron firewall, proxy ni políticas de red de la empresa.

## 12. Inventario de archivos instalados

Las rutas de esta sección son del dispositivo remoto. El Anexo A enlaza cada archivo con su última copia local conservada y su SHA256. «Nuevo» se refiere a nuevo respecto de la base de este trabajo, aunque posteriormente haya sido ampliado.

### 12.1 Python, driver y rutas

| Ruta dentro de `/home/pi/telsy-monitor/telsy/` | Acción | Responsabilidad del cambio |
|---|---|---|
| `monitor/raspberry/measure.py` | Modificado | Integración de los cuatro consumidores, ring ECG, estados PANI, encendido/configuración y limpieza. |
| `monitor/raspberry/driver_KR.py` | Modificado | Envío de comandos por 4001, espera de consumo y semáforos. |
| `monitor/raspberry/ecg_ring.py` | Nuevo | Lectura por cursor, adquisición, métricas y rechazo de pérdidas. |
| `monitor/raspberry/lifecycle.py` | Nuevo y ampliado | Exclusión, trabajador, cancelación, telemetría/prueba de cero, recuperación y espera final de 9 s. |
| `monitor/raspberry/geekworm_x728.py` | Modificado | Lectura coherente, cierre I²C, estado inválido y limpieza limitada de GPIO. |
| `monitor/shutdown.py` | Nuevo | Vista POST de apagado, CSRF y condiciones de sesión/presión. |
| `monitor/urls.py` | Modificado | Ruta `/shutdown/`. |
| `monitor/views.py` | Modificado | Batería ampliada y política Referer específica de las vistas de vídeo. |
| `vspm-pm6750` | Sustituido | Driver compilado con ring ECG y metadatos PANI. |

### 12.2 JavaScript, CSS y recursos

| Ruta dentro del proyecto remoto | Acción | Responsabilidad del cambio |
|---|---|---|
| `monitor/static/js/telsy.js` | Modificado repetidamente | Resultados, cancelación, recuperación, diálogos, temporizador 5 s, batería y delegación de navegación/vídeos. |
| `monitor/static/js/shutdown.js` | Nuevo | Confirmación Sí/No, POST con CSRF y estado de apagado. |
| `monitor/static/js/battery-indicator.js` | Nuevo | Porcentaje, nivel, estados de carga y ausencia de lectura. |
| `monitor/static/css/battery-indicator.css` | Nuevo | Icono horizontal, relleno, texto interior y colores. |
| `monitor/static/images/icons/measurement-stopped.svg` | Nuevo | Prohibición visible sin depender de un emoji. |
| `monitor/static/js/teleconsult-navigation.js` | Nuevo | Verificación de regreso disponible y navegación en la misma pestaña al dominio permitido. |
| `monitor/static/js/goals-media.js` | Nuevo | Selección de meta, identificación de vídeo, política Referer y mensajes por enlace inválido. |

### 12.3 Plantillas

| Ruta dentro del proyecto remoto | Acción | Responsabilidad del cambio |
|---|---|---|
| `monitor/templates/information.html` | Modificada | Apagado, CSRF y ubicación final inferior derecha. |
| `monitor/templates/monitor.html` | Modificada | Omitir instrucciones, menor tamaño y amarillo. |
| `monitor/templates/measure.html` | Modificada | Versión del JS, identificadores de navegación y estilos del estado detenido. |
| `monitor/templates/home.html` | Modificada | Batería con porcentaje interior y carga de recursos nuevos. |
| `monitor/templates/videocall.html` | Modificada | Retirada de `_blank` y carga de navegación con regreso. |
| `monitor/templates/goalsv.html` | Modificada | Referer explícito, iframe sin vídeo ajeno por defecto, mensaje y recursos. |
| `monitor/templates/goalsd.html` | Modificada | Carga del auxiliar de metas y del JS actualizado. |
| `monitor/templates/monitoringinfov.html` | Modificada | Política del iframe y recursos actualizados para vídeos de instrucciones. |

### 12.4 Fuera del proyecto Django

| Ruta remota absoluta | Acción | Función |
|---|---|---|
| `/home/pi/telsy-monitor/startweb.sh` | Modificado | Añade `--load-extension` al inicio del kiosco. |
| `/home/pi/telsy-monitor/chromium-extensions/telsy-return/manifest.json` | Nuevo | Declara la extensión local y sus dominios/páginas limitados. |
| `/home/pi/telsy-monitor/chromium-extensions/telsy-return/return-button.js` | Nuevo y ajustado | Control persistente y posición/texto finales. |
| `/etc/systemd/system/telsy-server.service.d/90-ecg-bench.conf` | Nuevo | Envía stdout/stderr del servidor al journal. |
| `/etc/systemd/system/telsy-driver.service.d/11-session-power.conf` | Nuevo | Sustituye el prearranque GPIO27 HIGH por LOW. |

No se afirma una modificación de los archivos unit base de systemd. Los cambios descritos se hicieron mediante drop-ins. Tampoco se presenta `10-gpio.conf`, ya existente, como archivo nuevo de esta etapa.

## 13. Fuentes C, recursos preexistentes y elementos no instalados

### 13.1 Fuentes de construcción del driver

| Archivo de fuentes | Acción comprobada frente a baseline |
|---|---|
| `vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750_main.c` | Modificado para publicar ECG y telemetría PANI. |
| `vspm-pm6750_DRIVER/vspm/common/inc/ecg_ring.h` | Nuevo ABI del ring ECG. |
| `vspm-pm6750_DRIVER/vspm/common/inc/nibp_meta.h` | Nuevo ABI de metadatos PANI. |

La comparación local confirma que los demás archivos C y cabeceras copiados en esos paquetes no presentan cambios respecto del baseline seleccionado. Entre ellos están `pm6750.c`, `pm6750.h`, `serial.c/.h`, `ipc.c/.h`, `cli.c/.h` y `jutils.c/.h`.

**Los instaladores no copiaron todo ese árbol de fuentes al proyecto remoto:** instalaron el binario compilado. Las fuentes y cabeceras están conservadas en los paquetes locales y en el staging de construcción de aquella etapa. No se certifica que el árbol C originalmente encontrado en la Raspberry haya sido sincronizado con ellas.

### 13.2 Archivos consultados o reutilizados sin atribuir modificación

`monitor/routing.py` se copió para estudiar rutas, pero no se registra un cambio funcional suyo. Los estilos `telsy.css`, Bootstrap, SweetAlert2, FontAwesome y los iconos `arrow-left.png`, `ico-nav.png` y similares ya existían. Reutilizarlos o añadir una referencia en una plantilla no equivale a crear esas bibliotecas o recursos.

Tampoco se atribuyen cambios en `manage.py`, el ASGI raíz, `telsy/settings.py`, el teclado GPIO, los algoritmos internos del PM6750, el backend de Telsy Web, la base de datos de pacientes ni el Apache de FCV. Las inspecciones históricas de estos componentes no constituyen una modificación.

### 13.3 Propuestas que no deben confundirse con producción

- `outputs/teleconsult-battery/teleconsult.js`, `videocall.html` y la variante completa `telsy.js` de esa carpeta pertenecen a la propuesta de iframe. No fueron activados como esa solución.
- La batería se desplegó con la variante `telsy-battery-only.js`; después, `telsy.js` evolucionó de nuevo en el paquete de Teleconsulta/YouTube.
- `transport-draft.patch` es un borrador de transporte anterior a la integración completa.
- Los archivos de `baseline/` son copias para comparación o reversión, no archivos nuevos que deban instalarse sobre la aplicación.
- Los ZIP de ECG y lifecycle son paquetes históricos de banco. No incorporan automáticamente las modificaciones de octubre 16.
- El documento de despliegue del 12 de octubre propone migración ASGI, empaquetado y otras medidas. No demuestra instalación de Gunicorn/Uvicorn, PyInstaller, Nginx, nuevos workers ni una política nueva de logs.
- Las recomendaciones de logs volátiles, retención y almacenamiento en memoria permanecen como análisis de despliegue. El cambio de logging aplicado que se acredita aquí es el drop-in de salida al journal; no se afirma haber configurado journald en modo volátil ni montado nuevos tmpfs.

## 14. Verificación y resultados

### 14.1 Evidencia automática y de banco

Los conteos siguientes corresponden a hitos distintos; **no se suman como pruebas independientes**, porque las suites fueron ampliándose y reutilizándose.

| Evidencia registrada | Resultado y alcance |
|---|---|
| Transporte ECG inicial | 17 pruebas Python y 14 escenarios frontend en esa etapa. |
| Productor C / lectores con IPC privado | 15.000 muestras, orden conservado y lectores independientes; desbordamiento y reinicio detectados. |
| Compilación/enlace C en Raspberry | Correctos; persistían advertencias heredadas. |
| Ring observado pasivamente | 1.256 muestras en 5,006 s; 250,885 Hz; pausa máxima 9,907 ms. |
| ECG real por WebSocket | 2.508 muestras en 10 s; 250,8 Hz; cierre 1000. |
| Monitor real de 60 s | PANI 124/77, PAM92; ECG 15.047 muestras, 250,783 Hz; pausa máxima 13,755 ms; cierre 1000 y GPIO27 LOW. |
| Sesiones sucesivas | SpO₂ completa/cancelada/repetida y ECG posterior sin reiniciar servicios; apagado comprobado en cada cierre. |
| Cancelación PANI/Monitor | Presurización, STOP, seguimiento de descarga y cierres comprobados en banco. |
| Recuperación sin trama cero | Comprobación física explícita del usuario, respuesta recovered y nuevas mediciones sin reiniciar. |
| Ciclo común ampliado | 25 pruebas Python; suites frontend históricas alcanzaron 42 escenarios antes de ampliaciones posteriores. |
| Resultado completo 200/120 | Composición simulada conservó SYS200, DIA120, PAM150 y ECG; sin envío antes de recuperar ni programación duplicada. |
| Apagado del SO | 8 pruebas Python y 4 escenarios JavaScript; comando simulado; GET405 y POST sin CSRF403. |
| Cierre de las cuatro mediciones | Verificado que Enviar y su temporizador esperan Aceptar, que la cancelación no envía y que el plazo final es 5.000 ms. |
| Batería | 5 pruebas del lector y 2 de la vista; porcentajes, estados y lectura ausente; endpoint real y captura de Inicio. |
| Teleconsulta | Chromium real aislado, botón en ambos portales y retorno a menú en la misma pestaña; disponibilidad de extensión en confirmación. |
| Último ajuste de regreso | Comprobación de left16/bottom12, 166×70 px, texto visible y clic de retorno. |
| YouTube | Cuatro categorías enviaron el Referer real requerido; reproductor público de prueba cargado sin error153. |

Los clientes automáticos de adquisición no enviaron registros de prueba al Telecentro. Las pruebas de composición de datos usaron un envío simulado. No se presenta la cifra de muestras como una evaluación de exactitud diagnóstica.

### 14.2 Confirmaciones comunicadas por Diego

- Aproximadamente 15.000 muestras ECG y funcionamiento correcto de ECG en sus pruebas.
- Funcionamiento de Monitor, NIBP, ECG y SpO₂ después de las correcciones, con incidencias intermedias que motivaron nuevos ajustes.
- Aparición de los diálogos y actualización visible después de corregir la carga de JavaScript.
- Botón de apagado visible y funcionando.
- Omitir instrucciones funcionando y aceptación del tamaño/color.
- Aceptación de la nueva ubicación de apagado y de los textos aprobados.
- Nuevo icono de batería funcionando.
- Botón de regreso de Teleconsulta funcionando; solicitud posterior de ubicación inferior izquierda y texto final.
- Al cerrar el trabajo del 16 de octubre: las pruebas realizadas estaban bien y faltaban verificaciones adicionales.

## 15. Respaldos y reversión

### 15.1 Principio utilizado

Se conservaron originales antes de sustituir componentes, se compararon hashes en las instalaciones correspondientes y se verificó lo servido por HTTP. Los cambios exclusivamente visuales no necesitaron reiniciar servicios; las modificaciones backend se instalaron con el módulo apagado y la sesión en estado seguro.

Los respaldos son por etapa. Restaurar uno antiguo puede retirar mejoras posteriores. No existe evidencia de un único rollback integral que reconstruya automáticamente todos los cambios del 16 de octubre.

### 15.2 Rutas históricas registradas

Todas las rutas de la tabla se encuentran bajo `/home/pi/telsy-monitor/`. Su existencia actual no se comprobó en esta elaboración.

| Directorio de respaldo | Contexto |
|---|---|
| `TELSY_ECG_BACKUP.x0Lviwvv` | Primer intento ECG, revertido automáticamente. |
| `TELSY_ECG_BACKUP.AL0fXGJU` | Base previa a la instalación ECG efectiva. |
| `TELSY_LIFECYCLE_BACKUP.kpeknQpS` | Base ECG previa al ciclo común; también preservó variantes de diagnóstico/arranque. |
| `TELSY_UI_BACKUP.12kYY532` | Estado previo a los cambios de recuperación de resultados completos. |
| `TELSY_UI_BACKUP.oAErFP2z` | Primera variante de la corrección de recuperación frontend. |
| `TELSY_CANCEL_ACK_BACKUP.hQK2uFrx` | Antes de powered_off y su aviso de cancelación. |
| `TELSY_SHUTDOWN_BACKUP.U1FXbDwg` | Información y rutas previas al botón de apagado. |
| `TELSY_SKIP_INSTRUCTIONS_BACKUP.8G0LkbOa` | Monitoreo previo a Omitir instrucciones. |
| `TELSY_BUTTON_STYLE_BACKUP.FGno3iFo` | Antes de tamaño/color y ubicación inferior del apagado. |
| `TELSY_DIALOG_COPY_BACKUP.wjCJDJFD` | Antes de textos aprobados. |
| `TELSY_DIALOG_CACHE_BACKUP.iiCybuRb` | Antes de forzar renovación del JS en medición. |
| `TELSY_FINISH_FLOW_BACKUP.90MLGEde` | Antes de finalización común y espera PANI de 9 s. |
| `TELSY_STOPPED_UI_BACKUP.dWYxgm5u` | Antes de icono detenido, flecha y temporizador de 5 s. |
| `TELSY_BATTERY_UI_BACKUP.z9ec6nhL` | Antes de lectura/indicador de batería. |
| `TELSY_TELECONSULT_YOUTUBE_BACKUP.oBcFmUJN` | Antes de regreso persistente y corrección de vídeos. |
| `TELSY_RETURN_POSITION_BACKUP.uzVNFd8x` | Antes de la última ubicación/texto del regreso. |

Los instaladores históricos exigen `TELSY_BENCH_CONFIRMED=YES` y una base concreta. No deben ejecutarse indiscriminadamente sobre el estado actual. Una reversión de componentes C/Python debe mantener su compatibilidad de ABI y realizarse sin medición activa ni presión pendiente. Los scripts históricos no eliminan arbitrariamente recursos IPC de terceros.

## 16. Pendientes y límites

1. **Campaña final de pruebas:** repetir mediciones completas/canceladas, secuencias sucesivas, reinicio completo y recuperación bajo las condiciones de uso previstas. La aceptación parcial del usuario no sustituye esa campaña.
2. **YouTube en otra red:** reproducir los vídeos realmente asignados a las cuatro categorías, con las restricciones empresariales resueltas. La verificación técnica del Referer ya se hizo, pero no se garantizan disponibilidad ni permisos de cada vídeo.
3. **Teleconsulta:** probar llamadas completas con cámara, micrófono, colgado y regreso; revisar dominios adicionales si el proveedor cambia el recorrido. El iframe continúa pendiente de la autorización formal del servidor FCV.
4. **Batería:** identificar hardware real, batería y divisor, contrastar voltaje con instrumento y calibrar porcentaje. Conservar la distinción entre dato estimado y señal de carga completa.
5. **PANI 30/15 en adulto con arritmia:** no se demuestra que el PM6750 pueda producir esa medición; se corrigió el manejo del error y la salida de la sesión.
6. **Temperatura y derivaciones:** se documentaron discrepancias heredadas de conversión decimal de temperatura y selección de V en snapshots. No se aplicaron esas correcciones como parte del transporte ECG.
7. **Estados ECG y HR/RR:** existen discrepancias 200/216 y de unidades en el protocolo revisado. No se presenta como resuelta una política de interpretación que no se cambió específicamente.
8. **Calidad de datos:** las métricas de ECG verifican transporte y continuidad de banco, no exactitud clínica ni paquetes que nunca llegaron a UART.
9. **Arquitectura de procesos:** el bloqueo de hardware es local al proceso. No se debe escalar a varios workers sin rediseñar la coordinación. GaugeConsumer, TelsyConsumer y otros auxiliares no están certificados bajo el mismo flujo de los cuatro consumidores principales.
10. **Caídas y actualizaciones:** la memoria de sesión no persiste ante un reinicio. Debe probarse la recuperación si cae el servidor/driver durante PANI. No realizar actualizaciones durante una medición.
11. **Despliegue de producción:** la evidencia registra runserver con ASGI/Daphne. La migración propuesta a un servidor ASGI de producción, estáticos y dependencias soportadas no se instaló en estas intervenciones.
12. **Actualización GitLab e Información:** se observaron intentos heredados con autenticación fallida y un usuario de lanzamiento inexistente. El identificador/fecha mostrado en Información puede ser antiguo y no representa por sí solo el conjunto de cambios de esta documentación. No se repararon credenciales ni la actualización automática.
13. **Código consolidado:** falta reunir en un repositorio completo la última combinación de cambios. Ningún ZIP histórico ni carpeta individual de outputs equivale por sí sola a la entrega final.

## 17. Propuesta de versionado

### 17.1 Adoptar SemVer con un contrato definido

Se recomienda `MAYOR.MENOR.PARCHE`, con etiquetas de prelanzamiento y metadatos cuando correspondan. La regla oficial vincula MAYOR con cambios incompatibles del API, MENOR con funcionalidades compatibles y PARCHE con correcciones compatibles. TELSY debe definir previamente su contrato de compatibilidad: HTTP/WebSocket, datos hacia Telsy Web, ABI del driver, configuración y dependencias del dispositivo. Fuente: [SemVer 2.0.0 en español](https://semver.org/lang/es/).

| Cambio futuro en TELSY | Incremento sugerido, sujeto al contrato |
|---|---|
| Corregir posición de un botón, una etiqueta o un fallo de URL sin romper compatibilidad | PARCHE. |
| Añadir una función compatible, como un nuevo control de interfaz | MENOR. |
| Cambiar incompatiblemente mensajes WebSocket, estructura de registros, configuración exigida o ABI público | MAYOR. |

La repercusión funcional y la compatibilidad importan más que la cantidad de archivos editados. Un cambio pequeño puede ser incompatible; muchos cambios visuales compatibles no obligan a subir MAYOR.

### 17.2 Qué versión asignar al estado actual

No se encontró una entrega oficial SemVer completa y trazable que permita afirmar «la versión instalada es X». `Telsy_v2.5` es el nombre de esta carpeta, y los hashes/fechas antiguos de Información no certifican una versión nueva.

**Ejemplo condicionado:** si el responsable confirma que la línea oficial previa era 2.5.x y que el contrato de aplicación sigue siendo compatible, podría consolidarse el estado actual como **`2.6.0-rc.1`**. Tras cerrar las pruebas, liberar `2.6.0`. Si el contrato declarado considera incompatible la transición de componentes, correspondería una versión MAYOR, por ejemplo `3.0.0-rc.1`. La decisión debe apoyarse en el contrato real y el historial del proyecto, no aplicarse automáticamente por el nombre de la carpeta.

No se creó una etiqueta Git, una versión instalada ni un archivo VERSION de ejecución durante esta documentación. La sugerencia es una propuesta para el siguiente proceso de liberación.

### 17.3 Un paquete coordinado y versiones de componentes

Conviene una versión principal del producto que identifique exactamente la combinación probada de aplicación Python, frontend, driver, configuración y extensión. Un manifiesto debe registrar también:

- Commit de la aplicación y estado limpio/sucio de construcción.
- SHA256 del driver y versiones ABI ECG/PANI.
- Versión y SHA256 de la extensión y de sus recursos.
- Plataforma destino, arquitectura, SO, Python y dependencias fijadas.
- Fechas de compilación e instalación, dispositivo y paquete de respaldo.
- Compatibilidad con Telsy Web y con las configuraciones de hardware previstas.

La extensión puede mantener su versión propia de componente, vinculada al producto. Su ajuste visual posterior al manifiesto 1.0.0 justificaría evaluar un parche 1.0.1 en la siguiente entrega, sin fingir que esa numeración ya está aplicada.

### 17.4 Archivos recomendados para el repositorio completo

| Archivo propuesto | Propósito |
|---|---|
| `VERSION` o metadato equivalente único | Fuente de versión del aplicativo, consumida por construcción e Información. |
| `CHANGELOG.md` | Cambios por versión: añadidos, correcciones, incompatibilidades y pendientes relevantes. |
| `release-manifest.json` | Componentes, ABI, hashes, commit, plataforma y dependencias de cada entrega. |
| Dependencias fijadas y receta de build | Reconstrucción del entorno Python y del binario C. |
| `docs/` | Arquitectura, pruebas, instalación, reversión y documentación de hardware. |
| Registro de aceptación de versión | Qué se probó, quién aprobó y qué límites quedan. |

Los archivos de esta tabla son recomendaciones; no se crearon en la aplicación con esta tarea.

### 17.5 Flujo de liberación sugerido

1. Consolidar el código completo y las últimas copias de cada componente, manteniendo secretos y datos de pacientes fuera del repositorio.
2. Crear una base Git trazable y commits con cambios concretos. No reconstruir commits históricos ficticios con fechas o autorías inventadas.
3. Desarrollar y revisar cambios; decidir el incremento SemVer por compatibilidad.
4. Preparar una candidata `X.Y.Z-rc.N` con sus componentes, manifiesto y pruebas.
5. Ejecutar regresión de las cuatro mediciones, descarga/recuperación, apagado, batería, Teleconsulta y vídeos en el entorno previsto.
6. Aprobar y etiquetar la entrega con una etiqueta anotada, por ejemplo `v2.6.0`, únicamente si esa versión fue efectivamente adoptada.
7. Publicar un artefacto inmutable con hashes, instrucciones y reversión compatibles. Si cambia contenido después de publicarlo, publicar otra versión.
8. Instalar solo en mantenimiento, registrar la versión anterior/nueva y comprobar el conjunto antes de habilitar mediciones.

El identificador `?v=...` de recursos debe derivarse del build/manifiesto o del contenido. No debe quedar a criterio de cambios manuales inconexos. Información debería mostrar versión del producto, build/commit y fecha coherentes con la instalación, además de diferenciar si existe una actualización realmente válida.

Las candidatas y los metadatos complementan SemVer: `-rc.1` identifica una candidata y `+build.20261017` identifica construcción; los metadatos no convierten dos builds distintos en una nueva precedencia de versión. Para una nueva entrega funcional o corrección se incrementa el campo correspondiente.

## 18. Fuentes y trazabilidad

### 18.1 Evidencia local utilizada

- `CONTEXTO_TELSY_PROYECTO.md`: reconstrucción anterior, útil para el periodo 5–8; sus textos se actualizan aquí con los cambios posteriores.
- `outputs/diagnostico-inicial-2026-10-06.md`.
- `outputs/ecg-proposal/README.md`, `RESULTADOS.md`, fuentes, pruebas e instaladores.
- `outputs/lifecycle-fix/README.md`, `RESULTADOS.md`, fuentes, pruebas e instaladores.
- `outputs/shutdown-button/RESULTADOS.md`.
- `outputs/measurement-finish-flow/RESULTADOS.md`.
- `outputs/teleconsult-battery/ANALISIS_BATERIA.md` y `SOLICITUD_TELECONSULTA.md`.
- `outputs/teleconsult-return-youtube/RESULTADOS.md`.
- Los archivos `INSTALL.txt`, logs, parches y baselines de las etapas posteriores.
- Los scripts de despliegue se leyeron como evidencia; no se ejecutaron para elaborar este documento.
- La conversación aporta requisitos, aprobaciones y resultados de usuario, especialmente las confirmaciones finales del 16 de octubre.

### 18.2 Referencias externas

- [SemVer 2.0.0 — español](https://semver.org/lang/es/): consultada para la recomendación de versionado. La especificación completa es la referencia normativa; las decisiones específicas propuestas para TELSY son recomendaciones de este documento.
- [Ayuda oficial YouTube: inserción y HTTP Referer](https://support.google.com/youtube/answer/171780?hl=en): referencia usada en la corrección del error153.
- [Chrome: content scripts](https://developer.chrome.com/docs/extensions/develop/concepts/content-scripts): mecanismo de la extensión de regreso.
- [MDN: X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Frame-Options) y [frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/frame-ancestors): solicitud de integración pendiente para FCV.
- [Microchip: MCP3021](https://ww1.microchip.com/downloads/en/DeviceDoc/20001805C.pdf): referencia de ADC para el análisis de batería.

Los informes originales pueden describir estados intermedios, como 10 s de envío, el botón superior derecho o el texto Módulo apagado. Para el estado final documentado prevalecen las secciones de este archivo, las últimas copias del Anexo A y las verificaciones fechadas posteriores.

---

<!-- ANEXOS_GENERADOS_LOCALMENTE -->

## Anexo A. Últimas copias locales y huellas
Este mapa identifica **27 archivos de ejecución o recursos** de la última combinación documentada. Las dos configuraciones systemd se detallan por separado: se conoce su contenido por los instaladores, pero no se conserva aquí una copia individual obtenida de cada archivo instalado.
Las huellas siguientes se calcularon el 17 de octubre sobre los bytes de las copias locales existentes. **No se consultó ni comparó el dispositivo por SSH hoy.** Una copia posterior al paquete inicial puede estar en otra carpeta; por eso no se debe escoger una carpeta completa suponiendo que representa toda la versión actual.
| Archivo remoto | Última copia local conservada | SHA256 de la copia local |
|---|---|---|
| `/home/pi/telsy-monitor/telsy/monitor/raspberry/measure.py` | [outputs/lifecycle-fix/files/monitor/raspberry/measure.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/lifecycle-fix/files/monitor/raspberry/measure.py) | `aad2b6c7e6f98c20dd1eee482116ec2eb8f18df271fce48118c03da02e1a4ed5` |
| `/home/pi/telsy-monitor/telsy/monitor/raspberry/driver_KR.py` | [outputs/lifecycle-fix/files/monitor/raspberry/driver_KR.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/lifecycle-fix/files/monitor/raspberry/driver_KR.py) | `18801664c5940f4a0768e7604b758f8dfa77a8db9d582f87dbe3c6ef7f176a12` |
| `/home/pi/telsy-monitor/telsy/monitor/raspberry/ecg_ring.py` | [outputs/lifecycle-fix/files/monitor/raspberry/ecg_ring.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/lifecycle-fix/files/monitor/raspberry/ecg_ring.py) | `4e5c528473ec094d33d32e8175b446babb7109f4a25c59df3dd043aa0c050c5b` |
| `/home/pi/telsy-monitor/telsy/monitor/raspberry/lifecycle.py` | [outputs/measurement-finish-flow/lifecycle.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/measurement-finish-flow/lifecycle.py) | `bbc452576d0458693858c5de5cd0a54f68078385d80104b25aa5ea78f4dafaf5` |
| `/home/pi/telsy-monitor/telsy/monitor/raspberry/geekworm_x728.py` | [outputs/teleconsult-battery/geekworm_x728.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-battery/geekworm_x728.py) | `717d9492c5365f64bc9b9d074bfbd1e9aa262430570f06f0a13dbc6a94fdf957` |
| `/home/pi/telsy-monitor/telsy/monitor/shutdown.py` | [outputs/shutdown-button/files/monitor/shutdown.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/shutdown-button/files/monitor/shutdown.py) | `8e070489c6990882abaade8bdbba20757e17f0d15f2f291ce0a30c650848545b` |
| `/home/pi/telsy-monitor/telsy/monitor/urls.py` | [outputs/shutdown-button/files/monitor/urls.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/shutdown-button/files/monitor/urls.py) | `8fa2195db21acf40d7bd9975b771e826978412b1f6a4ef874cf402474a74c3b6` |
| `/home/pi/telsy-monitor/telsy/monitor/views.py` | [outputs/teleconsult-return-youtube/views.py](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/views.py) | `7cfcf8a53c11c1a00a4fe18b4760cce90b78e69b6a594fb8d21463bc9c8d3753` |
| `/home/pi/telsy-monitor/telsy/vspm-pm6750` | [outputs/lifecycle-fix/candidate](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/lifecycle-fix/candidate) | `9e23bdb78eb9a0696f7d3c0af7af6e9c80da71b0d72058d5fa4d92331097d176` |
| `/home/pi/telsy-monitor/telsy/monitor/static/js/telsy.js` | [outputs/teleconsult-return-youtube/telsy.js](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/telsy.js) | `d6f9acafca85ae4ecf6d8d4fcd54b235e399b9bc2ee86df5fba593d9f45330a3` |
| `/home/pi/telsy-monitor/telsy/monitor/static/js/shutdown.js` | [outputs/shutdown-button/files/monitor/static/js/shutdown.js](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/shutdown-button/files/monitor/static/js/shutdown.js) | `0e8e01dcc3c2ebf56a85cde759995d07d8245f0314db537e4cefa5c71608c549` |
| `/home/pi/telsy-monitor/telsy/monitor/static/js/battery-indicator.js` | [outputs/teleconsult-battery/battery-indicator.js](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-battery/battery-indicator.js) | `e4c40edf84e3debd63659b7c836476e9a4fc1d457e24992add113a6b57923035` |
| `/home/pi/telsy-monitor/telsy/monitor/static/css/battery-indicator.css` | [outputs/teleconsult-battery/battery-indicator.css](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-battery/battery-indicator.css) | `7e9c3f5b369c038d57daea74996fc12bf648bc0c105ef9eadbf6ac30ab0b33d5` |
| `/home/pi/telsy-monitor/telsy/monitor/static/images/icons/measurement-stopped.svg` | [outputs/stopped-button-and-send-delay/measurement-stopped.svg](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/stopped-button-and-send-delay/measurement-stopped.svg) | `89c183d81181b8d7db73b1ae7d6f3485908bdb067da502c8c0bedd823c2bc1b1` |
| `/home/pi/telsy-monitor/telsy/monitor/static/js/teleconsult-navigation.js` | [outputs/teleconsult-return-youtube/teleconsult-navigation.js](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/teleconsult-navigation.js) | `aff4bbe4be22a362a5b6e97e895d4b588fb7d340fcfbc93da534b3729960647b` |
| `/home/pi/telsy-monitor/telsy/monitor/static/js/goals-media.js` | [outputs/teleconsult-return-youtube/goals-media.js](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/goals-media.js) | `17ffa71c409bb941a36d75097ab64ee3f1aa5440e7ea95f16a204b2ddebf5bd6` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/information.html` | [outputs/button-style-adjustments/information.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/button-style-adjustments/information.html) | `374e6aa1ac11fb5ce080df5f77c9814125c3f4d7fc4c0b7396f7e9d1b8471222` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/monitor.html` | [outputs/button-style-adjustments/monitor.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/button-style-adjustments/monitor.html) | `96a11da62752adf703a97a4118f1513b996822177c3b1c2eb4806eac6dc179e0` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/measure.html` | [outputs/stopped-button-and-send-delay/measure.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/stopped-button-and-send-delay/measure.html) | `525f80f04c6e81f5908cdc901c91886142dc309ee197a735c16a9b43639d249c` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/home.html` | [outputs/teleconsult-battery/home.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-battery/home.html) | `06b81bd902c0887aaf27dccaa56e2ca65678ac4a62dfefa95df8339bfba8abec` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/videocall.html` | [outputs/teleconsult-return-youtube/videocall.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/videocall.html) | `1df5cbd18eca1c4a71740783754b5097acbf1d12eadf1a070de838428b8de1d1` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/goalsv.html` | [outputs/teleconsult-return-youtube/goalsv.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/goalsv.html) | `d50d7ea8c3cda577f1ce9e2cad4f9694f90b587c54a9505bc45d04d644289c72` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/goalsd.html` | [outputs/teleconsult-return-youtube/goalsd.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/goalsd.html) | `611261e59adf978dc7b5ee8807543d2ff3206571e83dee35ecbee01a76c02150` |
| `/home/pi/telsy-monitor/telsy/monitor/templates/monitoringinfov.html` | [outputs/teleconsult-return-youtube/monitoringinfov.html](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/monitoringinfov.html) | `9f0ecd389e75fcb198aaf266abcd0660d160e7c2922140b6292efff8a606e7e9` |
| `/home/pi/telsy-monitor/startweb.sh` | [outputs/teleconsult-return-youtube/startweb.sh](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/startweb.sh) | `2a73427d3513d857214136743ed569ae221adb81bddbbf124b310d60ce257a8c` |
| `/home/pi/telsy-monitor/chromium-extensions/telsy-return/manifest.json` | [outputs/teleconsult-return-youtube/extension/manifest.json](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-youtube/extension/manifest.json) | `8d7dda2600a87034b8b0835119c84aade8d147cc4eacf0af227e7be52cfe0527` |
| `/home/pi/telsy-monitor/chromium-extensions/telsy-return/return-button.js` | [outputs/teleconsult-return-position/return-button.js](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/teleconsult-return-position/return-button.js) | `ff841880c721243c74d95bd80d780d43c786cb67b85c3051eb17895cdcd1f5c7` |

### A.1 Fuentes de construcción de la etapa final del driver
Estas copias no se presentan como fuentes sincronizadas en el proyecto remoto; son las conservadas para construir el driver de la etapa lifecycle.
| Archivo | Copia local | SHA256 |
|---|---|---|
| `vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750_main.c` | [outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750_main.c](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750_main.c) | `581325b61b7ca70b6a52c71651f502ecb88e67f9e596306661825ac2c32ac20c` |
| `vspm-pm6750_DRIVER/vspm/common/inc/ecg_ring.h` | [outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/ecg_ring.h](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/ecg_ring.h) | `679e9f32d691c8d83edce3c3389a3e8fea948cd56a24fc4271015cf4bada56bd` |
| `vspm-pm6750_DRIVER/vspm/common/inc/nibp_meta.h` | [outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/nibp_meta.h](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/nibp_meta.h) | `598ad303d42375e87c1870c77be5addb8e8580ca66388bb08e642d100e9e3a57` |

### A.2 Contenido registrado de los drop-ins nuevos
`/etc/systemd/system/telsy-server.service.d/90-ecg-bench.conf`:

```ini
[Service]
StandardOutput=journal
StandardError=journal
```
`/etc/systemd/system/telsy-driver.service.d/11-session-power.conf`:

```ini
[Service]
ExecStartPre=
ExecStartPre=/usr/bin/raspi-gpio set 27 op dl
```
La asignación vacía de ExecStartPre elimina el prearranque anterior de esa lista antes de definir el nuevo. Se conserva la configuración general de los servicios que no se sustituyó mediante esos drop-ins.

## Anexo B. Inventario local de entregables y herramientas
Este inventario es exhaustivo para los archivos locales de `outputs/` y `work/` presentes al documentar, con estas exclusiones: dependencias de terceros instaladas en `work/ssh-libs`, cachés `__pycache__`, bytecode Python y archivos de control del sistema. Las dependencias no son código desarrollado para TELSY; las cachés tampoco representan cambios funcionales. Se listan nombres y tamaños, sin reproducir datos, imágenes o secretos de su contenido.
**Clasificación:** `BASE` = copia anterior/referencia; `FUENTE` = fuente o recurso de implementación de una etapa, no necesariamente la última; `PRUEBA` = prueba/fixture; `HERRAMIENTA` = construcción/despliegue/inspección; `EVIDENCIA` = captura/log/resultado; `DOCUMENTO` = texto técnico; `PAQUETE` = binario, ZIP, instalador o rollback histórico. Estar en esta lista no significa que el archivo se haya instalado como parte de la aplicación. Para eso debe consultarse la sección 12 y el Anexo A.
Los scripts de inspección, prueba o despliegue pueden contactar el equipo o iniciar hardware si se ejecutan. **Se inventarían como archivos; no se ejecutaron para elaborar este documento.** Los paths de esta tabla son relativos al workspace indicado en la sección 3.

**Total inventariado: 360 archivos locales.** Los tamaños se expresan en bytes.

### B · outputs/button-style-adjustments

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/button-style-adjustments/baseline/information.html` | BASE | 5155 |
| `outputs/button-style-adjustments/baseline/monitor.html` | BASE | 5925 |
| `outputs/button-style-adjustments/information-preview.png` | EVIDENCIA | 120504 |
| `outputs/button-style-adjustments/information.html` | FUENTE | 5183 |
| `outputs/button-style-adjustments/information.html.patch` | FUENTE | 2056 |
| `outputs/button-style-adjustments/INSTALL.txt` | EVIDENCIA | 59 |
| `outputs/button-style-adjustments/monitor-preview.png` | EVIDENCIA | 114093 |
| `outputs/button-style-adjustments/monitor.html` | FUENTE | 5957 |
| `outputs/button-style-adjustments/monitor.html.patch` | FUENTE | 1368 |

### B · outputs

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/diagnostico-inicial-2026-10-06.md` | DOCUMENTO | 3512 |
| `outputs/TELSY_ECG_BANCO.zip` | PAQUETE | 90522 |
| `outputs/TELSY_LIFECYCLE_BANCO.zip` | PAQUETE | 115054 |
| `outputs/VALIDACION_DESPLIEGUE_DISTRIBUCION_TELSY.md` | DOCUMENTO | 20372 |

### B · outputs/dialog-cache-fix

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/dialog-cache-fix/INSTALL.txt` | EVIDENCIA | 59 |
| `outputs/dialog-cache-fix/measure.html` | FUENTE | 9035 |
| `outputs/dialog-cache-fix/measure.original.html` | BASE | 9020 |
| `outputs/dialog-cache-fix/telsy.js` | FUENTE | 92106 |
| `outputs/dialog-cache-fix/test_frontend.js` | PRUEBA | 10212 |

### B · outputs/ecg-proposal

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/ecg-proposal/BASELINE_SHA256SUMS.txt` | EVIDENCIA | 266 |
| `outputs/ecg-proposal/candidate` | PAQUETE | 31152 |
| `outputs/ecg-proposal/ecg-integrated.patch` | FUENTE | 26047 |
| `outputs/ecg-proposal/ecg_ring.h` | FUENTE | 1694 |
| `outputs/ecg-proposal/ecg_ring.py` | FUENTE | 5621 |
| `outputs/ecg-proposal/files/monitor/raspberry/driver_KR.py` | FUENTE | 7852 |
| `outputs/ecg-proposal/files/monitor/raspberry/ecg_ring.py` | FUENTE | 5621 |
| `outputs/ecg-proposal/files/monitor/raspberry/measure.py` | FUENTE | 63268 |
| `outputs/ecg-proposal/files/monitor/routing.py` | FUENTE | 651 |
| `outputs/ecg-proposal/files/monitor/static/js/telsy.js` | FUENTE | 85351 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/inc/cli.h` | FUENTE | 496 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/inc/ecg_ring.h` | FUENTE | 1694 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/inc/ipc.h` | FUENTE | 1103 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/inc/jutils.h` | FUENTE | 467 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/inc/serial.h` | FUENTE | 266 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/src/cli.c` | FUENTE | 3213 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/src/ipc.c` | FUENTE | 3388 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/src/jutils.c` | FUENTE | 1134 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/common/src/serial.c` | FUENTE | 2131 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/pm6750/inc/pm6750.h` | FUENTE | 3603 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750.c` | FUENTE | 10312 |
| `outputs/ecg-proposal/files/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750_main.c` | FUENTE | 15850 |
| `outputs/ecg-proposal/install-bench.sh` | PAQUETE | 3779 |
| `outputs/ecg-proposal/README.md` | DOCUMENTO | 8820 |
| `outputs/ecg-proposal/RESULTADOS.md` | DOCUMENTO | 4551 |
| `outputs/ecg-proposal/rollback-bench.sh` | PAQUETE | 1421 |
| `outputs/ecg-proposal/SHA256SUMS.txt` | EVIDENCIA | 3422 |
| `outputs/ecg-proposal/tests/ipc_integration_check.py` | PRUEBA | 1820 |
| `outputs/ecg-proposal/tests/observe_ecg.py` | PRUEBA | 901 |
| `outputs/ecg-proposal/tests/ring_producer.c` | PRUEBA | 865 |
| `outputs/ecg-proposal/tests/smoke_ecg_ws.py` | PRUEBA | 3026 |
| `outputs/ecg-proposal/tests/test_acquisition.py` | PRUEBA | 2081 |
| `outputs/ecg-proposal/tests/test_consumers.py` | PRUEBA | 3449 |
| `outputs/ecg-proposal/tests/test_ecg_ring.py` | PRUEBA | 3402 |
| `outputs/ecg-proposal/tests/test_frontend.js` | PRUEBA | 3143 |
| `outputs/ecg-proposal/transport-draft.patch` | FUENTE | 9231 |

### B · outputs/lifecycle-fix

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/lifecycle-fix/BASELINE_SHA256SUMS.txt` | EVIDENCIA | 459 |
| `outputs/lifecycle-fix/candidate` | PAQUETE | 31224 |
| `outputs/lifecycle-fix/files/monitor/raspberry/driver_KR.py` | FUENTE | 7088 |
| `outputs/lifecycle-fix/files/monitor/raspberry/ecg_ring.py` | FUENTE | 5621 |
| `outputs/lifecycle-fix/files/monitor/raspberry/lifecycle.py` | FUENTE | 11742 |
| `outputs/lifecycle-fix/files/monitor/raspberry/measure.py` | FUENTE | 64878 |
| `outputs/lifecycle-fix/files/monitor/routing.py` | FUENTE | 651 |
| `outputs/lifecycle-fix/files/monitor/static/js/telsy.js` | FUENTE | 91694 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/cli.h` | FUENTE | 496 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/ecg_ring.h` | FUENTE | 1694 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/ipc.h` | FUENTE | 1103 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/jutils.h` | FUENTE | 467 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/nibp_meta.h` | FUENTE | 709 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/inc/serial.h` | FUENTE | 266 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/src/cli.c` | FUENTE | 3213 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/src/ipc.c` | FUENTE | 3388 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/src/jutils.c` | FUENTE | 1134 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/common/src/serial.c` | FUENTE | 2131 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/pm6750/inc/pm6750.h` | FUENTE | 3603 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750.c` | FUENTE | 10312 |
| `outputs/lifecycle-fix/files/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750_main.c` | FUENTE | 16333 |
| `outputs/lifecycle-fix/install-bench.sh` | PAQUETE | 2599 |
| `outputs/lifecycle-fix/lifecycle.patch` | FUENTE | 48899 |
| `outputs/lifecycle-fix/lifecycle.py` | FUENTE | 11742 |
| `outputs/lifecycle-fix/logs/lifecycle-smoke_cancel.py.log` | EVIDENCIA | 301 |
| `outputs/lifecycle-fix/logs/lifecycle-smoke_cancel_notice.py.log` | EVIDENCIA | 552 |
| `outputs/lifecycle-fix/logs/lifecycle-smoke_monitor_cancel.py.log` | EVIDENCIA | 2572 |
| `outputs/lifecycle-fix/logs/lifecycle-smoke_nibp_cancel.py.log` | EVIDENCIA | 336 |
| `outputs/lifecycle-fix/logs/lifecycle-smoke_spo2_sequence.py.log` | EVIDENCIA | 606 |
| `outputs/lifecycle-fix/logs/recovery-lifecycle-smoke_cancel.py.log` | EVIDENCIA | 915 |
| `outputs/lifecycle-fix/logs/recovery-lifecycle-smoke_spo2_sequence.py.log` | EVIDENCIA | 607 |
| `outputs/lifecycle-fix/logs/recovery-lifecycle-smoke_zero_recovery.py.log` | EVIDENCIA | 206 |
| `outputs/lifecycle-fix/nibp_meta.h` | FUENTE | 709 |
| `outputs/lifecycle-fix/README.md` | DOCUMENTO | 6979 |
| `outputs/lifecycle-fix/RESULTADOS.md` | DOCUMENTO | 10452 |
| `outputs/lifecycle-fix/rollback-bench.sh` | PAQUETE | 1236 |
| `outputs/lifecycle-fix/SHA256SUMS.txt` | EVIDENCIA | 4719 |
| `outputs/lifecycle-fix/tests/meta_abi.c` | PRUEBA | 232 |
| `outputs/lifecycle-fix/tests/smoke_cancel.py` | PRUEBA | 3527 |
| `outputs/lifecycle-fix/tests/smoke_cancel_notice.py` | PRUEBA | 11800 |
| `outputs/lifecycle-fix/tests/smoke_ecg_ws.py` | PRUEBA | 3423 |
| `outputs/lifecycle-fix/tests/smoke_monitor_cancel.py` | PRUEBA | 3530 |
| `outputs/lifecycle-fix/tests/smoke_monitor_complete.py` | PRUEBA | 3508 |
| `outputs/lifecycle-fix/tests/smoke_nibp_cancel.py` | PRUEBA | 3526 |
| `outputs/lifecycle-fix/tests/smoke_spo2_sequence.py` | PRUEBA | 14791 |
| `outputs/lifecycle-fix/tests/smoke_zero_recovery.py` | PRUEBA | 3503 |
| `outputs/lifecycle-fix/tests/test_frontend.js` | PRUEBA | 9222 |
| `outputs/lifecycle-fix/tests/test_lifecycle.py` | PRUEBA | 12593 |

### B · outputs/measurement-dialog-copy

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/measurement-dialog-copy/baseline/telsy.js` | BASE | 91694 |
| `outputs/measurement-dialog-copy/change.patch` | FUENTE | 1766 |
| `outputs/measurement-dialog-copy/INSTALL.txt` | EVIDENCIA | 139 |
| `outputs/measurement-dialog-copy/telsy.js` | FUENTE | 92106 |
| `outputs/measurement-dialog-copy/test_frontend.js` | PRUEBA | 9815 |

### B · outputs/measurement-finish-flow

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/measurement-finish-flow/baseline/lifecycle.py` | BASE | 11742 |
| `outputs/measurement-finish-flow/baseline/measure.html` | BASE | 9035 |
| `outputs/measurement-finish-flow/baseline/telsy.js` | BASE | 92106 |
| `outputs/measurement-finish-flow/INSTALL.txt` | EVIDENCIA | 58 |
| `outputs/measurement-finish-flow/lifecycle.py` | FUENTE | 11752 |
| `outputs/measurement-finish-flow/lifecycle.py.patch` | FUENTE | 519 |
| `outputs/measurement-finish-flow/measure.html` | FUENTE | 9035 |
| `outputs/measurement-finish-flow/measure.html.patch` | FUENTE | 338 |
| `outputs/measurement-finish-flow/RESULTADOS.md` | DOCUMENTO | 1702 |
| `outputs/measurement-finish-flow/telsy.js` | FUENTE | 92837 |
| `outputs/measurement-finish-flow/telsy.js.patch` | FUENTE | 6842 |
| `outputs/measurement-finish-flow/test_frontend.js` | PRUEBA | 12001 |
| `outputs/measurement-finish-flow/test_lifecycle.py` | PRUEBA | 12549 |

### B · outputs/monitor-skip-instructions

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/monitor-skip-instructions/change.patch` | FUENTE | 1078 |
| `outputs/monitor-skip-instructions/INSTALL.txt` | EVIDENCIA | 64 |
| `outputs/monitor-skip-instructions/monitor.html` | FUENTE | 5925 |
| `outputs/monitor-skip-instructions/monitor.original.html` | BASE | 5410 |

### B · outputs/shutdown-button

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/shutdown-button/baseline/monitor/static/css/telsy.css` | BASE | 35517 |
| `outputs/shutdown-button/baseline/monitor/templates/information.html` | BASE | 4326 |
| `outputs/shutdown-button/baseline/monitor/urls.py` | BASE | 1432 |
| `outputs/shutdown-button/files/monitor/shutdown.py` | FUENTE | 2021 |
| `outputs/shutdown-button/files/monitor/static/js/shutdown.js` | FUENTE | 1547 |
| `outputs/shutdown-button/files/monitor/templates/information.html` | FUENTE | 5155 |
| `outputs/shutdown-button/files/monitor/urls.py` | FUENTE | 1523 |
| `outputs/shutdown-button/INSTALL.txt` | EVIDENCIA | 63 |
| `outputs/shutdown-button/RESULTADOS.md` | DOCUMENTO | 1153 |
| `outputs/shutdown-button/test_shutdown.js` | PRUEBA | 1472 |
| `outputs/shutdown-button/test_shutdown.py` | PRUEBA | 3369 |

### B · outputs/stopped-button-and-send-delay

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/stopped-button-and-send-delay/baseline/measure.html` | BASE | 9035 |
| `outputs/stopped-button-and-send-delay/baseline/telsy.js` | BASE | 92837 |
| `outputs/stopped-button-and-send-delay/INSTALL.txt` | EVIDENCIA | 57 |
| `outputs/stopped-button-and-send-delay/measure.html` | FUENTE | 9713 |
| `outputs/stopped-button-and-send-delay/measure.html.patch` | FUENTE | 2376 |
| `outputs/stopped-button-and-send-delay/measurement-stopped.svg` | FUENTE | 168 |
| `outputs/stopped-button-and-send-delay/preview.png` | EVIDENCIA | 114967 |
| `outputs/stopped-button-and-send-delay/render_preview.py` | FUENTE | 1195 |
| `outputs/stopped-button-and-send-delay/telsy.js` | FUENTE | 93245 |
| `outputs/stopped-button-and-send-delay/telsy.js.patch` | FUENTE | 1465 |
| `outputs/stopped-button-and-send-delay/test_frontend.js` | PRUEBA | 12341 |

### B · outputs/teleconsult-battery

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/teleconsult-battery/ANALISIS_BATERIA.md` | DOCUMENTO | 3202 |
| `outputs/teleconsult-battery/baseline/geekworm_x728.py` | BASE | 6912 |
| `outputs/teleconsult-battery/baseline/home.html` | BASE | 4207 |
| `outputs/teleconsult-battery/baseline/telsy.js` | BASE | 93245 |
| `outputs/teleconsult-battery/baseline/videocall.html` | BASE | 3751 |
| `outputs/teleconsult-battery/baseline/views.py` | BASE | 10044 |
| `outputs/teleconsult-battery/battery-indicator.css` | FUENTE | 1023 |
| `outputs/teleconsult-battery/battery-indicator.js` | FUENTE | 1307 |
| `outputs/teleconsult-battery/battery-installed.png` | EVIDENCIA | 77871 |
| `outputs/teleconsult-battery/geekworm_x728.py` | FUENTE | 6950 |
| `outputs/teleconsult-battery/home-preview.png` | EVIDENCIA | 16335 |
| `outputs/teleconsult-battery/home.html` | FUENTE | 4715 |
| `outputs/teleconsult-battery/INSTALL_BATTERY.txt` | EVIDENCIA | 57 |
| `outputs/teleconsult-battery/render_preview.py` | FUENTE | 1436 |
| `outputs/teleconsult-battery/SOLICITUD_TELECONSULTA.md` | DOCUMENTO | 7855 |
| `outputs/teleconsult-battery/teleconsult-preview.png` | EVIDENCIA | 39369 |
| `outputs/teleconsult-battery/teleconsult-public-headers.txt` | EVIDENCIA | 514 |
| `outputs/teleconsult-battery/teleconsult.js` | FUENTE | 2009 |
| `outputs/teleconsult-battery/telsy-battery-only.js` | FUENTE | 91235 |
| `outputs/teleconsult-battery/telsy.js` | FUENTE | 90873 |
| `outputs/teleconsult-battery/test_battery.py` | PRUEBA | 2039 |
| `outputs/teleconsult-battery/test_battery_view.py` | PRUEBA | 1318 |
| `outputs/teleconsult-battery/test_measurements.js` | PRUEBA | 12341 |
| `outputs/teleconsult-battery/test_ui.js` | PRUEBA | 2796 |
| `outputs/teleconsult-battery/videocall.html` | FUENTE | 4526 |
| `outputs/teleconsult-battery/views.py` | FUENTE | 10171 |

### B · outputs/teleconsult-return-position

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/teleconsult-return-position/baseline/return-button.js` | BASE | 2029 |
| `outputs/teleconsult-return-position/INSTALL.txt` | EVIDENCIA | 62 |
| `outputs/teleconsult-return-position/preview.png` | EVIDENCIA | 14679 |
| `outputs/teleconsult-return-position/return-button.js` | FUENTE | 2043 |
| `outputs/teleconsult-return-position/verify_browser.js` | PRUEBA | 2619 |

### B · outputs/teleconsult-return-youtube

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `outputs/teleconsult-return-youtube/baseline/goalsd.html` | BASE | 2596 |
| `outputs/teleconsult-return-youtube/baseline/goalsv.html` | BASE | 2350 |
| `outputs/teleconsult-return-youtube/baseline/monitoringinfov.html` | BASE | 2429 |
| `outputs/teleconsult-return-youtube/baseline/startweb.sh` | BASE | 952 |
| `outputs/teleconsult-return-youtube/baseline/telsy.js` | BASE | 91235 |
| `outputs/teleconsult-return-youtube/baseline/videocall.html` | BASE | 3751 |
| `outputs/teleconsult-return-youtube/baseline/views.py` | BASE | 10171 |
| `outputs/teleconsult-return-youtube/extension/manifest.json` | FUENTE | 418 |
| `outputs/teleconsult-return-youtube/extension/return-button.js` | FUENTE | 2029 |
| `outputs/teleconsult-return-youtube/fcv-return-browser.png` | EVIDENCIA | 7506 |
| `outputs/teleconsult-return-youtube/goals-media.js` | FUENTE | 1474 |
| `outputs/teleconsult-return-youtube/goalsd.html` | FUENTE | 2674 |
| `outputs/teleconsult-return-youtube/goalsv.html` | FUENTE | 2527 |
| `outputs/teleconsult-return-youtube/INSTALL.txt` | EVIDENCIA | 66 |
| `outputs/teleconsult-return-youtube/monitoringinfov.html` | FUENTE | 2636 |
| `outputs/teleconsult-return-youtube/reopen_browser.py` | FUENTE | 1026 |
| `outputs/teleconsult-return-youtube/RESULTADOS.md` | DOCUMENTO | 3360 |
| `outputs/teleconsult-return-youtube/startweb.sh` | HERRAMIENTA | 1029 |
| `outputs/teleconsult-return-youtube/teleconsult-navigation.js` | FUENTE | 1242 |
| `outputs/teleconsult-return-youtube/telsy.js` | FUENTE | 90659 |
| `outputs/teleconsult-return-youtube/test_browser.js` | PRUEBA | 2114 |
| `outputs/teleconsult-return-youtube/test_fcv_portal_browser.js` | PRUEBA | 2288 |
| `outputs/teleconsult-return-youtube/test_measurements.js` | PRUEBA | 12341 |
| `outputs/teleconsult-return-youtube/test_ui.js` | PRUEBA | 2386 |
| `outputs/teleconsult-return-youtube/test_youtube_browser.js` | PRUEBA | 2374 |
| `outputs/teleconsult-return-youtube/videocall.html` | FUENTE | 3823 |
| `outputs/teleconsult-return-youtube/views.py` | FUENTE | 10355 |
| `outputs/teleconsult-return-youtube/youtube-browser.png` | EVIDENCIA | 238131 |
| `outputs/teleconsult-return-youtube/youtube-frame-tree.json` | EVIDENCIA | 503 |

### B · work · herramientas y evidencias

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `work/add_cancel_poweroff_ack.py` | HERRAMIENTA | 3402 |
| `work/add_high_pressure_ui_tests.py` | HERRAMIENTA | 3166 |
| `work/add_recovery_retry.py` | HERRAMIENTA | 662 |
| `work/add_recovery_tests.py` | HERRAMIENTA | 3550 |
| `work/add_resume_builder.py` | HERRAMIENTA | 237 |
| `work/add_session_tests.py` | HERRAMIENTA | 1919 |
| `work/add_shutdown_recovery_test.py` | HERRAMIENTA | 705 |
| `work/add_zero_recovery.py` | HERRAMIENTA | 1965 |
| `work/add_zero_ui_builder.py` | HERRAMIENTA | 226 |
| `work/adjust_button_styles.py` | HERRAMIENTA | 1363 |
| `work/adjust_return_position.py` | HERRAMIENTA | 553 |
| `work/append_sequence_results.py` | HERRAMIENTA | 413 |
| `work/apply_recovery_resume_ui.py` | HERRAMIENTA | 3769 |
| `work/apply_zero_ui.py` | HERRAMIENTA | 2970 |
| `work/build_dialog_copy.py` | HERRAMIENTA | 3475 |
| `work/build_finish_flow.py` | HERRAMIENTA | 3817 |
| `work/build_lifecycle.py` | HERRAMIENTA | 14019 |
| `work/build_lifecycle_frontend_test.py` | HERRAMIENTA | 1984 |
| `work/build_proposal.py` | HERRAMIENTA | 2368 |
| `work/build_shutdown.py` | HERRAMIENTA | 1794 |
| `work/build_skip_monitor.py` | HERRAMIENTA | 1181 |
| `work/build_stopped_button.py` | HERRAMIENTA | 3133 |
| `work/build_teleconsult_battery.py` | HERRAMIENTA | 2708 |
| `work/build_teleconsult_return_youtube.py` | HERRAMIENTA | 3583 |
| `work/build_teleconsult_ui.py` | HERRAMIENTA | 3897 |
| `work/c-syntax-check.txt` | EVIDENCIA | 5768 |
| `work/cancel-ack-install.txt` | EVIDENCIA | 55 |
| `work/check_c_syntax.py` | HERRAMIENTA | 1041 |
| `work/check_lifecycle_stage.py` | HERRAMIENTA | 2497 |
| `work/deploy-log.txt` | EVIDENCIA | 524 |
| `work/deploy_battery_ui.ps1` | HERRAMIENTA | 2585 |
| `work/deploy_bench.py` | HERRAMIENTA | 1289 |
| `work/deploy_cancel_ack.py` | HERRAMIENTA | 2097 |
| `work/deploy_cancel_js.py` | HERRAMIENTA | 469 |
| `work/deploy_finish_flow.ps1` | HERRAMIENTA | 2002 |
| `work/deploy_idle_fix.py` | HERRAMIENTA | 1140 |
| `work/deploy_lifecycle.py` | HERRAMIENTA | 1304 |
| `work/deploy_recovery_final.py` | HERRAMIENTA | 889 |
| `work/deploy_shutdown.py` | HERRAMIENTA | 2499 |
| `work/deploy_teleconsult_youtube.ps1` | HERRAMIENTA | 3000 |
| `work/deploy_ui_recovery_resume.py` | HERRAMIENTA | 2044 |
| `work/deploy_ui_recovery_retry.py` | HERRAMIENTA | 2105 |
| `work/deploy_warmup.py` | HERRAMIENTA | 1044 |
| `work/deploy_zero_fix.py` | HERRAMIENTA | 1054 |
| `work/dialog-copy-review/page-1.png` | EVIDENCIA | 211389 |
| `work/document_cancel_notice.py` | HERRAMIENTA | 1470 |
| `work/document_ui_resume.py` | HERRAMIENTA | 2730 |
| `work/document_zero_recovery.py` | HERRAMIENTA | 3006 |
| `work/extend_smoke_runner.py` | HERRAMIENTA | 260 |
| `work/fetch_shutdown.py` | HERRAMIENTA | 609 |
| `work/fetch_sources.py` | HERRAMIENTA | 935 |
| `work/final-stage-console.txt` | EVIDENCIA | 18053 |
| `work/final_session_report.py` | HERRAMIENTA | 732 |
| `work/finalize_cancel_notice.py` | HERRAMIENTA | 574 |
| `work/finalize_lifecycle.py` | HERRAMIENTA | 704 |
| `work/finish_shutdown_ui.py` | HERRAMIENTA | 850 |
| `work/fix_builder_indent.py` | HERRAMIENTA | 412 |
| `work/fix_cancel_alert.py` | HERRAMIENTA | 333 |
| `work/fix_dialog_cache.py` | HERRAMIENTA | 1371 |
| `work/fix_idle.py` | HERRAMIENTA | 1060 |
| `work/fix_session_proof.py` | HERRAMIENTA | 5011 |
| `work/fix_shutdown_error_recovery.py` | HERRAMIENTA | 448 |
| `work/fix_terminal_zero.py` | HERRAMIENTA | 1995 |
| `work/inspect_telsy.py` | HERRAMIENTA | 824 |
| `work/integrate_consumers.py` | HERRAMIENTA | 7529 |
| `work/lifecycle-check-log.txt` | EVIDENCIA | 18092 |
| `work/lifecycle-deploy-log.txt` | EVIDENCIA | 480 |
| `work/lifecycle-smoke_cancel.py.log` | EVIDENCIA | 915 |
| `work/lifecycle-smoke_cancel_notice.py.log` | EVIDENCIA | 552 |
| `work/lifecycle-smoke_ecg_ws.py.log` | EVIDENCIA | 238 |
| `work/lifecycle-smoke_monitor_cancel.py.log` | EVIDENCIA | 2572 |
| `work/lifecycle-smoke_nibp_cancel.py.log` | EVIDENCIA | 336 |
| `work/lifecycle-smoke_spo2_sequence.py.log` | EVIDENCIA | 607 |
| `work/lifecycle-smoke_zero_recovery.py.log` | EVIDENCIA | 206 |
| `work/lifecycle-stage-console.txt` | EVIDENCIA | 5170 |
| `work/lifecycle-stage.txt` | EVIDENCIA | 29 |
| `work/manual-1.png` | EVIDENCIA | 166195 |
| `work/manual-2.png` | EVIDENCIA | 230909 |
| `work/manual-3.png` | EVIDENCIA | 280439 |
| `work/manual-4.png` | EVIDENCIA | 164755 |
| `work/manual-5.png` | EVIDENCIA | 122608 |
| `work/manual.txt` | EVIDENCIA | 12741 |
| `work/monitor-skip/installed-preview.png` | EVIDENCIA | 130320 |
| `work/monitor-skip/p1-Image10.jpg` | EVIDENCIA | 98387 |
| `work/monitor-skip/p1-Image9.png` | EVIDENCIA | 74826 |
| `work/monitor-skip/p2-Image13.png` | EVIDENCIA | 124868 |
| `work/monitor-skip/p2-Image14.jpg` | EVIDENCIA | 52379 |
| `work/monitor-skip/p3-Image17.png` | EVIDENCIA | 189949 |
| `work/monitor-skip/p3-Image18.png` | EVIDENCIA | 111821 |
| `work/monitor-skip/p4-Image21.png` | EVIDENCIA | 160988 |
| `work/monitor-skip/p4-Image22.jpg` | EVIDENCIA | 48692 |
| `work/monitor-skip/p5-Image25.png` | EVIDENCIA | 149449 |
| `work/monitor-skip/p5-Image26.png` | EVIDENCIA | 152238 |
| `work/monitor-skip/reference-2.png` | EVIDENCIA | 228569 |
| `work/monitor-skip/reference-5.png` | EVIDENCIA | 212174 |
| `work/monitor-skip/telsy.js` | HERRAMIENTA | 91694 |
| `work/package_lifecycle.py` | HERRAMIENTA | 1883 |
| `work/package_release.py` | HERRAMIENTA | 2714 |
| `work/prepare_cancel.py` | HERRAMIENTA | 643 |
| `work/prepare_cancel_notice_smoke.py` | HERRAMIENTA | 1887 |
| `work/prepare_finish_tests.py` | HERRAMIENTA | 3194 |
| `work/prepare_late_cancel.py` | HERRAMIENTA | 205 |
| `work/prepare_lifecycle_smoke.py` | HERRAMIENTA | 936 |
| `work/prepare_monitor60.py` | HERRAMIENTA | 372 |
| `work/prepare_monitor_complete.py` | HERRAMIENTA | 445 |
| `work/prepare_nibp_cancel.py` | HERRAMIENTA | 434 |
| `work/prepare_spo2_sequence.py` | HERRAMIENTA | 892 |
| `work/prepare_stopped_tests.py` | HERRAMIENTA | 1314 |
| `work/prepare_ui_update.py` | HERRAMIENTA | 406 |
| `work/prepare_zero_deploy.py` | HERRAMIENTA | 777 |
| `work/prepare_zero_recovery_smoke.py` | HERRAMIENTA | 1144 |
| `work/probe_idle.py` | HERRAMIENTA | 801 |
| `work/probe_stop.py` | HERRAMIENTA | 423 |
| `work/remote-stage.txt` | EVIDENCIA | 28 |
| `work/run_lifecycle_smoke.py` | HERRAMIENTA | 1419 |
| `work/run_smoke.py` | HERRAMIENTA | 933 |
| `work/separate_battery_deploy.py` | HERRAMIENTA | 853 |
| `work/smoke-log.txt` | EVIDENCIA | 238 |
| `work/stage-test-log.txt` | EVIDENCIA | 19105 |
| `work/stage_and_test.py` | HERRAMIENTA | 2754 |
| `work/teleconsult-review/page-1.png` | EVIDENCIA | 307619 |
| `work/test_acquisition.py` | PRUEBA | 2074 |
| `work/test_consumers.py` | PRUEBA | 3436 |
| `work/test_ecg_ring.py` | PRUEBA | 3395 |
| `work/test_frontend.js` | PRUEBA | 3129 |
| `work/test_lifecycle.py` | PRUEBA | 12767 |
| `work/test_recovery_retry.py` | PRUEBA | 780 |
| `work/test_shutdown_remote.py` | PRUEBA | 810 |
| `work/trace_nibp.py` | HERRAMIENTA | 1388 |
| `work/ui-recovery-install.txt` | EVIDENCIA | 210 |
| `work/update_idle_results.py` | HERRAMIENTA | 2442 |
| `work/update_lifecycle_builder.py` | HERRAMIENTA | 890 |
| `work/update_modal_tests.py` | HERRAMIENTA | 430 |
| `work/update_session_results.py` | HERRAMIENTA | 2180 |
| `work/update_zero_frontend_tests.py` | HERRAMIENTA | 652 |
| `work/verification-log.txt` | EVIDENCIA | 7970 |
| `work/verify_deployment.py` | PRUEBA | 1614 |
| `work/verify_session_install.py` | PRUEBA | 1311 |
| `work/write_lifecycle_results.py` | HERRAMIENTA | 2682 |

### B · work/baseline

| Archivo local | Tipo | Bytes |
|---|---|---:|
| `work/baseline/monitor/raspberry/driver_KR.py` | BASE | 7852 |
| `work/baseline/monitor/raspberry/measure.py` | BASE | 66052 |
| `work/baseline/monitor/routing.py` | BASE | 651 |
| `work/baseline/monitor/static/js/telsy.js` | BASE | 83595 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/inc/cli.h` | BASE | 496 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/inc/ipc.h` | BASE | 1103 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/inc/jutils.h` | BASE | 467 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/inc/serial.h` | BASE | 266 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/src/cli.c` | BASE | 3213 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/src/ipc.c` | BASE | 3388 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/src/jutils.c` | BASE | 1134 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/common/src/serial.c` | BASE | 2131 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/pm6750/inc/pm6750.h` | BASE | 3603 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750.c` | BASE | 10312 |
| `work/baseline/vspm-pm6750_DRIVER/vspm/pm6750/src/pm6750_main.c` | BASE | 15013 |

### B · Documentos raíz relacionados

- [CONTEXTO_TELSY_PROYECTO.md](C:/Users/Diego/Documents/ChatGPT/Telsy_v2.5/CONTEXTO_TELSY_PROYECTO.md): contexto histórico previo, conservado sin sustituirlo por este archivo.

**Cierre documental:** última prueba de usuario registrada el 16 de octubre de 2026; nueva documentación del 17 de octubre. La versión oficial del producto y la liberación de una entrega consolidada permanecen como decisiones pendientes del responsable del proyecto.
