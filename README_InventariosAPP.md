# InventariosAPP

Aplicacion local para Windows destinada a administrar inventario personal o profesional de componentes electronicos. Permitira registrar, consultar y mantener componentes, sus especificaciones tecnicas, proveedores, precios, stock, ubicaciones fisicas y datasheets.

## Estado actual

Las fases F1 a F10 estan completadas. La aplicacion dispone de administracion Django, buscador, dashboard, movimientos de stock, gestion de datasheets, backup local y pruebas automatizadas.

## Alcance del MVP

- Gestion administrativa de componentes, categorias, proveedores y ofertas.
- Especificaciones tecnicas configurables por componente.
- Stock, movimientos de inventario y alertas de bajo stock.
- Ubicacion fisica y adjuntos PDF de datasheets.
- Busqueda con filtros, paginacion y dashboard de indicadores.
- Ejecucion local en `127.0.0.1`; sin servicios externos obligatorios.

## Politica tecnologica

Todo el desarrollo utilizara exclusivamente herramientas, librerias y servicios gratuitos u open source. No se incorporaran dependencias de pago, licencias propietarias obligatorias, APIs comerciales ni infraestructura cloud necesaria para operar el MVP.

La aplicacion se construira como un monolito modular con Django y SQLite. La interfaz principal de mantenimiento sera Django Admin personalizado; la busqueda y el dashboard tendran vistas propias.

## Estructura inicial

```text
InventariosAPP/
|-- apps/                 # Apps Django por dominio (se crean en F2)
|-- config/               # Configuracion raiz de Django (F2)
|-- docs/                 # Documentacion tecnica y de proceso
|-- media/datasheets/     # PDFs cargados en ejecucion; no se versionan
|-- static/css/           # Recursos CSS propios o vendorizados
|-- static/js/            # Recursos JavaScript propios o vendorizados
|-- templates/            # Plantillas globales Django
|-- tests/                # Pruebas automatizadas
|-- backups/              # Copias locales; no se versionan
|-- .env.example          # Plantilla de variables de entorno (F1)
|-- .gitignore
`-- README.md
```

Consulte los documentos de [arquitectura](docs/ARQUITECTURA.md), [plan de desarrollo](docs/PLAN_DESARROLLO.md), [dependencias](docs/DEPENDENCIAS.md), [flujo Git](docs/FLUJO_GIT.md) y el registro de fases en [info_FASES.md](info_FASES.md).

## Flujo de trabajo

El desarrollo se realiza en `dev`. Cada fase se documenta, prueba y confirma mediante un commit antes de integrarse de forma controlada en `main`. No se almacenan secretos, entornos virtuales, bases de datos locales, respaldos ni archivos cargados por usuarios.

## Entorno local

Se requiere Python 3.12. Desde PowerShell, situado en la carpeta del proyecto:

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item .env.example .env
.\.venv\Scripts\python.exe manage.py migrate
.\.venv\Scripts\python.exe manage.py createsuperuser
```

El archivo `.env` es local y no se versiona. Cambie `SECRET_KEY` por un valor privado antes de usar la aplicacion fuera de pruebas locales.

## Arranque

```powershell
.\.venv\Scripts\python.exe manage.py runserver 127.0.0.1:8000
```

Abra estas direcciones en el navegador:

- Administracion: http://127.0.0.1:8000/admin/
- Buscador: http://127.0.0.1:8000/search/
- Dashboard: http://127.0.0.1:8000/dashboard/

## Primera prueba: agregar un componente

1. Entre en `/admin/` con el superusuario creado.
2. En **Categories**, cree una categoria, por ejemplo `Sensores`.
3. En **Components**, seleccione **Add Component**.
4. Complete `Category`, `Reference`, `Name` y, si aplica, `Value` y `Part number`.
5. Guarde el componente. Desde su pantalla de cambio puede añadir especificaciones en la seccion **Specifications**.
6. Cree el registro de stock del componente desde **Stocks** y defina cantidad actual, minima y maxima.
7. Compruebe el componente en `/search/` y los indicadores en `/dashboard/`.

## Pruebas y backup

```powershell
.\.venv\Scripts\python.exe manage.py test
.\.venv\Scripts\python.exe manage.py check
```

El backup local se puede ejecutar desde una consola Python con `backup_project` y restaurar con `restore_backup`, funciones disponibles en [backups/backup_local.py](backups/backup_local.py). Las copias deben conservarse fuera de Git.
