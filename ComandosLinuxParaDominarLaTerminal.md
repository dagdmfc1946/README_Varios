# 🐧 GUÍA ESENCIAL: COMANDOS LINUX PARA DOMINAR LA TERMINAL

A continuación se listan los comandos esenciales de Linux para administrar cualquier distribución (como Debian o Ubuntu) desde la línea de comandos. Esta guía cubre:

- 📂 Navegación por el sistema de archivos.
- 📝 Creación, copia, movimiento y eliminación de archivos/directorios.
- 🔐 Gestión de usuarios y permisos.
- 📦 Administración de paquetes.
- ⚙️ Monitorización y optimización del sistema.
- 🌐 Transferencia de archivos y respaldos.

---

## 📂 NAVEGACIÓN Y ARCHIVOS

### pwd
📍 **P**rint **W**orking **D**irectory
Muestra la ruta absoluta del directorio actual en el que te encuentras.
```bash
pwd 
```

### ls
📁 **L**i**s**t
Lista archivos y directorios dentro de la ubicación actual o la especificada.
```bash
ls          # Listado básico
ls -l       # Listado detallado (permisos, propietario, tamaño, fecha)
ls -la      # Listado detallado incluyendo archivos ocultos (los que empiezan por punto)
ls -lh      # Listado detallado con tamaño legible para humanos (K, M, G)
```

### cd
🔄 **C**hange **D**irectory
Cambia el directorio de trabajo actual.
```bash
cd /              # Navega a la raíz del sistema
cd /var/log       # Navega a una ruta absoluta específica
cd ..             # Sube un nivel en la jerarquía de directorios
cd ~              # Navega al directorio "home" del usuario actual
```

### clear
🧹 **Clear**
Limpia la pantalla de la terminal, ocultando los comandos anteriores.
```bash
clear
```
*(Atajo de teclado: `Ctrl + L`)*

### tree
🌲 **Tree**
Muestra de forma jerárquica (en árbol) la estructura de directorios y archivos. *Nota: puede requerir instalación (`sudo apt install tree`).*
```bash
tree              # Muestra el árbol desde el directorio actual
tree -L 2         # Limita la profundidad del árbol a 2 niveles
```

### mkdir
📁 **M**a**k**e **Dir**ectory
Crea nuevos directorios.
```bash
mkdir nuevo_directorio
mkdir "Nuevo Directorio"  # Uso de comillas si el nombre contiene espacios
mkdir -p /ruta/a/nueva/carpeta  # Crea directorios padre si no existen
```

### touch
📄 **Touch**
Crea archivos vacíos o actualiza la fecha de modificación de un archivo existente.
```bash
touch archivo.txt
touch config.json script.sh  # Crea múltiples archivos a la vez
```

### echo
🗣️ **Echo**
Imprime texto en la terminal o lo redirige a un archivo.
```bash
echo "Hola Mundo"
echo "Texto de prueba" > archivo.txt   # Sobrescribe el archivo con el texto
echo "Nueva línea" >> archivo.txt      # Añade el texto al final del archivo sin sobrescribir
```

### cp
📋 **C**o**p**y
Copia archivos y directorios de un origen a un destino.
```bash
cp archivo.txt copia.txt               # Copia un archivo en la misma ruta
cp archivo.txt /ruta/destino/          # Copia un archivo a otro directorio
cp -r carpeta_origen/ carpeta_destino/ # Copia un directorio y todo su contenido de forma recursiva
```

### mv
🚚 **M**o**v**e
Mueve archivos/directorios a otra ubicación o los renombra.
```bash
mv archivo.txt /ruta/destino/          # Mueve el archivo
mv nombre_viejo.txt nombre_nuevo.txt   # Renombra el archivo en la misma ubicación
```

### rm
🗑️ **R**e**m**ove
Elimina archivos o directorios permanentemente (no van a la papelera).
```bash
rm archivo.txt                 # Elimina un archivo
rm -r directorio/              # Elimina un directorio y su contenido (recursivo)
rm -rf directorio/             # Elimina forzosamente sin pedir confirmación (¡Usar con precaución!)
```

---

## 🔍 VISUALIZACIÓN Y BÚSQUEDA

