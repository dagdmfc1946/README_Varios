# Guía práctica de Git, SSH, GitHub/GitLab, Raspberry Pi y VSCode

Referencia técnica para trabajar con **Git**, **GitHub/GitLab**, **SSH**, **Raspberry Pi**, **Linux**, **VSCode** y transferencia de archivos. Incluye desde la configuración inicial hasta el flujo de trabajo diario con repositorios locales y remotos.

> **Nota:** donde aparezcan valores como `usuario`, `IP_RASPBERRY`, `192.168.1.120`, `nombre.service`, `nombre_repositorio`, etc., son valores de ejemplo que deben sustituirse por los correspondientes al entorno real.

---

## Índice

- [1. Conceptos fundamentales](#1-conceptos-fundamentales)
- [2. SSH: llaves y autenticación](#2-ssh-llaves-y-autenticación)
- [3. Configuración de múltiples llaves SSH](#3-configuración-de-múltiples-llaves-ssh)
- [4. Configuración de Git](#4-configuración-de-git)
- [5. Repositorios Git: clonar y configurar](#5-repositorios-git-clonar-y-configurar)
- [6. Flujo de trabajo diario en Git](#6-flujo-de-trabajo-diario-en-git)
- [7. Actualizar el repositorio local](#7-actualizar-el-repositorio-local)
- [8. Estado, historial y remotos](#8-estado-historial-y-remotos)
- [9. Gestión de ramas](#9-gestión-de-ramas)
- [10. SSH desde Windows hacia Raspberry Pi](#10-ssh-desde-windows-hacia-raspberry-pi)
- [11. Navegación básica en Linux](#11-navegación-básica-en-linux)
- [12. Gestión de archivos y directorios](#12-gestión-de-archivos-y-directorios)
- [13. Transferencia de archivos con SCP](#13-transferencia-de-archivos-con-scp)
- [14. Procesos y aplicaciones en Raspberry Pi](#14-procesos-y-aplicaciones-en-raspberry-pi)
- [15. Servicios de Linux y systemd](#15-servicios-de-linux-y-systemd)
- [16. Git en Raspberry Pi](#16-git-en-raspberry-pi)
- [17. Mantenimiento básico de Raspberry Pi](#17-mantenimiento-básico-de-raspberry-pi)
- [18. SSH Agent en Windows PowerShell](#18-ssh-agent-en-windows-powershell)
- [19. VSCode](#19-vscode)
- [20. Configuración SSH `~/.ssh/config`](#20-configuración-ssh-sshconfig)
- [21. Seguridad y `.gitignore`](#21-seguridad-y-gitignore)
- [22. Chuletas rápidas](#22-chuletas-rápidas)
- [23. Recomendaciones para Raspberry Pi nuevas](#23-recomendaciones-para-raspberry-pi-nuevas)

---

# 1. Conceptos fundamentales

Antes de trabajar con Git y SSH conviene diferenciar dos conceptos que cumplen funciones distintas.

### 1.1. Autenticación SSH

La autenticación SSH determina **qué identidad utiliza el equipo para autenticarse frente a un servidor remoto**, por ejemplo GitHub o GitLab.

Se gestiona mediante:

- Llaves SSH.
- El cliente SSH del sistema operativo.
- El archivo `~/.ssh/config`.
- Opcionalmente, `ssh-agent`.

### 1.2. Identidad de Git

La identidad de Git determina **quién aparece como autor de los commits**.

Se configura mediante:

```bash
git config user.name "Tu Nombre"
git config user.email "correo@ejemplo.com"
```

La identidad puede configurarse:

- **Globalmente:** aplica por defecto a todos los repositorios del usuario.
- **Localmente:** aplica únicamente al repositorio actual.

Para evitar cruces de identidad entre proyectos, es posible utilizar configuración local:

```bash
git config --local user.name "Usuario Plataforma"
git config --local user.email "correo@ejemplo.com"
```

> **Importante:** la identidad de Git (`user.name` / `user.email`) y la autenticación SSH son mecanismos diferentes. Una define la autoría del commit y la otra controla la autenticación frente al servidor remoto.

---

# 2. SSH: llaves y autenticación

## 2.1. Verificar las llaves existentes

Desde **PowerShell** o **Git Bash**:

```bash
ls -la ~/.ssh
```

También:

```bash
ls ~/.ssh
```

Esto permite comprobar si ya existen llaves como:

```text
id_ed25519
id_ed25519.pub
```

## 2.2. Generar una llave SSH

Una llave Ed25519 puede generarse mediante:

```bash
ssh-keygen -t ed25519 -C "tu_correo@ejemplo.com"
```

Los parámetros principales son:

| Parámetro | Función |
|---|---|
| `-t ed25519` | Especifica el algoritmo Ed25519. |
| `-C "..."` | Añade un comentario o etiqueta a la llave. |

Al solicitar una frase de paso (*passphrase*), puede establecerse una o dejarse vacía.

## 2.3. Archivos de una llave SSH

Llave privada:

```text
~/.ssh/id_ed25519
```

> **Nunca compartir la llave privada.**

Llave pública:

```text
~/.ssh/id_ed25519.pub
```

> La llave pública es la que se registra en el servicio remoto.

## 2.4. Mostrar la llave pública

```bash
cat ~/.ssh/id_ed25519.pub
```

Copiar el contenido de la llave pública únicamente para registrarlo en el servicio correspondiente.

## 2.5. SSH Agent

Iniciar el agente:

```bash
eval "$(ssh-agent -s)"
```

Cargar la llave:

```bash
ssh-add ~/.ssh/id_ed25519
```

Comprobar las identidades cargadas:

```bash
ssh-add -l
```

## 2.6. Probar la conexión con GitHub

```bash
ssh -T git@github.com
```

Para obtener información detallada durante la autenticación:

```bash
ssh -vT git@github.com
```

Este último comando es útil para diagnosticar problemas como:

```text
Permission denied (publickey)
```

---

# 3. Configuración de múltiples llaves SSH

Cuando se utilizan varias cuentas o plataformas Git, es recomendable utilizar llaves independientes y configurar explícitamente qué llave debe utilizar SSH.

Este esquema puede utilizarse para:

- Diferentes cuentas de GitHub.
- GitHub y GitLab.
- GitHub, GitLab y otras plataformas compatibles con SSH.
- Diferentes identidades para proyectos independientes.

## 3.1. Crear una llave específica para GitHub

```bash
ssh-keygen -t ed25519 -C "correo_github@dominio.com" -f ~/.ssh/id_github
```

## 3.2. Crear una llave específica para GitLab

```bash
ssh-keygen -t ed25519 -C "correo_gitlab@dominio.com" -f ~/.ssh/id_gitlab
```

El parámetro `-f` permite definir explícitamente el nombre y ubicación del archivo de salida.

## 3.3. Mostrar las llaves públicas

GitHub:

```bash
cat ~/.ssh/id_github.pub
```

GitLab:

```bash
cat ~/.ssh/id_gitlab.pub
```

Las llaves públicas deben registrarse en las respectivas plataformas.

## 3.4. Configurar `~/.ssh/config`

Crear o editar:

```bash
notepad ~/.ssh/config
```

Ejemplo:

```text
# Configuración para GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github
    IdentitiesOnly yes

# Configuración para GitLab
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_gitlab
    IdentitiesOnly yes
```

### Directivas principales

| Directiva | Función |
|---|---|
| `Host` | Define el host utilizado por SSH. |
| `HostName` | Define el dominio o dirección real del servidor. |
| `User` | Usuario utilizado para la conexión SSH. |
| `IdentityFile` | Define la llave privada que debe utilizarse. |
| `IdentitiesOnly yes` | Hace que SSH utilice únicamente las identidades especificadas para ese host. |

## 3.5. Verificar las conexiones

GitHub:

```bash
ssh -T git@github.com
```

GitLab:

```bash
ssh -T git@gitlab.com
```

---

# 4. Configuración de Git

## 4.1. Verificar instalación

```bash
git --version
```

## 4.2. Configurar identidad global

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "correo@ejemplo.com"
```

## 4.3. Ver configuración

```bash
git config --list
```

Para consultar únicamente la configuración local del repositorio actual:

```bash
git config --local -l
```

---

# 5. Repositorios Git: clonar y configurar

## 5.1. Clonar un repositorio de GitHub

Utilizando SSH:

```bash
git clone git@github.com:usuario/repositorio.git
```

La estructura de una URL SSH de GitHub es:

```text
git@github.com:usuario/repositorio.git
```

En lugar de:

```text
https://github.com/usuario/repositorio.git
```

## 5.2. Clonar un repositorio de GitLab

```bash
git clone git@gitlab.com:usuario/repositorio.git
```

## 5.3. Configurar identidad local después de clonar

Entrar al repositorio:

```bash
cd repositorio
```

Configurar la identidad:

```bash
git config user.name "Usuario GitHub"
git config user.email "correo_github@dominio.com"
```

## 5.4. Crear un repositorio local nuevo

Inicializar Git:

```bash
git init
```

Configurar identidad:

```bash
git config user.name "Usuario Plataforma"
git config user.email "correo_plataforma@dominio.com"
```

Agregar el repositorio remoto.

GitHub:

```bash
git remote add origin git@github.com:usuario/nombre-repositorio.git
```

GitLab:

```bash
git remote add origin git@gitlab.com:usuario/nombre-repositorio.git
```

Realizar el primer commit:

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

---

# 6. Flujo de trabajo diario en Git

El flujo básico para enviar cambios al repositorio remoto es:

```text
Working Directory
       ↓
    git add
       ↓
   Staging Area
       ↓
   git commit
       ↓
 Local Repository
       ↓
    git push
       ↓
Remote Repository
```

## 6.1. Paso 1 — Preparar archivos (`git add`)

Agregar todos los archivos modificados y nuevos:

```bash
git add .
```

Agregar un archivo específico:

```bash
git add nombre_del_archivo.ext
```

Agregar una carpeta:

```bash
git add ruta/a/carpeta/
```

## 6.2. Paso 2 — Crear un commit

```bash
git commit -m "Descripción clara y corta de los cambios realizados"
```

Ejemplo:

```bash
git commit -m "feat: agrega interfaz de autenticación para usuarios"
```

## 6.3. Paso 3 — Enviar cambios (`git push`)

```bash
git push
```

Para publicar una rama local por primera vez:

```bash
git push -u origin nombre_de_la_rama
```

La opción `-u` establece la rama remota como *upstream*, permitiendo utilizar posteriormente:

```bash
git push
```

---

# 7. Actualizar el repositorio local

Cuando existen cambios en el repositorio remoto, deben descargarse antes de continuar trabajando.

## 7.1. `git pull`

Descarga e integra los cambios remotos en la rama actual:

```bash
git pull
```

## 7.2. `git fetch`

Descarga información y cambios del repositorio remoto sin integrarlos directamente en la rama actual:

```bash
git fetch
```

> **Buena práctica:** antes de comenzar a trabajar o antes de realizar un `push`, conviene comprobar si existen cambios remotos que deban integrarse.

---

# 8. Estado, historial y remotos

## 8.1. Ver estado del repositorio

```bash
git status
```

Permite identificar archivos:

- Modificados.
- Preparados para commit.
- No rastreados.

## 8.2. Ver historial resumido

```bash
git log --oneline
```

## 8.3. Ver repositorios remotos

```bash
git remote -v
```

---

# 9. Gestión de ramas

Las ramas permiten desarrollar funcionalidades o realizar cambios de manera independiente.

## 9.1. Listar ramas

```bash
git branch
```

La rama activa aparece marcada con `*`.

## 9.2. Crear una rama y cambiar a ella

Sintaxis tradicional:

```bash
git checkout -b nombre_de_nueva_rama
```

Sintaxis moderna:

```bash
git switch -c nombre_de_nueva_rama
```

## 9.3. Cambiar entre ramas

Sintaxis tradicional:

```bash
git checkout nombre_de_rama
```

Sintaxis moderna:

```bash
git switch nombre_de_rama
```

## 9.4. Fusionar una rama

Estando ubicado en la rama que recibirá los cambios:

```bash
git merge nombre_de_otra_rama
```

---

# 10. SSH desde Windows hacia Raspberry Pi

## 10.1. Conectarse mediante dirección IP

```bash
ssh usuario@192.168.1.120
```

## 10.2. Conectarse mediante hostname `.local`

Si mDNS está funcionando:

```bash
ssh usuario@telsy-rpi.local
```

## 10.3. Obtener la IP de la Raspberry

Desde la Raspberry:

```bash
hostname -I
```

## 10.4. Consultar información del hostname

```bash
hostnamectl
```

## 10.5. Cambiar el hostname

```bash
sudo hostnamectl set-hostname telsy-rpi
```

Después puede intentarse:

```bash
ssh usuario@telsy-rpi.local
```

---

# 11. Navegación básica en Linux

| Comando | Descripción | Uso |
|---|---|---|
| `pwd` | Muestra el directorio actual. | Saber dónde se encuentra la terminal. |
| `ls` | Lista archivos y carpetas. | Ver contenido del directorio. |
| `ls -la` | Lista archivos, incluidos ocultos, con información detallada. | Consultar permisos, propietario, tamaño, etc. |
| `cd carpeta` | Cambia de directorio. | Entrar en una carpeta. |
| `cd ..` | Sube un nivel. | Ir al directorio padre. |
| `cd ~` | Va al directorio HOME. | Regresar rápidamente al directorio personal. |

Ejemplo:

```bash
pwd
ls -la
cd proyecto
ls
cd ..
cd ~
```

## 11.1. Acceder a una TTY

Si la Raspberry inicia automáticamente una aplicación gráfica, puede utilizarse una consola virtual mediante:

```text
CTRL + ALT + F2
```

o:

```text
CTRL + ALT + F3
```

Para regresar al entorno gráfico, dependiendo de la configuración:

```text
CTRL + ALT + F1
```

o:

```text
CTRL + ALT + F7
```

> Estos son **atajos de teclado**, no comandos de terminal.

---

# 12. Gestión de archivos y directorios

| Comando | Descripción | Uso |
|---|---|---|
| `mkdir nombre` | Crea un directorio. | Crear una carpeta. |
| `cp origen destino` | Copia archivos. | Crear una copia en otra ubicación. |
| `mv origen destino` | Mueve o renombra archivos. | Cambiar ubicación o nombre. |
| `rm archivo.txt` | Elimina un archivo. | Borrar un archivo. |
| `rm -r carpeta` | Elimina recursivamente una carpeta. | Borrar una carpeta y su contenido. |
| `nano archivo.txt` | Abre un archivo en Nano. | Editar archivos desde la terminal. |

## 12.1. Atajos de Nano

| Atajo | Función |
|---|---|
| `CTRL + O` | Guardar. |
| `CTRL + X` | Salir. |

---

# 13. Transferencia de archivos con SCP

`scp` permite transferir archivos mediante SSH entre el PC y la Raspberry.

> Se requiere que el servidor SSH de la Raspberry esté funcionando.

## 13.1. PC → Raspberry: archivo individual

```bash
scp archivo.txt usuario@192.168.1.120:/home/usuario/
```

Ejemplo:

```bash
scp firmware.bin diegod@192.168.1.120:/home/diegod/
```

| Parte | Función |
|---|---|
| `scp` | Copia archivos mediante SSH. |
| `archivo.txt` | Archivo de origen. |
| `usuario@192.168.1.120` | Usuario y dirección de la Raspberry. |
| `/home/usuario/` | Directorio de destino. |

## 13.2. Raspberry → PC: archivo individual

```bash
scp usuario@192.168.1.120:/home/usuario/archivo.txt .
```

El punto `.` representa la carpeta actual del PC.

## 13.3. PC → Raspberry: carpeta completa

```bash
scp -r carpeta usuario@192.168.1.120:/home/usuario/
```

La opción `-r` permite copiar directorios de forma recursiva.

## 13.4. Raspberry → PC: carpeta completa

```bash
scp -r usuario@192.168.1.120:/home/usuario/carpeta .
```

---

# 14. Procesos y aplicaciones en Raspberry Pi

## 14.1. Listar procesos

```bash
ps aux
```

## 14.2. Buscar procesos de Python

```bash
ps aux | grep python
```

## 14.3. Buscar procesos de Node.js

```bash
ps aux | grep node
```

Ejemplo de resultado:

```text
diegod    1234 ... python3 app.py
```

Terminar el proceso:

```bash
kill 1234
```

Si no responde:

```bash
kill -9 1234
```

> `kill -9` fuerza la terminación del proceso. Debe utilizarse cuando una terminación normal no sea suficiente.

---

# 15. Servicios de Linux y systemd

## 15.1. Listar servicios instalados

```bash
systemctl list-unit-files --type=service
```

## 15.2. Listar servicios cargados

```bash
systemctl list-units --type=service
```

## 15.3. Consultar el estado de un servicio

```bash
systemctl status nombre.service
```

Ejemplo:

```bash
systemctl status mi_app.service
```

## 15.4. Deshabilitar el inicio automático

```bash
sudo systemctl disable nombre.service
```

---

# 16. Git en Raspberry Pi

## 16.1. Actualizar índices de paquetes

```bash
sudo apt update
```

## 16.2. Instalar Git

```bash
sudo apt install git -y
```

## 16.3. Verificar instalación

```bash
git --version
```

## 16.4. Clonar un repositorio

```bash
git clone git@github.com:usuario/repositorio.git
```

## 16.5. Actualizar un repositorio existente

```bash
git pull
```

---

# 17. Mantenimiento básico de Raspberry Pi

## 17.1. Actualizar el sistema

Actualizar índices:

```bash
sudo apt update
```

Actualizar paquetes:

```bash
sudo apt upgrade -y
```

También pueden ejecutarse ambas operaciones:

```bash
sudo apt update && sudo apt upgrade -y
```

## 17.2. Limpiar caché de paquetes

```bash
sudo apt clean
```

## 17.3. Eliminar dependencias que ya no son necesarias

```bash
sudo apt autoremove -y
```

## 17.4. Limpiar caché del usuario

```bash
rm -rf ~/.cache/*
```

> ⚠️ Este comando es más agresivo que `apt clean` y `apt autoremove`. Debe utilizarse únicamente cuando se quiera eliminar la caché del usuario.

## 17.5. Consultar almacenamiento

```bash
df -h
```

## 17.6. Consultar memoria RAM

```bash
free -h
```

## 17.7. Monitorizar procesos

```bash
top
```

Alternativa:

```bash
htop
```

Instalar `htop`:

```bash
sudo apt install htop -y
```

## 17.8. Consultar temperatura

```bash
vcgencmd measure_temp
```

## 17.9. Reiniciar

```bash
sudo reboot
```

## 17.10. Apagar

```bash
sudo shutdown now
```

## 17.11. Cambiar contraseña

```bash
passwd
```

## 17.12. Crear un usuario

```bash
sudo adduser diegod
```

Agregarlo al grupo `sudo`:

```bash
sudo usermod -aG sudo diegod
```

---

# 18. SSH Agent en Windows PowerShell

Consultar el servicio:

```powershell
Get-Service ssh-agent
```

Iniciar el servicio:

```powershell
Start-Service ssh-agent
```

Configurar inicio automático:

```powershell
Set-Service -StartupType Automatic ssh-agent
```

Cargar una llave:

```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

---

# 19. VSCode

VSCode permite trabajar con Git y conectarse remotamente mediante SSH.

## 19.1. Command Palette

Abrir la Command Palette:

```text
CTRL + SHIFT + P
```

## 19.2. Clonar un repositorio

Seleccionar:

```text
Git: Clone
```

Utilizar la URL SSH correspondiente, por ejemplo:

```text
git@github.com:usuario/repositorio.git
```

## 19.3. Conectarse a una Raspberry

Seleccionar:

```text
Remote-SSH: Connect to Host
```

Utilizar, por ejemplo:

```text
usuario@192.168.1.120
```

También puede utilizarse un host definido en `~/.ssh/config`.

---

# 20. Configuración SSH `~/.ssh/config`

El archivo `~/.ssh/config` permite definir configuraciones reutilizables para diferentes hosts.

Ejemplo:

```text
# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_github
    IdentitiesOnly yes

# GitLab
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile ~/.ssh/id_gitlab
    IdentitiesOnly yes

# Raspberry Pi
Host raspberry
    HostName 192.168.1.120
    User diegod
    IdentityFile ~/.ssh/id_ed25519
```

Después de definir el host de la Raspberry, puede utilizarse:

```bash
ssh raspberry
```

en lugar de:

```bash
ssh diegod@192.168.1.120
```

---

# 21. Seguridad y `.gitignore`

El archivo `.gitignore` permite indicar a Git qué archivos o patrones no deben incorporarse al repositorio.

Ejemplo:

```gitignore
.env
*.pem
*.key
*.crt
secrets.json
```

Puede utilizarse para evitar subir accidentalmente:

- Credenciales.
- API keys.
- Certificados.
- Llaves privadas.
- Archivos `.env`.
- Archivos de configuración que contengan secretos.

> **Importante:** un archivo incluido en `.gitignore` no sustituye una estrategia de gestión de secretos. Las credenciales nunca deben incorporarse al historial de Git.

---

# 22. Chuletas rápidas

## 22.1. Flujo Git diario

```bash
# 1. Traer cambios recientes
git pull

# 2. Realizar modificaciones en el proyecto

# 3. Revisar cambios
git status

# 4. Preparar cambios
git add .

# 5. Crear commit
git commit -m "Descripción breve del cambio"

# 6. Subir cambios
git push
```

## 22.2. Git básico

```bash
git --version
git status
git add .
git commit -m "Descripción del cambio"
git pull
git push
git fetch
git log --oneline
git remote -v
git branch
git checkout change_branch_name
git checkout -b create_branch_name
git switch -c nombre_de_rama
git switch nombre_de_rama
git merge nombre_de_otra_rama
```

## 22.3. SSH

```bash
# Ver llaves
ls -la ~/.ssh

# Crear llave
ssh-keygen -t ed25519 -C "correo@ejemplo.com"

# Iniciar agente
eval "$(ssh-agent -s)"

# Cargar llave
ssh-add ~/.ssh/id_ed25519

# Ver llaves cargadas
ssh-add -l

# Mostrar llave pública
cat ~/.ssh/id_ed25519.pub

# Probar GitHub
ssh -T git@github.com

# Diagnosticar SSH
ssh -vT git@github.com
```

## 22.4. Raspberry / Linux

```bash
# Conectarse
ssh usuario@IP_RASPBERRY

# Saber dónde estoy
pwd

# Ver archivos
ls -lah

# Entrar a una carpeta
cd carpeta

# Subir un nivel
cd ..

# Volver a HOME
cd ~

# Ver IP
hostname -I

# Ver espacio
df -h

# Ver RAM
free -h

# Ver procesos
top

# Ver temperatura
vcgencmd measure_temp

# Ver servicios
systemctl list-units --type=service

# Ver estado de un servicio
systemctl status nombre.service

# Reiniciar
sudo reboot

# Apagar
sudo shutdown now
```

## 22.5. Transferencia de archivos

```bash
# PC → Raspberry
scp archivo.txt usuario@IP_RASPBERRY:/home/usuario/

# Raspberry → PC
scp usuario@IP_RASPBERRY:/home/usuario/archivo.txt .

# PC → Raspberry, carpeta
scp -r carpeta usuario@IP_RASPBERRY:/home/usuario/

# Raspberry → PC, carpeta
scp -r usuario@IP_RASPBERRY:/home/usuario/carpeta .
```

---

# 23. Recomendaciones para Raspberry Pi nuevas

Cuando se reemplaza una Raspberry Pi por otra unidad:

1. Generar una llave SSH independiente en la nueva Raspberry.
2. Registrar la nueva llave pública en GitHub o GitLab.
3. Probar la autenticación:

   ```bash
   ssh -T git@github.com
   ```

4. Confirmar que Git funciona:

   ```bash
   git clone git@github.com:usuario/repositorio.git
   ```

5. Configurar la identidad Git correspondiente al proyecto:

   ```bash
   git config user.name "Usuario Plataforma"
   git config user.email "correo@ejemplo.com"
   ```

6. Cuando corresponda, retirar de la plataforma la llave asociada a la Raspberry anterior.

> **Recomendación:** cada dispositivo debería utilizar su propia identidad SSH. No es recomendable copiar la llave privada de una Raspberry anterior a una nueva unidad.
