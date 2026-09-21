# WebApp - Alertas Sanitarias

## 1. Contexto del proyecto

**WebApp - Alertas Sanitarias** es una aplicación web orientada a automatizar la consulta, revisión, consolidación y trazabilidad de **alertas sanitarias relacionadas con dispositivos médicos**, utilizando como fuente principal información publicada por organismos sanitarios oficiales.

El proyecto surge de una necesidad operativa del área de Dirección Técnica: reducir el trabajo manual asociado a la revisión periódica de alertas sanitarias y facilitar la identificación de aquellas que puedan tener relación con los dispositivos médicos comercializados o gestionados por la organización.

La aplicación debe diseñarse como un sistema **modular, mantenible y escalable**, de forma que sea posible incorporar nuevas fuentes sanitarias sin tener que modificar significativamente el núcleo de la aplicación.

---

## 2. Objetivo general

Desarrollar una WebApp que permita:

1. Consultar automáticamente fuentes oficiales de alertas sanitarias.
2. Extraer y normalizar la información publicada.
3. Comparar las alertas encontradas contra una base de productos/dispositivos médicos configurada por el usuario.
4. Identificar posibles coincidencias o alertas relevantes.
5. Registrar los resultados y conservar trazabilidad histórica.
6. Permitir la revisión manual de los resultados.
7. Generar reportes/exportaciones, inicialmente en **XLSX y PDF**.
8. Mantener una arquitectura preparada para incorporar nuevas entidades sanitarias y nuevas fuentes de información.
9. Permitir posteriormente una estrategia de despliegue apropiada para Windows y/o producción, sin asumir todavía una herramienta específica de empaquetamiento.

---

## 3. Alcance inicial

### 3.1 Fuente principal

La primera fuente objetivo es **INVIMA**, especialmente la información oficial relacionada con:

- Alertas sanitarias.
- Alertas de dispositivos médicos.
- Retiros, riesgos o comunicaciones sanitarias que puedan resultar relevantes para los productos registrados.

La aplicación debe consumir la información publicada por la fuente oficial y evitar depender de fuentes secundarias cuando exista información oficial disponible.

### 3.2 Fuentes futuras

La arquitectura debe permitir incorporar posteriormente otras agencias sanitarias nacionales o internacionales.

La implementación de cada fuente debe estar desacoplada del resto de la aplicación.

Conceptualmente:

```text
Fuente sanitaria
       │
       ▼
Extractor / Scraper
       │
       ▼
Normalizador
       │
       ▼
Modelo común de alerta
       │
       ├──► Comparación con productos
       │
       ├──► Historial / trazabilidad
       │
       └──► Interfaz / reportes
```

---

## 4. Base de productos

Actualmente existe una base inicial en Excel:

```text
BD_ListaRegSanitariosVigentes_QM.xlsx
```

La base contiene información de productos y registros sanitarios que será utilizada como referencia para determinar si una alerta sanitaria puede estar relacionada con productos gestionados por la organización.

La primera versión disponible contiene aproximadamente **28 productos**.

La estructura definitiva del modelo de datos no debe quedar limitada al formato actual del Excel. El Excel debe considerarse una fuente de carga/importación o una representación de los datos, mientras que la aplicación debe trabajar internamente con un modelo estructurado.

---

## 5. Documento de referencia existente

Existe también un archivo:

```text
Revision_Alertas_Sanitarias.md
```

Este archivo contiene información y enlaces relacionados con la revisión de alertas sanitarias y debe utilizarse como referencia durante el desarrollo cuando corresponda.

No debe asumirse que su contenido representa necesariamente la arquitectura definitiva de la aplicación.

---

## 6. Funcionalidades previstas

### 6.1 Gestión de fuentes

La aplicación debe permitir definir y administrar fuentes sanitarias.

Cada fuente debería poder tener, según corresponda:

- Nombre.
- Entidad.
- URL.
- Tipo de fuente.
- Método de consulta.
- Estado.
- Configuración específica.
- Fecha de última consulta.
- Resultado de la última ejecución.

La arquitectura debe permitir agregar nuevas fuentes mediante módulos independientes.

---

### 6.2 Extracción de información

La extracción puede requerir navegación automatizada en sitios web oficiales.

La herramienta principal prevista es:

**Playwright para Python.**

La ejecución debe realizarse utilizando **Google Chrome o Microsoft Edge**, según disponibilidad/configuración del entorno.

**No se debe diseñar el proyecto alrededor de Chromium como navegador objetivo.**

El scraper debe manejar, cuando sea necesario:

- Navegación.
- Formularios.
- Paginación.
- Contenido dinámico.
- Descarga de información.
- Esperas explícitas.
- Errores de navegación.
- Cambios en la estructura del sitio.
- Registro de errores.

Cuando una fuente pueda consultarse mediante HTTP/API de manera confiable y oficialmente soportada, no debe utilizarse automatización de navegador innecesariamente.

---

### 6.3 Normalización

Las diferentes fuentes pueden presentar estructuras y nombres de campos diferentes.

La aplicación debe transformar los datos extraídos a un **modelo común de alerta sanitaria**.

Como mínimo, el modelo debe contemplar campos conceptuales como:

- Identificador de alerta.
- Fecha de publicación.
- Fecha de actualización, si existe.
- Entidad/fuente.
- Tipo de alerta.
- Título.
- Producto.
- Marca.
- Fabricante.
- Importador/distribuidor, cuando aplique.
- Referencia/modelo.
- Lote/serie, cuando aplique.
- Registro sanitario, cuando aplique.
- Descripción.
- Riesgo.
- Medidas recomendadas.
- URL de la fuente oficial.
- Documento/archivo asociado, cuando exista.
- Fecha de extracción.
- Estado del procesamiento.

No se deben inventar campos específicos de una fuente como campos obligatorios globales si no tienen sentido para otras fuentes.

---

## 7. Comparación contra productos

Una de las funciones principales será determinar si una alerta puede estar relacionada con alguno de los productos registrados en la base de la organización.

La comparación puede considerar diferentes atributos, por ejemplo:

- Nombre del producto.
- Marca.
- Fabricante.
- Referencia.
- Modelo.
- Registro sanitario.
- Lote.
- Otros identificadores disponibles.

La lógica debe diseñarse de forma modular para permitir evolucionar desde coincidencias simples hacia mecanismos de comparación más robustos.

La aplicación debe diferenciar claramente entre:

- Coincidencia confirmada.
- Posible coincidencia.
- Sin coincidencia.
- Resultado pendiente de revisión.

No se debe presentar una coincidencia automática como confirmación definitiva cuando el algoritmo únicamente determine una similitud.

---

## 8. Revisión manual

Los resultados encontrados deben poder ser revisados por el usuario.

La interfaz debería permitir, como mínimo:

- Ver la alerta.
- Consultar la fuente original.
- Consultar los productos relacionados.
- Revisar el motivo de la coincidencia.
- Confirmar o descartar una relación.
- Agregar observaciones.
- Registrar el resultado de la revisión.

La revisión manual debe conservarse en el historial.

---

## 9. Trazabilidad e historial

La trazabilidad es un requisito importante del sistema.

Debe ser posible conocer:

- Cuándo se consultó una fuente.
- Qué información fue obtenida.
- Qué alertas fueron identificadas.
- Cuándo se detectó una alerta.
- Qué productos fueron relacionados.
- Qué resultado produjo la comparación.
- Qué usuario realizó una revisión, si existe autenticación.
- Qué decisión se tomó.
- Qué observaciones se registraron.
- Cuándo se modificó un resultado.

El sistema debe evitar perder información histórica simplemente porque una alerta deje de aparecer en el sitio de origen.

---

