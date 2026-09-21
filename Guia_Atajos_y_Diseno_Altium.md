# 🚀 Guía de Atajos de Teclado, Trucos y Buenas Prácticas en Altium Designer

Esta guía reúne los comandos esenciales, configuraciones óptimas del entorno de trabajo y metodologías recomendadas para optimizar el flujo de diseño tanto en el **Editor de Esquemáticos (SCH)** como en el **Editor de Circuitos Impresos (PCB)**.

---

## 📐 1. Editor de Esquemáticos (Schematic)

El objetivo en el esquemático es mantener la claridad visual para que cualquier ingeniero o fabricante pueda interpretar el circuito sin ambigüedades.

### ⌨️ Atajos de Teclado Esenciales en SCH
* **`P` + `W` (Place Wire):** Activa la herramienta para colocar hilos de conexión eléctrica.
* **`P` + `N` (Place Net Label):** Coloca una etiqueta de red para interconectar nodos eléctricamente sin necesidad de líneas físicas continuas.
* **`P` + `P` (Place Part):** Abre el panel para buscar e instanciar componentes de las bibliotecas activos.
* **`P` + `G` (Place GND):** Atajo directo para colocar el símbolo de referencia de tierra.
* **`P` + `O` (Place Power Port):** Coloca un puerto de alimentación (VCC, 3V3, 5V, etc.).
* **`TAB` (Al colocar un objeto):** Pausa la colocación del objeto y abre de inmediato sus propiedades en el panel derecho sin cancelar la herramienta.
* **`Barra Espaciadora`:** Rota el símbolo 90 grados en sentido antihorario mientras se arrastra.
* **`X` / `Y` (Al arrastrar):** Aplica un efecto espejo (Mirror) horizontal (X) o vertical (Y) al componente.

### ⚙️ Configuraciones Recomendadas en SCH
* **Tamaños de Rejilla (Grid):** Mantén siempre el **Grid de captura en 100 mils** o **50 mils**. Nunca utilices medidas impares como 7 o 13 mils para colocar pines o cables; de lo contrario, los hilos no se alinearán con los pines de los componentes y generarás errores de conexiones abiertas (*Floating Pins*) difíciles de detectar en la compilación.
* **Cambio rápido de Grid:** Presiona **`G`** para ciclar rápidamente entre los tamaños de rejilla preconfigurados.

---

## ⚡ 2. Editor de PCB (Layout)

El diseño de la PCB requiere precisión milimétrica, control de capas y gestión de restricciones físicas (Rules).

### ⌨️ Atajos de Teclado Esenciales en PCB
* **`Ctrl` + `W` (Interactive Routing):** Inicia el enrutamiento interactivo de pistas.
* **`TAB` (Al enrutar una pista):** Abre las propiedades de la pista (grosor, net asignada). Si lo presionas sobre una pista ya colocada, expande la selección paso a paso: primero el segmento, luego la conexión continua y finalmente la red completa.
* **`Barra Espaciadora`:** Alterna la dirección de la esquina de la pista mientras enrutas.
* **`Shift` + `Espacio`:** Cicla entre los diferentes **modos de esquinas** (Esquinas a 45°, 90°, Arcos redondeados o trazo libre).
* **`Shift` + `R`:** Cambia el modo de colisión en tiempo real (*Walkaround* [Esquivar], *Push* [Empujar pistas existentes], *Hug* [Ceñirse al obstáculo] o *Ignore*). 
* **`+` / `-` (Teclado numérico):** Cambia a la capa de cobre superior o inferior. Si estás enrutando, inserta automáticamente una **Vía** de interconexión.
* **`Shift` + `S` (Single Layer Mode):** Oculta o atenúa el contenido de todas las capas para visualizar únicamente la capa activa. Crucial para placas densas multicapa.
* **`Q`:** Alterna instantáneamente las unidades del espacio de trabajo entre el sistema métrico (**Milímetros**) y el sistema imperial (**Mils**).
* **`F3`:** Cambia instantáneamente de la vista de diseño 2D al modelado **3D realista**. Presiona **`2`** para volver a 2D y **`0`** para nivelar la cámara en 3D de forma perpendicular.
* **`Shift` + `C`:** Limpia los filtros aplicados y restablece los colores brillantes tras realizar consultas avanzadas o búsquedas.

---

## 🛠️ 3. Configuraciones Avanzadas y de Entorno (Estilo Mouse)

Tal como se configuró el Zoom directo con la rueda del ratón sin usar la tecla Ctrl, existen otras personalizaciones críticas para mejorar la ergonomía del software:

### 🖱️ Desplazamiento Lateral Dinámico (Pan)
Si reasignaste la rueda del ratón (`Wheel`) para hacer Zoom directo, puedes mapear los desplazamientos de la siguiente manera dentro de **Preferences > PCB Editor > Mouse Wheel Configuration**:
* **`Shift` + `Wheel`:** Configúralo para realizar el desplazamiento horizontal (*Scroll Left/Right*).
* **`Ctrl` + `Wheel`:** Configúralo para el desplazamiento vertical tradicional (*Scroll Up/Down*), invirtiendo el orden por defecto.
* **Botón derecho sostenido:** Mantén presionado el botón derecho del ratón y arrastra para desplazarte libremente por todo el lienzo (*Pan*).

### 🎯 Gestión de Filtros de Selección (Selection Filter)
Usa el panel de **General Shortcuts** o la barra superior de selección para restringir qué tipo de objetos puede tocar el ratón. Si vas a mover componentes, desactiva la selección de *Tracks* (pistas) y *Polygons* (polígonos). Esto evita desarmar conexiones por accidente cuando seleccionas grandes áreas arrastrando el cursor.

### 📐 Modos de Guías Inteligentes (Snap Options)
Accede mediante el menú **Tools > Options > Board Options** (o presionando `Ctrl` + `G`):
* Activa **Snap to Object Center**: Forza al cursor a engancharse al centro geométrico exacto de los pads y componentes. Al trazar pistas, esto garantiza un acabado profesional y conexiones simétricas.

---

## 📋 4. Lista de Verificación (Checklist) para un Diseño Exitoso

1. **En Esquemático:** Realiza siempre una verificación de reglas eléctricas mediante **Project > Validate PCB Project** antes de transferir los datos al layout para asegurar que no existan cortos ni pines al aire.
2. **Distribución de Componentes:** Ubica primero los componentes críticos de interfaz (conectores, pulsadores, LEDs) y los integrados principales (MCUs, FPGAs). Posteriormente, posiciona sus componentes de desacoplo (condensadores caps) lo más cerca posible de sus pines de alimentación.
3. **Plano de Masa (GND):** En lo posible, dedica una capa interna completa o realiza un vaciado de polígono de cobre (*Polygon Pour*) conectado a la red GND para reducir las interferencias electromagnéticas y asegurar la estabilidad de la señal.
4. **Verificación DRC (Design Rule Check):** Corre el análisis DRC en **Tools > Design Rule Check > Run Design Rule Check** al finalizar el diseño de la PCB para validar que no existan violaciones de distancias mínimas (*Clearance*), anchos de pista inválidos o colisiones mecánicas.
