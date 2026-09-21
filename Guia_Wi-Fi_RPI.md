# Guía de Configuración Wi-Fi vía Terminal en Raspberry Pi (NetworkManager / nmcli)

Esta guía documenta los procedimientos técnicos para preconfigurar y conectar una Raspberry Pi (incluyendo módulos CM4) a una red inalámbrica utilizando la herramienta `nmcli`, cubriendo escenarios donde el SSID es visible o invisible en el entorno local.

---

## 🛠️ Requisitos Previos
* Acceso a la terminal de la Raspberry Pi (vía SSH, puerto Serial o pantalla local).
* Permisos de administrador (`sudo`).
* Interfaz inalámbrica activa (por lo general `wlan0`).

---

## 1. Conexión Estándar (SSID Detectado en Laboratorio)
Utilice este método si la red Wi-Fi está al alcance de la Raspberry Pi durante la configuración y su SSID es visible.

```bash
sudo nmcli dev wifi connect "NOMBRE_DE_TU_WIFI" password "TU_CONTRASEÑA"
```

### Verificación de Estado
Para confirmar que el dispositivo se ha asociado y ha obtenido una dirección IP:
```bash
nmcli device status
```

---

## 2. Pre-Aprovisionamiento / SSID No Detectado (Contacto Cero)
Utilice este método para guardar las credenciales de una red que **no está presente en el laboratorio**, garantizando que el dispositivo se conecte de forma 100% automática e independiente inmediatamente al encenderse en su destino final.

### Paso 1: Crear el perfil de conexión inalámbrica
Este comando inyecta la configuración directamente en el almacenamiento de NetworkManager de forma persistente:
```bash
sudo nmcli connection add type wifi ifname wlan0 con-name "NOMBRE_DE_TU_WIFI" ssid "NOMBRE_DE_TU_WIFI" autoconnect yes -- wifi-sec.key-mgmt wpa-psk wifi-sec.psk "TU_CONTRASEÑA"
```

### Paso 2: Configurar reintentos infinitos de conexión
Por defecto, si NetworkManager no encuentra la red, puede desistir tras unos intentos. El valor `0` obliga al sistema a buscar la red en bucle infinito hasta encontrarla:
```bash
sudo nmcli connection modify "NOMBRE_DE_TU_WIFI" connection.autoconnect-retries 0
```

### Paso 3 (Opcional): Si la red de destino tiene el SSID oculto
Si el cliente final tiene configurada su red como oculta (no transmite el SSID), añada esta directiva al perfil creado:
```bash
sudo nmcli connection modify "NOMBRE_DE_TU_WIFI" 802-11-wireless.hidden yes
```

---

## 3. Comandos de Verificación y Diagnóstico

### Listar todos los perfiles guardados en el sistema
Permite comprobar que el perfil fue creado con éxito (aparecerá en la lista aunque la red no exista localmente):
```bash
nmcli connection show
```

### Eliminar un perfil de red erróneo o antiguo
Si se comete un error en el SSID o la contraseña, elimine el perfil antes de crearlo nuevamente:
```bash
sudo nmcli connection delete "NOMBRE_DE_TU_WIFI"
```

### Levantar la conexión manualmente (cuando la red ya esté disponible)
```bash
sudo nmcli connection up "NOMBRE_DE_TU_WIFI"
```

---

## 📦 Flujo de Despliegue Recomendado en Producción
1. **Ejecutar en Laboratorio:** Aplicar los comandos de la **Sección 2** con los datos exactos del cliente.
2. **Validar Perfil:** Asegurar con `nmcli connection show` que el nombre asignado en `con-name` esté en la base de datos.
3. **Apagado Seguro:** Apagar el equipo con `sudo poweroff` antes de empacar.
4. **Entrega:** El cliente final solo deberá energizar el dispositivo; la conexión se establecerá de forma transparente en segundo plano.