## 10. Interfaz web

La aplicación tendrá una interfaz web para consultar y administrar la información.

La interfaz deberá priorizar:

- Claridad.
- Simplicidad.
- Facilidad de revisión.
- Trazabilidad.
- Tablas y filtros.
- Estado de procesamiento.
- Resultados relevantes.

Funcionalidades previstas:

- Dashboard/resumen.
- Listado de alertas.
- Filtros.
- Búsqueda.
- Detalle de alerta.
- Productos.
- Coincidencias.
- Historial.
- Fuentes.
- Ejecuciones de consulta.
- Exportación de resultados.

La interfaz no debe condicionarse prematuramente a un framework frontend complejo si una solución más sencilla resulta suficiente para la primera versión.

---

## 11. Exportación

La aplicación debe permitir generar reportes.

Formatos previstos:

- **XLSX**
- **PDF**

La exportación debe conservar información suficiente para que el reporte pueda utilizarse como evidencia o registro operativo.

La generación de reportes debe estar separada de la lógica de extracción y comparación.

---

## 12. Arquitectura propuesta

La arquitectura debe ser modular.

Una estructura conceptual inicial podría ser:

```text
WebApp_AlertasSanitarias/
│
├── app/
│   ├── core/
│   ├── models/
│   ├── database/
│   ├── sources/
│   │   ├── invima/
│   │   └── ...
│   ├── scraping/
│   ├── matching/
│   ├── services/
│   ├── reports/
│   ├── api/
│   └── web/
│
├── tests/
│
├── data/
│
├── docs/
│
├── scripts/
│
├── .venv/
│
├── pyproject.toml
├── .gitignore
└── README.md
```

Esta estructura es conceptual y puede modificarse durante el desarrollo si una decisión arquitectónica mejor justificada resulta conveniente.

El principio importante es mantener separadas las responsabilidades.

---

## 13. Tecnologías previstas

### Backend

- Python
- FastAPI

### Persistencia / ORM

Se ha contemplado:

- SQLModel
- SQLite para desarrollo inicial
- PostgreSQL como alternativa para producción

La elección definitiva debe realizarse considerando el tamaño esperado, concurrencia, despliegue y necesidades reales del proyecto.

### Automatización web

- Playwright para Python
- Google Chrome y/o Microsoft Edge

### Procesamiento de datos

- Pandas, cuando resulte apropiado.

### Parsing HTML

- BeautifulSoup, cuando sea conveniente para contenido HTML que no requiera automatización de navegador.

### Plantillas / interfaz

Se han considerado:

- Jinja2
- HTMX
- Bootstrap 5

No es obligatorio utilizar todas estas tecnologías. Debe evitarse introducir dependencias innecesarias.

### Desarrollo y control de versiones

- Git
- Repositorio remoto
- Entorno virtual Python (`.venv`)

---

## 14. Versión de Python

El desarrollo local actual se realiza con:

```text
Python 3.12
```

El entorno virtual debe mantenerse aislado del Python global:

```text
.venv/
```

Las dependencias deben quedar declaradas de forma reproducible en el proyecto.

---

## 15. Dependencias y configuración

El proyecto debe evitar depender de instalaciones manuales no documentadas.

Toda dependencia necesaria debe quedar registrada en el mecanismo de gestión de dependencias adoptado.

Las configuraciones sensibles o dependientes del entorno no deben quedar hardcodeadas en el código.

Se debe considerar el uso de:

```text
.env
```

para configuración local cuando sea apropiado.

Los secretos, credenciales, tokens y otros datos sensibles **no deben almacenarse en Git**.

El `.gitignore` ya ha sido ajustado durante el desarrollo inicial para evitar subir archivos y directorios que no deben formar parte del repositorio.

---

## 16. Git y flujo de desarrollo

El proyecto utiliza Git para control de versiones.

Debe mantenerse un historial de cambios claro.