### cat
🐈 **C**oncaten**at**e
Muestra el contenido completo de un archivo directamente en la terminal.
```bash
cat archivo.txt
cat archivo1.txt archivo2.txt > combinados.txt # Une dos archivos en uno nuevo
```

### less
📖 **Less**
Permite visualizar y paginar el contenido de archivos grandes sin cargar todo en memoria.
```bash
less syslog.log
```
*(Usa las flechas para subir/bajar, `q` para salir, `/palabra` para buscar).*

### head
🔝 **Head**
Muestra las primeras líneas de un archivo (10 por defecto).
```bash
head archivo.txt
head -n 20 archivo.txt   # Muestra las primeras 20 líneas
```

### tail
🔚 **Tail**
Muestra las últimas líneas de un archivo. Muy útil para leer logs en tiempo real.
```bash
tail archivo.txt
tail -n 50 archivo.txt   # Muestra las últimas 50 líneas
tail -f /var/log/syslog  # Sigue el archivo en tiempo real a medida que se actualiza
```

### grep
🕵️ **G**lobal **R**egular **E**xpression **P**rint
Busca patrones de texto o palabras dentro de archivos o salidas de otros comandos.
```bash
grep "error" syslog.log            # Busca la palabra "error" en el archivo
grep -i "error" syslog.log         # Búsqueda ignorando mayúsculas/minúsculas
cat archivo.txt | grep "palabra"   # Filtra la salida de cat
```

---

## ⚙️ INFORMACIÓN Y RECURSOS DEL SISTEMA

### top
📊 **Top**
Muestra el administrador de tareas de la terminal: procesos, uso de CPU y RAM en tiempo real.
```bash
top
```
*(Presiona `q` para salir).*

### htop
📈 **Htop**
Versión interactiva y visualmente más amigable de `top`. *Nota: puede requerir instalación.*
```bash
htop
```

### df
💽 **D**isk **F**ree
Muestra el espacio disponible y utilizado en las particiones de los discos.
```bash
df -h      # Muestra tamaños legibles (MB, GB)
df -Th     # Incluye el tipo de sistema de archivos (ext4, fat32, etc.)
```

### free
🧠 **Free**
Muestra la cantidad de memoria RAM libre, usada y la memoria Swap.
```bash
free -h    # Muestra valores legibles (MB, GB)
```

### uname
🖥️ **U**nix **Name**
Muestra información sobre el sistema operativo y el kernel de Linux.
```bash
uname -a   # Muestra toda la información disponible (kernel, arquitectura, hostname)
uname -r   # Muestra solo la versión del kernel
```

---

## 📦 GESTIÓN DE PAQUETES (APT)

### apt update
🔄 **Update**
Descarga la información más reciente de los repositorios. Actualiza el "catálogo", pero no instala nada.
```bash
sudo apt update
```

### apt upgrade
⬆️ **Upgrade**
Instala las versiones más nuevas de todos los paquetes actualmente instalados en el sistema.
```bash
sudo apt upgrade
sudo apt upgrade -y    # Confirma automáticamente la instalación
```

### apt install
📥 **Install**
Descarga e instala un nuevo paquete/programa y sus dependencias.
```bash
sudo apt install nmap
sudo apt install git curl htop  # Instala múltiples paquetes
```

### apt remove
🗑️ **Remove**
Desinstala un paquete del sistema, conservando sus archivos de configuración global.
```bash
sudo apt remove nmap
sudo apt purge nmap    # Elimina el paquete y también sus archivos de configuración
```

---

## 🔐 USUARIOS Y PERMISOS

### whoami
👤 **Who am I?**
Imprime el nombre del usuario con el que estás logueado actualmente en la terminal.
```bash
whoami
```

### chmod
🔑 **Ch**ange **Mod**e
Cambia los permisos de lectura (r), escritura (w) y ejecución (x) de un archivo o directorio.
```bash
chmod +x script.sh          # Otorga permisos de ejecución al propietario
chmod 666 archivo.txt       # Lectura y escritura para todos (owner, group, others)
chmod 755 script.sh         # Owner: leer/escribir/ejecutar. Otros: leer/ejecutar
```

### chown
👑 **Ch**ange **Own**er
Cambia el usuario y/o grupo propietario de un archivo o directorio.
```bash
sudo chown root archivo.txt           # Cambia el propietario a root
sudo chown pi:www-data directorio/    # Cambia usuario a 'pi' y grupo a 'www-data'
sudo chown -R pi:pi directorio/       # Aplica el cambio de forma recursiva a todo el contenido
```

