# 👨🏽‍💻🍇 ¿Cómo instalar y ejecutar el proyecto TELSY HOGAR? 🐍📺

1. **Revisar el archivo `README.md`**  
   El proyecto incluye un `README.md` en la ruta `/telsy-monitor` y también en el repositorio de [GitLab: telsyHOGAR_Corea](https://gitlab.com/business-lab/telsyhogar_corea/-/blob/by_dagd/README.md?ref_type=heads).  
   Allí se encuentran la descripción general del sistema, su funcionamiento, primeros pasos y las versiones mínimas de dependencias.  
   > [!Note]
   > El archivo `requirements.txt` detalla las librerías y versiones exactas usadas.  

2. **Crear un entorno virtual (recomendado)**  
   Esto permite aislar las dependencias y evitar conflictos entre librerías.  
   ```bash
   # Crear entorno virtual
   python3 -m venv <venv_name>        # Crea el entorno virtual
   source <venv_name>/bin/activate    # Activar entorno virtual
   
   # Para salir del entorno virtual
   deactivate
   ```

3. **Instalar dependencias del proyecto**  
   ```bash
   # Opción 1 (script recomendado)
   ./install_dependencies_telsyAPP.sh
   # Este mismo script se utiliza para generar el ejecutable con PyInstaller.
   
   # Opción 2 (manual)
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

   > [!Note]
   > Para comprobar que todo funciona:  
   > ```bash
   > python manage.py runserver
   > ```

4. **Clonar el repositorio**  
   En la Raspberry, posicionarse o crear la carpeta `/home/pi/telsy-monitor` y clonar por SSH (hay dos formas):  
   ```bash
   git clone -b dev git@gitlab.com:business-lab/telsyhogar_corea.git
   ```
   
   ```bash
   git clone git@gitlab.com:business-lab/telsyhogar_corea.git
   ```
   > [!Note]
   > La rama activa de desarrollo es **by_dagd**, donde se encuentran los cambios más recientes.  

5. **Configurar inicio automático (autostart)**  
   Editar el archivo de autostart:  
   ```bash
   sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
   ```

   ![autostart_file](/Informes%20Terminados/Imagenes%20Informes/05.%20Ejecutable%20(PyInstaller)/autostart_new.png)

   - Para desarrollo: descomentar las siguientes líneas:  
     ```
     @lxpanel --profile LXDE-pi
     @pcmanfm --desktop --profile LXDE-pi
     ```  
   - Para producción: no modificar el archivo (queda con la configuración predeterminada).  

6. **Ejecutar el proyecto**  
   Una vez en la rama **by_dagd**, el proyecto puede iniciarse según lo definido en `autostart` o manualmente con:  
   ```bash
   ./startserver.sh
   ./startweb.sh
   ./run_driver.sh
   ```
   
7. 📂 **Contenido del repositorio (`by_dagd`)**  

   La rama **by_dagd** contiene los archivos y directorios principales necesarios para la ejecución y mantenimiento del proyecto:  

   - **Ejecutables/** → Binarios y archivos listos para ejecución.  
   - **Informes Terminados/** → Documentación y reportes técnicos del proyecto.  
   - **Manuales/** →  Datasheets y/o manuales usados durante el desarrollo.
   - **driver_files/** → Archivos asociados al driver (lenguaje C) desarrollado por el equipo de Corea del Sur (vspm-pm6750). 
   - **telsy/** → Código fuente principal del sistema TELSY HOGAR.  
   - **vspm-pm6750_DRIVER/** → Archivos con los que se construye el driver y el ejecutable del driver (vspm-pm6750).

   Archivos clave en la raíz:  
   - `LICENSE.md` → Información sobre la licencia del proyecto.  
   - `README.md` → Archivo principal de introducción y guía rápida.  
   - `install_dependencies_telsyAPP.sh` → Script para instalar dependencias y generar ejecutables.  
   - `requirements.txt` → Dependencias exactas del proyecto.  
   - `run_driver.sh`, `startserver.sh`, `startweb.sh` → Scripts de inicio y ejecución del sistema (sin usar los ejecutables generados con PyInstaller).  