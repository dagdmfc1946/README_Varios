# 🍇 Raspberry Pi 🐍  

---

## 📑 Tabla de Contenidos
- [¿Qué es una Raspberry Pi?](#-qué-es-una-raspberry-pi)  
- [Cómo empezar](#-cómo-empezar)  
- [Comandos básicos y útiles](#-comandos-básicos-y-útiles)  
  - [Gestión de paquetes](#gestión-de-paquetes)  
  - [Firmware y actualizaciones](#firmware-y-actualizaciones)  
  - [Salida de vídeo](#salida-de-vídeo)  
  - [Python y entornos virtuales](#python-y-entornos-virtuales)  
  - [Control de versiones (Git)](#control-de-versiones-git)  
- [Documentación oficial](#-documentación-oficial)  

---

## 🍇 ¿Qué es una Raspberry Pi?
- Es una computadora de bajo costo y tamaño reducido, utilizada en proyectos de **automatización, control, programación, IoT**, entre otros.  
- Existen diferentes modelos: Raspberry Pi 3B, Raspberry Pi 4B, Raspberry Pi 5, Raspberry Pi CM4, Raspberry Pi Pico, etc.  
- Según el modelo, se dispondrá de distintas interfaces: **HDMI/micro HDMI, pines GPIO, puertos USB, Ethernet, conector de cámara, etc.**  

🔗 Enlaces útiles:  
- [Productos (Hardware)](https://www.raspberrypi.com/products/)  
- [Software](https://www.raspberrypi.com/software/)  

---

## 💻 ¿Cómo empezar?
- 📘 [Primeros pasos con Raspberry Pi](https://www.raspberrypi.com/documentation/computers/getting-started.html)  
- 🖥️ [Sistema Operativo (Raspberry Pi OS)](https://www.raspberrypi.com/documentation/computers/os.html#introduction)  
- ⚙️ [Configuración con raspi-config](https://www.raspberrypi.com/documentation/computers/configuration.html#raspi-config)  

---

## 💲 Comandos básicos y útiles  

### 🔹 Atajos rápidos
- Abrir una nueva terminal: **`Ctrl + Alt + T`**  
- Abrir Geany (editor de texto):  
  ```bash
  geany &
  ```

### 🔷 Gestión de paquetes
```bash
# Actualizar la lista de paquetes
sudo apt update

# Actualizar todos los paquetes a la última versión
sudo apt full-upgrade
    
# Buscar un paquete
apt-cache search <keyword>
    
# Mostrar información de un paquete
apt-cache show <package-name>
    
# Instalar un paquete
sudo apt install <package-name>
    
# Desinstalar un paquete
sudo apt remove <package-name>
```
  
### 🔷 Firmware y actualizaciones
 
```bash
# Actualizar kernel, módulos y firmware (⚠️ hacer backup antes)
sudo rpi-update
sudo reboot
    
# Si algo falla → volver al firmware estable
sudo apt update
sudo apt install --reinstall raspi-firmware
sudo reboot
```
### 🔷 Salida de vídeo

```bash
# Reproducir un video en un dispositivo DRM específico
cvlc --play-and-exit --drm-vout-display <drm-device> video.mp4
    
# Listar dispositivos DRM disponibles
kmsprint | grep Connector
```
 
### 🐍 Python y entornos virtuales

```bash
# Crear un entorno virtual
python -m venv <venv_name>

# Activar el entorno
source <venv_name>/bin/activate

# Verificación
pip list

# Salir del entorno
deactivate
```

### 🌱 Control de versiones (Git)

```bash
sudo apt-get install git

# Configuración inicial:
git config --global user.name "TuNombre"
git config --global user.email "tuemail@example.com"
```

---

# 📚 Documentación oficial: 
## [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)

Este archivo resume la documentación oficial de Raspberry Pi con enlaces directos a cada sección principal.  

---

## 🚀 Introducción
- [Primeros pasos](https://www.raspberrypi.com/documentation/computers/getting-started.html)  
- [Raspberry Pi OS](https://www.raspberrypi.com/documentation/computers/os.html#introduction)  
- [Instalación de Raspberry Pi Imager](https://www.raspberrypi.com/software/)  

---

## ⚙️ Configuración
- [Configuración inicial (raspi-config)](https://www.raspberrypi.com/documentation/computers/configuration.html#raspi-config)  
- [Red y conectividad](https://www.raspberrypi.com/documentation/computers/configuration.html#networking)  
- [Localización (teclado, idioma, zona horaria)](https://www.raspberrypi.com/documentation/computers/configuration.html#localisation)  

---

## 🔌 Hardware
- [GPIO y periféricos](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#gpio)  
- [Cámara](https://www.raspberrypi.com/documentation/computers/camera.html)  
- [Pantallas y salida de vídeo](https://www.raspberrypi.com/documentation/computers/display.html)  
- [Almacenamiento y USB](https://www.raspberrypi.com/documentation/computers/storage.html)  

---

## 🐍 Software
- [Python en Raspberry Pi](https://www.raspberrypi.com/documentation/computers/python.html)  
- [Otros lenguajes (C, Scratch, etc.)](https://www.raspberrypi.com/documentation/computers/programming.html)  

---

## 🔧 Administración y mantenimiento
- [Gestión de paquetes (apt)](https://www.raspberrypi.com/documentation/computers/os.html#software-installation)  
- [Actualización de firmware](https://www.raspberrypi.com/documentation/computers/os.html#firmware)  
- [Seguridad](https://www.raspberrypi.com/documentation/computers/security.html)  

---

## 🌐 Conectividad avanzada
- [SSH y acceso remoto](https://www.raspberrypi.com/documentation/computers/remote-access.html#ssh)  
- [VNC](https://www.raspberrypi.com/documentation/computers/remote-access.html#vnc)  
- [Servidor web y nube](https://www.raspberrypi.com/documentation/computers/remote-access.html#web-server)  

---

## 📦 Proyectos e IoT
- [Proyectos oficiales](https://projects.raspberrypi.org/)  
- [IoT y automatización](https://www.raspberrypi.com/documentation/computers/iot.html)  

---

## 📌 Referencias generales
- [Raspberry Pi Documentation (Home)](https://www.raspberrypi.com/documentation/)  
- [Foro oficial](https://forums.raspberrypi.com/)  
- [GitHub Raspberry Pi](https://github.com/raspberrypi)  

---

> [!NOTE]
> **Fecha de creación**: 25 Agosto 2025  
> **README**: Documentación y comandos principales para usar una Raspberry Pi.
> **Estado**: ✅ REVISADO.
>
> **Hecho por:** @dagdmfc
> 
