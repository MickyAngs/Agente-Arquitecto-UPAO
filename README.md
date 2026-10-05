# 🏛️ Agente Arquitecto UPAO — Starter Kit Oficial
## Asignatura: Inteligencia Artificial Arquitectura (ARQU-373)
**Universidad Privada Antenor Orrego (UPAO) · Semestre 2026-20**  
**Cátedra:** Mg. Arq. Juan Miguel Angulo García *(CAP 6915)*  
**Plataforma Base:** Google Antigravity IDE · Model Context Protocol (MCP)

---

## 🎯 ¿Para qué sirve este repositorio?

Este kit transforma a tu entorno de **Google Antigravity** en un **Agente Director de Arquitectura** capaz de razonar, auditar normativa y conectarse en tiempo real con tu software de modelado (**Trimble SketchUp** y **Autodesk Revit**).

Dejamos atrás los chats pasivos de texto. Con este kit:
1. **Le das tu concepto o boceto** en lenguaje natural.
2. **Tu agente aplica normativa peruana (RNE / CAPECO / RNM)** y criterios bioclimáticos.
3. **Modela volúmenes 3D automáticamente** vía servidores MCP.

---

## 🚀 Instalación Rápida en 3 Pasos (Sin Comandos Complejos)

### Paso 1: Descargar el Repositorio
* Haz clic en el botón verde superior **`Code`** ➔ **`Download ZIP`**.
* Descomprime el archivo en tu computadora (ejemplo: `D:\Tesis_2026\` o `C:\Taller_Diseno\`).

### Paso 2: Abrir la Carpeta en Google Antigravity
* Abre **Google Antigravity IDE**.
* Ve a `File` ➔ `Open Folder` y selecciona la carpeta descomprimida.
* Antigravity detectará en automático la carpeta `.agents/skills` con los **20 superpoderes**.

### Paso 3: Conectar a SketchUp (Clase 02)
* Abre **Trimble SketchUp** en tu laptop.
* Copia el contenido de `mcp_config_ejemplo.json` en tu configuración de MCP.
* Escribe en el prompt de Antigravity:
  > *"Actúa como arquitecto director. Modela un pabellón de mediateca de 12x8m con pórticos de concreto y celosías solares orientadas al norte en SketchUp."*
* ¡Observa cómo el agente construye el modelo 3D directamente en SketchUp!

---

## ⚡ Los 20 Superpoderes (*Skills*) Incluidos

Diseñados específicamente a partir de los resultados de la **Encuesta Diagnóstica de 9no Ciclo**:

### 📐 Bloque 1: Modelado Agéntico & MCP (Sesión 02 y Taller)
1. **`optimizador-prompts`**: Estructura instrucciones maestras bajo el **Framework PARTS** (*Persona, Acción, Relevancia, Tono, Sujeto*).
2. **`superpowers`**: Planificación metódica y razonamiento paso a paso anti-alucinaciones.
3. **`explicador-tecnico-visual`**: Traduce conceptos complejos a diagramas y esquemas arquitectónicos.
4. **`archify`**: Diagramación de sistemas arquitectónicos, recorridos, flujos funcionales y esquemas técnicos.
5. **`auditoria-planos-dwg-rvt`**: Compatibilización dimensional entre planos 2D de AutoCAD y modelos 3D de Revit.
6. **`automatizacion-autolisp`**: Genera rutinas en AutoLISP para acotado paramétrico y tareas repetitivas en AutoCAD.

### 🎓 Bloque 2: Proyecto de Tesis & Taller X (Demanda #1: 76.5% de estudiantes)
7. **`redaccion-conclusiones-tesis`**: Redacción académica rigurosa, contraste de hipótesis y discusión metodológica.
8. **`revision-anteproyectos-mediateca`**: Auditoría de programas funcionales, accesibilidad universal y zonificación.
9. **`humanizer`**: Purga lenguaje detectable de chatbot y blindaje de originalidad para **Turnitin**.

### ☀️ Bloque 3: Sostenibilidad, Confort & Bioclimática Pasiva
10. **`verificacion-normativa-rfg`**: Cumplimiento de confort higrotérmico y factores de diseño según RNE (A.010 y EM.110).
11. **`optimizacion-radiacion-solar`**: Análisis de asoleamiento, sombras, aleros y celosías *brise-soleil*.
12. **`calculo-convectivo-venturi`**: Cálculo de tiro térmico, efecto chimenea y toberas de ventilación pasiva.
13. **`sistemas-pasivos-heliostatos`**: Captación e iluminación cenital pasiva para espacios profundos o enterrados.
14. **`auditoria-sustentable-muba`**: Balance de ganancias de calor y envolventes bioclimáticas pasivas.

### 🏗️ Bloque 4: Construcción Civil, Metrados 5D & Costos (Demanda #2: 70.6%)
15. **`auditar-modelo-bim`**: Diagnóstico semafórico previo a metrados (*LISTO / REQUIERE AJUSTE*).
16. **`clasificar-partidas-oe3`**: Clasificación normativa según el **Reglamento Nacional de Metrados (RNM)**.
17. **`extraer-metrados-arquitectura`**: Cómputo métrico exacto de muros, vanos y acabados con trazabilidad por Element ID.
18. **`generar-apu`**: Elaboración de Análisis de Precios Unitarios bajo metodología oficial **CAPECO**.
19. **`exportar-presupuesto-excel`**: Generación de presupuestos dinámicos en hojas de cálculo con fórmulas vivas.
20. **`agente-supervisor-obras-ssoma`**: Control de calidad en concreto armado (RNE E.060) y seguridad en obra (NTP G.050).

---

## 📋 Estructura del Repositorio

```text
├── .agents/
│   └── skills/                  <-- 20 carpetas de skills listas para Antigravity
│       ├── auditar-modelo-bim/
│       ├── automatizacion-autolisp/
│       ├── calculo-convectivo-venturi/
│       ├── clasificar-partidas-oe3/
│       ├── explicador-tecnico-visual/
│       ├── exportar-presupuesto-excel/
│       ├── extraer-metrados-arquitectura/
│       ├── generar-apu/
│       ├── humanizer/
│       ├── optimizacion-radiacion-solar/
│       ├── optimizador-prompts/
│       ├── redaccion-conclusiones-tesis/
│       ├── revision-anteproyectos-mediateca/
│       ├── sistemas-pasivos-heliostatos/
│       ├── superpowers/
│       └── ...
├── GUIA_CLASE_02_CREA_TU_AGENTE_ANTIGRAVITY_SKETCHUP.md  <-- Taller práctico paso a paso
├── GUIA_INSTALACION_OMNIROUTE_LOCAL_ESTUDIANTES.md       <-- Conexión de modelos gratuitos locales
├── LEEME_PRIMERO_INSTALACION.md                         <-- Guía rápida en texto plano
├── mcp_config_ejemplo.json                              <-- Configuración MCP SketchUp / Revit
└── README.md                                            <-- Esta documentación oficial
```

---

## 🏛️ Créditos y Autoría
**Cátedra de Inteligencia Artificial Aplicada a la Arquitectura**  
Facultad de Arquitectura, Urbanismo y Arte — UPAO  
Docente Titular: **Mg. Arq. Juan Miguel Angulo García** (CAP 6915)  
Trujillo / Piura, Perú · 2026
