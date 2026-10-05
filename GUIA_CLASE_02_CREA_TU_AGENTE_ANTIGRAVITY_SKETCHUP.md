# GUÍA DE TALLER EN VIVO — CLASE 02
## Asignatura: Inteligencia Artificial Arquitectura (ARQU-373) — UPAO 2026-20
**Docente:** Mg. Arq. Juan Miguel Angulo García  
**Tema:** "Creación de tu Primer Agente en Antigravity y Conexión Ciberfísica con SketchUp MCP"  
**Duración:** 90 minutos sincrónicos  

---

## 🎯 OBJETIVO DE LA CLASE DE HOY
Hoy dejarás de usar la Inteligencia Artificial como un simple chat de texto. Al finalizar esta sesión:
1. Tendrás tu propio **Agente de Arquitectura** configurado dentro de **Antigravity**.
2. Tu agente tendrá instaladas **10 superpoderes (skills)** especializadas en diseño, normativa peruana (RNE) y construcción.
3. Conectarás tu agente directamente a **SketchUp** mediante el protocolo **MCP (Model Context Protocol)**.
4. Convertirás un concepto visual (generado con IA o a partir de una captura/boceto a mano) en un **modelo tridimensional editable en SketchUp**, sin tocar una sola herramienta de dibujo manual.

---

## 🛠️ PASO 1: LA ANATOMÍA DE TU AGENTE (ESTRUCTURA DE CARPETAS)

Crea en tu computadora la carpeta de tu proyecto de tesis o taller de diseño. Dentro de ella, tu agente vivirá en una carpeta oculta llamada `.agents`:

```
mi-proyecto-arquitectura/
├── .agents/
│   ├── rules/
│   │   └── 00_reglas_estudio.md       <-- Define el estilo y normas de tu estudio
│   └── skills/                         <-- Las 10 herramientas especializadas
│       ├── optimizador-prompts/
│       ├── superpowers/
│       ├── verificacion-normativa-rfg/
│       └── ... (Starter Kit)
├── anteproyecto_sketchup.skp          <-- El archivo que tu agente modelará
└── bocetos/                            <-- Tus capturas o renders conceptuales
```

---

## 🔌 PASO 2: CONECTAR EL SERVIDOR SKETCHUP MCP

En Antigravity, el protocolo **Model Context Protocol (MCP)** permite que el modelo de lenguaje "tenga manos" para interactuar con tu software de diseño.

### Las herramientas que tu agente usará hoy en SketchUp:
- `create_component`: Crea grupos, cajas, muros o pórticos con dimensiones exactas en milímetros/metros.
- `transform_component`: Mueve, rota o eleva los volúmenes a su cota de nivel ($Z$).
- `set_material`: Aplica texturas y colores arquitectónicos (concreto, madera, vidrio, césped).
- `get_selection`: Lee qué elementos tienes seleccionados en tu pantalla de SketchUp.

---

## 🎨 PASO 3: EL FLUJO CREATIVO (DEL CONCEPTO AL MODELO 3D)

### Fase A: Creación de la Atmósfera Visual (ImageFX / Google / Boceto)
1. Toma una captura de pantalla de un croquis tuyo a mano alzada, o genera una imagen conceptual con el Framework PARTS:
   ```text
   [PERSONA]: Arquitecto contemporáneo minimalista.
   [ACCIÓN]: Genera una vista axonométrica isométrica de un pabellón cultural de 2 niveles con estructura de pórticos de madera laminada, grandes vanos acristalados y terraza ajardinada.
   [RELEVANCIA]: Terreno en pendiente suave, clima templado.
   [TONO]: Fotografía de maqueta arquitectónica hiperrealista, iluminación suave de atardecer.
   [SUJETO]: Pabellón de exposiciones de 12 m x 20 m.
   ```

### Fase B: El Agente traduce la Imagen a Geometría
Pásale la imagen o captura de pantalla a tu agente en Antigravity con esta instrucción:

```text
"Observa este concepto arquitectónico. Extrae su modulación estructural (ancho, largo y altura de entrepiso). 
Conéctate a mi sesión de SketchUp mediante MCP y levanta los volúmenes principales:
1. La losa de cimentación de 12 m x 20 m x 0.30 m.
2. Los 6 pórticos estructurales principales espaciados a 4.00 m.
3. La cubierta inclinada con alero de 1.50 m.
Asigna materiales diferenciados para madera y vidrio."
```

### Fase C: Ejecución y Supervisión
- Observa cómo el agente razona: calcula las coordenadas $X, Y, Z$.
- El agente llama sucesivamente a `create_component` y `transform_component`.
- **¡SketchUp cobra vida en tu pantalla y la maqueta 3D aparece modelada en segundos!**

---

## 📋 PASO 4: ENTREGA PARA CANVAS UPAO (EVIDENCIA DE CLASE 2)

Al terminar la clase, debes subir a la tarea de Canvas:
1. **Captura 1:** Tu concepto visual o boceto inicial.
2. **Captura 2:** Tu pantalla mostrando Antigravity llamando a las herramientas de SketchUp MCP y la maqueta 3D resultante en SketchUp.
3. **Archivo de texto (`prompt_usado.txt`):** El prompt PARTS que le diste a tu agente para lograr el modelado.

---

> [!TIP]
> **Regla de Oro del Arquitecto 4.0:**
> Si el agente comete un error en las medidas, no lo arregles con el mouse en SketchUp. Dile al agente: *"El pórtico 3 está desalineado 0.50 m en el eje Y; corrígelo con transform_component"*. Así entrenas tu capacidad de dirección agéntica.
