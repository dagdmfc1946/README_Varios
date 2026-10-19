# Guía Completa de Transferencia de Archivos por Terminal

Esta guía detalla los métodos para transferir archivos y carpetas entre distintos sistemas operativos (Windows y Linux/Raspberry Pi), la sintaxis según la terminal utilizada (CMD, PowerShell, Git Bash) y el manejo de archivos comprimidos `.tar` y `.tar.gz`.

---

## 1. Requisitos Previos Generales

1. **SSH Habilitado:** La Raspberry Pi de origen o destino debe tener activo el servicio SSH.
   ```bash
   sudo systemctl enable --now ssh
   ```
2. **Dirección IP o Nombre de Host:** Debes conocer la IP local de los dispositivos (ejemplo: `172.16.70.41`).
3. **Herramienta SCP / Rsync:** 
   * `scp` viene integrado nativamente en Windows 10/11 y sistemas Linux.
   * `rsync` se usa principalmente entre sistemas Linux por su rendimiento y opciones avanzadas de filtrado.

---

## 2. Reglas para Rutas, Espacios y Terminales

El manejo de comillas y barras en las rutas es la causa principal de errores en la terminal.

### Regla de Oro para Espacios
Si cualquier parte de la ruta local o remota contiene espacios (ej. `Mis Documentos`), **toda la ruta debe ir entre comillas dobles `""`**.

### Manejo de Barras según la Terminal

| Terminal | Formato de Ruta Local | Manejo de Espacios y Comillas | Error Común |
| :--- | :--- | :--- | :--- |
| **CMD** | `C:\Ruta\Carpeta` o `C:/Ruta/Carpeta` | `"C:\Ruta Con Espacio"` | **¡OJO!** Colocar `\"` al final (ej. `"C:\Ruta\"`) escapa la comilla y causa error. Usa `"C:\Ruta"` o `"C:/Ruta/"`. |
| **PowerShell** | `C:\Ruta\Carpeta` o `C:/Ruta/Carpeta` | `"C:\Ruta Con Espacio"` | Mismo problema con `\"` al final. Quita la última barra diagonal inversa antes de la comilla. |
| **Git Bash** | `/c/Ruta/Carpeta` | `"/c/Ruta Con Espacio"` | Usa sintaxis estilo Linux (`/c/Users/...` en lugar de `C:\Users\...`). |

---

## 3. Casos de Transferencia

### Caso A: De Raspberry Pi a Windows

Ejecuta el comando **desde la terminal de Windows** (CMD, PowerShell o Git Bash).

#### 1. Archivo único o múltiples archivos
* **CMD / PowerShell:**
  ```cmd
  scp pi@172.16.70.41:/home/pi/telsy-monitor/startweb.sh "C:\Users\diegogarcia\Documents\proyectosQM_Repositorio\TAR_dagd"
  ```
* **Git Bash:**
  ```bash
  scp pi@172.16.70.41:/home/pi/telsy-monitor/startweb.sh "/c/Users/diegogarcia/Documents/proyectosQM_Repositorio/TAR_dagd"
  ```

#### 2. Carpeta completa (Parámetro `-r`)
* **CMD / PowerShell:**
  ```cmd
  scp -r pi@172.16.70.41:/home/pi/telsy-monitor "C:\Users\diegogarcia\Mis Documentos\Proyectos"
  ```

---

### Caso B: De Windows a Raspberry Pi

Ejecuta el comando **desde la terminal de Windows**.

#### 1. Archivo único
* **CMD / PowerShell:**
  ```cmd
  scp "C:\Users\diegogarcia\Documentos\script.sh" pi@172.16.70.41:/home/pi/
  ```

#### 2. Carpeta completa (Parámetro `-r`)
* **CMD / PowerShell:**
  ```cmd
  scp -r "C:\Users\diegogarcia\Documents\proyectosQM_Repositorio\TAR_dagd" pi@172.16.70.41:/home/pi/
  ```

---

### Caso C: De Raspberry Pi a Raspberry Pi

Ejecuta el comando **desde la Raspberry Pi de origen**.

#### Opción 1: Usando `scp`
* **Copiar carpeta entera:**
  ```bash
  scp -r /home/pi/telsy-monitor pi@172.16.70.44:/home/pi/
  ```
* **Si la ruta tiene espacios:**
  ```bash
  scp -r "/home/pi/Mi Carpeta" pi@172.16.70.44:"/home/pi/Carpeta Destino"
  ```

#### Opción 2: Usando `rsync` (Recomendado entre Linux)
`rsync` muestra progreso, es más rápido y permite excluir carpetas.

* **Copiar carpeta completa:**
  ```bash
  rsync -avzP /home/pi/telsy-monitor pi@172.16.70.44:/home/pi/
  ```

* **Copiar excluyendo una subcarpeta (ejemplo: `telsy_venv`):**
  ```bash
  rsync -avzP --exclude='telsy_venv' /home/pi/telsy-monitor pi@172.16.70.44:/home/pi/
  ```

---

### Caso D: De Windows a Windows

#### Opción 1: Entre carpetas locales o discos (CMD / PowerShell)
* **Usando `robocopy` (CMD / PowerShell):**
  ```cmd
  robocopy "C:\Ruta Origen" "D:\Ruta Destino" /E /Z
  ```
* **Usando `Copy-Item` (PowerShell):**
  ```powershell
  Copy-Item -Path "C:\Ruta Origen\*" -Destination "D:\Ruta Destino" -Recurse
  ```

#### Opción 2: Entre dos equipos Windows en red local vía SCP
*(Requiere que el equipo de destino tenga activado el "Servicio de servidor OpenSSH" en Windows).*

```cmd
scp -r "C:\Users\diegogarcia\Proyecto" usuario@192.168.1.100:"C:\Users\OtroUsuario\Destino"
```

---

## 4. Compresión y Descompresión en Terminal (`.tar` y `.tar.gz`)

El uso de archivos comprimidos acelera notablemente la transferencia SSH cuando se trabaja con miles de archivos pequeños.

### Explicación de Banderas Principales
* `-c` : Create (Crear archivo).
* `-x` : Extract (Extraer/Descomprimir).
* `-z` : gzip (Aplicar o procesar compresión gzip).
* `-v` : Verbose (Muestra la lista de archivos procesados).
* `-f` : File (Especifica el nombre del archivo de salida/origen).
* `-C` : Dir (Especifica la carpeta destino de extracción).

---

### Archivos `.tar` (Empaquetado sin compresión)

1. **Crear `.tar` de una carpeta:**
   ```bash
   tar -cvf proyecto.tar /ruta/a/la/carpeta
   ```
2. **Extraer `.tar` en la carpeta actual:**
   ```bash
   tar -xvf proyecto.tar
   ```
3. **Extraer `.tar` en un directorio específico:**
   ```bash
   tar -xvf proyecto.tar -C /ruta/destino/
   ```

---

### Archivos `.tar.gz` (Empaquetado + Compresión Gzip)

1. **Crear `.tar.gz` de una carpeta:**
   ```bash
   tar -czvf proyecto.tar.gz /home/pi/telsy-monitor
   ```

2. **Crear `.tar.gz` EXCLUYENDO una subcarpeta:**
   ```bash
   tar --exclude='telsy-monitor/telsy_venv' -czvf proyecto.tar.gz telsy-monitor
   ```

3. **Extraer `.tar.gz` en la carpeta actual:**
   ```bash
   tar -xzvf proyecto.tar.gz
   ```

4. **Extraer `.tar.gz` en un directorio específico:**
   ```bash
   tar -xzvf proyecto.tar.gz -C /home/pi/destino/
   ```