### passwd
🔒 **Passw**or**d**
Permite cambiar la contraseña de un usuario.
```bash
passwd              # Cambia tu propia contraseña
sudo passwd root    # Cambia la contraseña del usuario root
```

### sudo
🦸 **S**uper**u**ser **Do**
Permite ejecutar comandos con privilegios administrativos (root).
```bash
sudo rm archivo_protegido.txt
sudo nano /etc/hosts
```

### sudo su / su
🦹 **S**witch **U**ser
Abre una sesión interactiva permanente como otro usuario (por defecto root) para no tener que escribir `sudo` en cada comando.
```bash
sudo su             # Inicia sesión como root utilizando tu contraseña (sudo)
su -                # Inicia sesión como root utilizando la contraseña de root
```

---

## 🛠️ OTROS COMANDOS Y UTILIDADES

### which
🔎 **Which**
Muestra la ruta absoluta del ejecutable binario de un comando. Útil para saber qué versión de un programa se está ejecutando.
```bash
which python3
which nano
```

### date
📅 **Date**
Muestra o establece la fecha y hora del sistema.
```bash
date
date -u    # Muestra la hora en formato UTC
```

### uptime
⏱️ **Uptime**
Muestra cuánto tiempo lleva el equipo encendido ininterrumpidamente y la carga media del sistema.
```bash
uptime
```

### nano
📝 **Nano**
Editor de texto en la terminal, sencillo e intuitivo.
```bash
nano archivo.txt
```
*(Guarda con `Ctrl+O` y sal con `Ctrl+X`).*

### gzip
🗜️ **Gzip**
Comprime y descomprime archivos individuales (no empaqueta carpetas). Reemplaza el archivo original por su versión `.gz`.
```bash
gzip archivo.txt           # Comprime y crea archivo.txt.gz
gzip -d archivo.txt.gz     # Descomprime el archivo (-d o gunzip)
```

### tar
📦 **T**ape **Ar**chive
Empaqueta múltiples archivos y directorios en un solo archivo (tarball). Generalmente se combina con `gzip` para comprimir al mismo tiempo (`.tar.gz`).
```bash
tar -cvzf archivo.tar.gz /ruta/carpeta   # Crea (c), Verbose (v), Gzip (z), File (f)
tar -xvzf archivo.tar.gz                 # Extrae (x) el contenido en el directorio actual
tar -xvzf archivo.tar.gz -C /ruta/dest   # Extrae en un directorio específico
tar -ztvf archivo.tar.gz                 # Lista (t) el contenido sin extraerlo
```

### scp
🌐 **S**ecure **C**o**p**y
Copia archivos de forma segura entre un host local y remoto a través del protocolo SSH. Ideal para transferir datos a placas como la Raspberry Pi.
```bash
scp archivo.txt usuario@192.168.1.50:/ruta/destino/        # Sube un archivo al servidor
scp usuario@192.168.1.50:/ruta/remota.txt /ruta/local/     # Descarga un archivo del servidor
scp -r carpeta/ pi@raspberry:/home/pi/                     # Copia un directorio entero recursivamente
```

### rsync
🔄 **Rsync**
Herramienta avanzada para sincronizar archivos de forma rápida y eficiente localmente o a través de red. Solo transfiere los cambios (deltas), ahorrando ancho de banda.
```bash
rsync -av carpeta_local/ carpeta_respaldo/                 # Sincronización local conservando permisos y recursivo (a), con modo verboso (v)
rsync -avz local/ pi@192.168.1.50:/home/pi/remoto/         # Sincroniza hacia un host remoto, comprimiendo (z) la transferencia
```

### systemctl
⚙️ **System Control**
Administra los servicios del sistema y los demonios en sistemas basados en systemd (como Debian, Ubuntu o Raspberry Pi OS).
```bash
sudo systemctl status ssh            # Revisa el estado de un servicio
sudo systemctl start ssh             # Inicia un servicio
sudo systemctl stop ssh              # Detiene un servicio
sudo systemctl restart mi_app        # Reinicia un servicio
sudo systemctl enable mi_app         # Configura el servicio para que arranque automáticamente al encender
```