Se recomienda trabajar mediante ramas para funcionalidades o cambios relevantes y evitar realizar todo el desarrollo directamente sobre `main`.

Los commits deben describir claramente el cambio realizado.

No se deben incluir:

- Entornos virtuales.
- Secretos.
- Credenciales.
- Archivos temporales.
- Datos locales innecesarios.
- Artefactos generados automáticamente.

---

## 17. Estado actual

El proyecto se encuentra en **fase de desarrollo inicial / preparación de arquitectura**.

Ya se han realizado actividades relacionadas con:

- Creación del repositorio.
- Configuración inicial de Git.
- Creación del entorno virtual.
- Instalación/configuración de Python 3.12.
- Preparación de `.gitignore`.
- Definición preliminar de tecnologías.
- Preparación de documentación inicial.
- Investigación y pruebas iniciales relacionadas con Playwright.

Todavía deben desarrollarse progresivamente los componentes funcionales de la aplicación.

---

## 18. Despliegue

El objetivo final es disponer de una aplicación que pueda utilizarse en un entorno Windows y/o desplegarse en un entorno de producción.

**No asumir PyInstaller como solución de empaquetamiento.**

La decisión sobre:

- Ejecutable Windows.
- Instalador.
- Aplicación portable.
- Servicio.
- Contenedor Docker.
- Despliegue web.

queda abierta hasta evaluar las necesidades reales del proyecto.

El desarrollo debe evitar acoplar la aplicación a una estrategia de empaquetamiento específica.

---

## 19. Requisitos de calidad

El código debe priorizar:

- Legibilidad.
- Modularidad.
- Tipado cuando aporte valor.
- Manejo explícito de errores.
- Logging.
- Testabilidad.
- Separación de responsabilidades.
- Configuración externa.
- Reutilización.
- Mantenibilidad.

Debe evitarse:

- Código duplicado.
- Variables de configuración hardcodeadas.
- Scrapers monolíticos.
- Lógica de negocio dentro de la interfaz.
- Dependencias innecesarias.
- Soluciones excesivamente complejas para problemas simples.

---

## 20. Logging y diagnóstico

Las ejecuciones de extracción deben generar información suficiente para diagnosticar problemas.

Se debe poder identificar, como mínimo:

- Fuente consultada.
- Inicio y final de la ejecución.
- Cantidad de registros encontrados.
- Cantidad de registros procesados.
- Errores.
- Alertas nuevas.
- Alertas previamente conocidas.
- Coincidencias encontradas.

Los errores de una fuente no deberían provocar necesariamente el fallo completo de las demás fuentes.

---

## 21. Manejo de cambios en sitios web

Los sitios de organismos sanitarios pueden cambiar su HTML, estructura, URLs o mecanismos de consulta.

Por esta razón, cada scraper debe estar aislado y ser relativamente independiente.

Una modificación en el scraper de INVIMA no debería obligar a modificar el núcleo de la aplicación.

Se debe favorecer una arquitectura basada en interfaces/contratos comunes para los conectores de fuentes.

Conceptualmente:

```text
SourceConnector
    │
    ├── INVIMAConnector
    ├── OtraAgenciaConnector
    └── FuturoConnector
```

Todos deben producir datos compatibles con el modelo común de alerta.

---

## 22. Principios para el desarrollo con Codex

Codex debe utilizar este documento como **contexto inicial**, pero no asumir que todas las decisiones arquitectónicas están cerradas.

Antes de realizar cambios estructurales importantes:

1. Revisar el estado real del repositorio.
2. Revisar los archivos existentes.
3. Identificar qué funcionalidades ya están implementadas.
4. Evitar sobrescribir trabajo existente.
5. Mantener compatibilidad con las decisiones ya confirmadas.
6. Proponer cambios incrementales.
7. Explicar brevemente las decisiones técnicas relevantes.
8. No introducir tecnologías adicionales sin justificar su necesidad.
9. Mantener la documentación actualizada cuando una decisión cambie.
10. Priorizar una primera versión funcional antes de implementar características avanzadas.

