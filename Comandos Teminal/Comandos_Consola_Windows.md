# Guía Rápida de Comandos para Windows (CMD y PowerShell)

Esta guía contiene los comandos esenciales y más utilizados para navegar y administrar el sistema a través de la consola de Windows (**Símbolo del sistema / CMD**) y **PowerShell**.

---

## 1. Navegación de Archivos y Directorios

| Acción | Comando en CMD | Comando en PowerShell | Notas |
| :--- | :--- | :--- | :--- |
| **Ver el contenido** | `dir` | `ls` o `Get-ChildItem` | Muestra archivos y carpetas de la ubicación actual. |
| **Cambiar de carpeta** | `cd "Ruta\Carpeta"` | `cd "Ruta\Carpeta"` | Usa comillas si el nombre tiene espacios. |
| **Cambiar de disco y carpeta** | `cd /d "D:\Ruta"` | `cd "D:\Ruta"` | En CMD se requiere `/d` obligatoriamente. |
| **Subir de nivel (atrás)** | `cd ..` | `cd ..` | Regresa a la carpeta contenedora o superior. |
| **Ir a la raíz del disco** | `cd \` | `cd \` | Te lleva directamente al inicio del disco actual (ej. `C:\`). |

---

## 2. Gestión de Archivos y Carpetas

| Acción | Comando en CMD | Comando en PowerShell |
| :--- | :--- | :--- |
| **Crear una carpeta** | `mkdir NombreCarpeta` | `mkdir NombreCarpeta` o `New-Item -ItemType Directory` |
| **Crear un archivo vacío** | `type nul > archivo.txt` | `New-Item archivo.txt` o `echo $null > archivo.txt` |
| **Copiar un archivo** | `copy origen.txt destino.txt` | `cp origen.txt destino.txt` o `Copy-Item` |
| **Mover o renombrar** | `move archivo.txt nueva_ruta\` | `mv archivo.txt nueva_ruta\` o `Move-Item` |
| **Eliminar un archivo** | `del archivo.txt` | `rm archivo.txt` o `Remove-Item` |
| **Eliminar una carpeta** | `rmdir /s NombreCarpeta` | `rm -Recurse NombreCarpeta` |

---

## 3. Información del Sistema y Red

| Acción | Comando en CMD | Comando en PowerShell |
| :--- | :--- | :--- |
| **Ver configuración de red** | `ipconfig` | `ipconfig` o `Get-NetIPAddress` |
| **Probar conectividad** | `ping google.com` | `ping google.com` o `Test-Connection` |
| **Información del sistema** | `systeminfo` | `systeminfo` o `Get-ComputerInfo` |
| **Ver procesos activos** | `tasklist` | `ps` o `Get-Process` |
| **Cerrar un proceso forzado** | `taskkill /f /im nombre.exe` | `Stop-Process -Name nombre` |

---

## 4. Trucos de Productividad

* **Limpiar la pantalla:** Escribe `cls` en cualquiera de las dos consolas.
* **Autocompletado:** Escribe las primeras letras de una carpeta o archivo y presiona la tecla **Tabulador (Tab)** para que la consola lo complete automáticamente.
* **Historial de comandos:** Usa las **flechas arriba y abajo** del teclado para navegar por los comandos que ya has escrito antes.
* **Cancelar comando:** Si un comando se queda pegado o tarda mucho, presiona **Ctrl + C** para detenerlo inmediatamente.

---

This is for informational purposes only. For medical advice or diagnosis, consult a professional. AI responses may include mistakes.