---

## 23. Estrategia de desarrollo

El desarrollo debe realizarse incrementalmente.

Orden conceptual sugerido:

```text
1. Estructura del proyecto
        ↓
2. Configuración y dependencias
        ↓
3. Modelo de datos
        ↓
4. Base de productos
        ↓
5. Conector INVIMA
        ↓
6. Extracción y normalización
        ↓
7. Persistencia
        ↓
8. Motor de comparación
        ↓
9. Historial / trazabilidad
        ↓
10. API
        ↓
11. Interfaz web
        ↓
12. Exportación XLSX/PDF
        ↓
13. Pruebas
        ↓
14. Optimización y robustecimiento
        ↓
15. Estrategia de despliegue
```

Este orden es orientativo. Codex puede proponer modificaciones cuando exista una razón técnica clara.

---

## 24. Requisitos no negociables actualmente definidos

- Python **3.12** como versión de desarrollo actual.
- Utilizar entorno virtual.
- Playwright para automatización web cuando sea necesario.
- Utilizar **Chrome o Edge** como navegador objetivo.
- **No utilizar Chromium como navegador objetivo del proyecto.**
- INVIMA es la primera fuente objetivo.
- La arquitectura debe permitir incorporar otras fuentes.
- Debe existir trazabilidad histórica.
- Debe existir comparación contra una base de productos.
- Debe existir exportación a XLSX y PDF.
- La aplicación debe ser modular.
- No asumir PyInstaller para el empaquetamiento.
- No almacenar secretos en el repositorio.
- No acoplar el núcleo de la aplicación a un único scraper.

---

## 25. Decisiones aún abiertas

Las siguientes decisiones deben evaluarse durante el desarrollo y no deben considerarse cerradas:

- SQLite vs. PostgreSQL para producción.
- Estrategia definitiva de autenticación/autorización.
- Framework definitivo de interfaz.
- Necesidad real de HTMX.
- Estrategia de ejecución programada de consultas.
- Sistema de notificaciones.
- Estrategia definitiva de despliegue.
- Necesidad de ejecutable Windows.
- Uso de Docker en producción.
- Nivel de automatización de la coincidencia de productos.
- Estrategia de actualización de alertas previamente registradas.
- Arquitectura definitiva de plugins/conectores para nuevas agencias.

---

## 26. Criterio general de implementación

La prioridad es construir una aplicación **funcional, confiable y mantenible**, no una arquitectura innecesariamente compleja.

Cada componente debe resolver una responsabilidad concreta.

La aplicación debe poder evolucionar progresivamente desde una primera versión funcional con INVIMA hacia un sistema de monitoreo de alertas sanitarias con múltiples fuentes.

Antes de agregar una funcionalidad, evaluar:

- ¿Es necesaria?
- ¿Aporta valor operativo?
- ¿Aumenta innecesariamente la complejidad?
- ¿Puede implementarse de forma modular?
- ¿Afecta la trazabilidad?
- ¿Puede probarse de forma automatizada?
- ¿Queda desacoplada de una fuente específica?

---

## 27. Primera tarea recomendada para Codex

Antes de comenzar a programar nuevas funcionalidades:

1. Analizar el repositorio actual.
2. Identificar la estructura existente.
3. Revisar `README.md`, `pyproject.toml`, `.gitignore` y demás archivos relevantes.
4. Revisar la documentación existente.
5. Identificar qué partes del proyecto ya están implementadas.
6. Comparar el estado real del repositorio con este documento.
7. Presentar un diagnóstico breve.
8. Proponer el siguiente paso de implementación.
9. No realizar cambios destructivos ni reestructuraciones masivas sin justificación.

El desarrollo debe comenzar desde el estado real del repositorio, no desde una recreación hipotética del proyecto.
