---
name: auditar-modelo-bim
description: >
  Use when the user wants to audit a Revit BIM model before performing architectural quantity takeoffs (metrados) in Peru. Evaluates model readiness across categories, phases, levels, families, materials, rooms, and parameters. Outputs a traffic-light report (LISTO / REQUIERE AJUSTE / NO MODELADO / NO VERIFICABLE) with element IDs and recommendations. Read-only — never modifies the model. Must be the first step in the 5D workflow before extraer-metrados-arquitectura.
version: 1.0.0
author: Antigravity
license: MIT
metadata:
  hermes:
    tags: [bim, revit, metrados, auditoria, arquitectura, peru, oe3]
    related_skills: [extraer-metrados-arquitectura, clasificar-partidas-oe3, generar-apu, exportar-presupuesto-excel]
---

# Auditar Modelo BIM

## Overview

Evalúa si un modelo Revit está preparado para extraer metrados de arquitectura conformes al Reglamento Nacional de Metrados del Perú (RNM). Usa exclusivamente las 15 herramientas confirmadas del MCP `rvt-mcp` v0.5.0. No modifica el modelo. Genera un informe estructurado con semáforo por categoría e IDs de incidencias.

**Herramientas MCP disponibles y confirmadas (rvt-mcp v0.5.0):**

| Herramienta | Capacidad |
|---|---|
| `get_current_view_info` | Vista activa, tipo, escala, nivel |
| `analyze_model_statistics` | Estadísticas globales, conteo por categoría |
| `list_phases` | Fases y filtros de fase del proyecto |
| `list_schedules` | Tablas de planificación existentes |
| `get_schedule_data` | Datos de una tabla (paginado, startRow/maxRows) |
| `list_materials` | Materiales definidos, clase, activos |
| `list_loaded_families` | Familias cargadas en el proyecto |
| `list_linked_models` | Modelos Revit vinculados |
| `list_linked_cad` | Archivos CAD vinculados o importados |
| `get_link_elements` | Elementos dentro de un modelo vinculado |
| `activate_view` | Activa una vista o tabla por nombre |
| `capture_view_image` | Captura imagen de la vista activa |
| `send_code_to_revit` | Ejecuta C# en Revit (solo lectura permitida) |
| `show_message` | Muestra mensaje en Revit |
| `workflow_model_audit` | Auditoría integrada del servidor MCP |

> ⚠️ **Restricción de sólo lectura:** `send_code_to_revit` puede usarse para leer datos no accesibles por otras herramientas (niveles, habitaciones, parámetros). Está PROHIBIDO usarlo para crear, modificar o eliminar elementos. Toda transacción debe fallar deliberadamente si intenta escritura.

## When to Use

- El usuario quiere iniciar un proceso de metrados 5D desde un modelo Revit.
- Antes de ejecutar `extraer-metrados-arquitectura`.
- Cuando se sospecha que el modelo tiene problemas de calidad BIM.
- Cuando se recibe un modelo de terceros y se necesita evaluar su estado.

**No usar para:**
- Modelos en formato IFC, SketchUp, AutoCAD sin conversión previa a Revit.
- Modelos de estructuras o instalaciones sin disciplina de arquitectura.

## Inputs Obligatorios

Antes de ejecutar, confirmar con el usuario:

1. **Revit abierto** con el modelo a auditar activo.
2. **rvt-mcp activo** (verificar conexión con `get_current_view_info`).
3. **Disciplina objetivo:** Arquitectura (OE.3 RNM).
4. **¿El usuario tiene el Reglamento Nacional de Metrados disponible?** (necesario para skill 3).

## Procedimiento Paso a Paso

### Paso 1 — Verificar conexión MCP
```
get_current_view_info → confirmar respuesta válida
```
**Criterio de completitud:** Se recibe `viewId`, `viewName`, `viewType` sin error. Si falla, detener y reportar: "MCP no conectado. Verificar que Revit esté abierto con el plugin rvt-mcp activo."

### Paso 2 — Estadísticas globales del modelo
```
analyze_model_statistics → capturar projectName, elementsCounted, totalCategories, categories[]
```
Registrar:
- Nombre del proyecto
- Total de elementos
- Categorías presentes y su conteo
- Categorías arquitectónicas clave: Muros, Suelos, Techos, Puertas, Ventanas, Habitaciones, Barandas, Paneles de muro cortina, Montantes de muro cortina

### Paso 3 — Fases del proyecto
```
list_phases → capturar phases[], phase_filters[]
```
Evaluar:
- ¿Existe fase "Nueva Construcción" o equivalente?
- ¿Hay fases con nombres genéricos (Fase 1, Phase 1)?
- ¿El número de fases es coherente con el alcance del proyecto?

### Paso 4 — Familias cargadas
```
list_loaded_families → capturar todas las familias
```
Detectar:
- Familias con nombre genérico: "Familia básica", "Basic Wall", "Generic Model"
- Familias sin tipo asignado
- Familias de sistema vs familias cargadas

### Paso 5 — Materiales
```
list_materials(include_assets=true, include_use_count=false, limit=100)
```
Si `limit_hit=true`, iterar con `skip` hasta obtener todos.
Detectar:
- Materiales sin clase definida (`material_class = "Não atribuído"` o equivalente)
- Materiales sin activo de apariencia
- Materiales con nombres genéricos (Material, Default)
- Total de materiales vs categorías activas

### Paso 6 — Tablas de planificación existentes
```
list_schedules(categoryFilter="", namePattern="")
```
Registrar:
- Tablas existentes por categoría
- Si existen tablas de: Muros, Suelos, Puertas, Ventanas, Habitaciones
- Número de campos por tabla

### Paso 7 — Lectura de datos de tablas relevantes
Para cada tabla identificada con categorías arquitectónicas:
```
get_schedule_data(scheduleName="<nombre>", startRow=0, maxRows=500)
```
Si `truncated=true`, paginar: `startRow=500, maxRows=500`, etc.
Registrar columnas disponibles y detectar celdas vacías en campos clave.

### Paso 8 — Consultas C# de solo lectura (elementos sin tabla)
Para datos no accesibles por otras herramientas (habitaciones, niveles, parámetros de instancia), usar `send_code_to_revit` con código de **solo lectura**:

```csharp
// PLANTILLA SOLO LECTURA — niveles
var levels = new FilteredElementCollector(doc)
    .OfClass(typeof(Level))
    .Cast<Level>()
    .Select(l => new { 
        Id = l.Id.Value, 
        Name = l.Name, 
        Elevation = l.Elevation 
    });
return string.Join("\n", levels.Select(l => $"{l.Id}|{l.Name}|{l.Elevation:F3}"));
```

> ⚠️ Toda consulta C# debe ser SELECT-only. Prohibido: `doc.Create.*`, `Transaction`, `Element.Delete`, setters de parámetros.

Consultas de solo lectura permitidas en este paso:
- Niveles y elevaciones
- Habitaciones: nombre, número, área, nivel, si están cerradas (Volume > 0)
- Parámetros de instancia de muros, pisos, techos con IDs

### Paso 9 — Captura de imagen (opcional)
```
capture_view_image → capturar vista 3D o planta para referencia visual
```

### Paso 10 — Detectar modelos y CAD vinculados
```
list_linked_models
list_linked_cad(include_imports=true, include_links=true)
```
Registrar vínculos activos y advertir si elementos de acabado pueden estar en modelo vinculado.

## Esquema de Salida Estructurado

```markdown
# INFORME DE AUDITORÍA BIM
Proyecto: [nombre]          Fecha: [ISO 8601]
Software: Revit [año]       Versión MCP: 0.5.0
Modelo: [nombre archivo]    Elementos totales: [N]

---

## RESUMEN EJECUTIVO
[3-5 líneas: estado general, categorías críticas, acción inmediata]

---

## TABLA DE CATEGORÍAS ARQUITECTÓNICAS

| Categoría       | Instancias | Estado          | Observación |
|-----------------|-----------|-----------------|-------------|
| Muros           | 73        | LISTO           |             |
| Suelos          | 22        | REQUIERE AJUSTE | Sin material|
| Techos          | 0         | NO MODELADO     |             |
| Puertas         | 23        | LISTO           |             |
| Ventanas        | 41        | LISTO           |             |
| Habitaciones    | 51        | REQUIERE AJUSTE | 3 abiertas  |
| Barandas        | 12        | LISTO           |             |
| Paneles MC      | 3372      | REQUIERE AJUSTE |             |
| Cubiertas       | 0         | NO MODELADO     |             |
| Cielorrasos     | 0         | NO VERIFICABLE  | Sin tabla   |

---

## LISTA DE INCIDENCIAS

| ID | Categoría | Elemento ID | Tipo | Incidencia | Severidad |
|----|-----------|-------------|------|-----------|-----------|
| 1  | Suelos    | 456789      | Piso | Sin material asignado | ALTA |
| 2  | Habitaciones | 123456  | Sala | No cerrada (V=0) | MEDIA |

---

## SEMÁFORO GLOBAL
- ✅ LISTO: Muros, Puertas, Ventanas, Barandas
- ⚠️ REQUIERE AJUSTE: Suelos, Habitaciones, Paneles MC
- ❌ NO MODELADO: Techos, Cubiertas, Cielorrasos
- ℹ️ NO VERIFICABLE: Zócalos, Contrazócalos

---

## FASES
[lista de fases con IDs y evaluación]

## MATERIALES
Total: [N] | Sin clase: [N] | Genéricos: [N]

## FAMILIAS
Total: [N] | Genéricas: [N] | Sin tipo: [N]

## TABLAS EXISTENTES
[lista con nombre, categoría, campos]

## MODELOS VINCULADOS
[lista o "Ninguno"]

---

## RECOMENDACIONES (sin modificar el modelo)
1. [acción específica por el usuario en Revit]
2. ...

## PREREQUISITOS PARA SIGUIENTE SKILL
- [ ] Categorías marcadas como LISTO o REQUIERE AJUSTE: [lista]
- [ ] Materiales asignados en elementos a metrar: [estado]
- [ ] Habitaciones cerradas: [estado]
```

## Reglas de Validación

- **No asumir** que una categoría existe si `analyze_model_statistics` no la muestra.
- Si `count=0` para una categoría, marcar como `NO MODELADO`, no como error.
- Si no hay tabla para una categoría, marcar como `NO VERIFICABLE`.
- Un elemento con `Volume=0` en habitaciones indica **no cerrada** → incidencia MEDIA.
- Materiales con nombre en idioma diferente al proyecto (ej. "Bois", "Liquid") → incidencia BAJA (probable plantilla de biblioteca).
- Familias con "Generic" o "Genérico" en el nombre → incidencia MEDIA.
- **Parar** si `get_current_view_info` falla: sin MCP no hay auditoría.

## Condiciones para Detenerse

1. **MCP no conectado** — rvt-mcp no responde → detener, indicar causa.
2. **Modelo vacío** — `elementsCounted < 10` → detener, modelo no válido.
3. **Sin categorías arquitectónicas** — ninguna de las 10 categorías clave presente → detener, posible modelo de otra disciplina.
4. **Error de compilación en `send_code_to_revit`** → registrar error, continuar con datos disponibles de otras herramientas, marcar como NO VERIFICABLE.

## Limitaciones Conocidas

| Limitación | Causa | Alternativa |
|---|---|---|
| IDs de elementos individuales no accesibles sin `send_code_to_revit` | Herramientas de listado no exponen IDs de instancia | Usar C# de solo lectura |
| `get_schedule_data` no retorna ElementId por fila | Diseño del MCP | Crear tabla con campo ID en Revit si se necesita trazabilidad exacta |
| Cielorrasos requieren tabla preexistente o C# | No hay herramienta directa | Marcar NO VERIFICABLE si no existe tabla |
| Elementos en fases incorrectas no detectables sin C# | `analyze_model_statistics` no filtra por fase | Usar `send_code_to_revit` con filtro de fase |
| `list_materials` retorna máx. `limit` por llamada, iterar si `limit_hit=true` | Paginación del MCP | Iterar con `skip` |
| Sin acceso a parámetros de proyecto (unidades, autor) por herramientas directas | No implementado en v0.5.0 | Usar `send_code_to_revit` para leer `ProjectInfo` |
| SketchUp no soportado | No existe MCP SketchUp en este entorno | Solo Revit |

## Ejemplo de Uso

**Input del usuario:**
> "Audita el modelo Casa 2 pisos que tengo abierto en Revit"

**Secuencia de llamadas:**
1. `get_current_view_info` → confirma conexión
2. `analyze_model_statistics` → 13 966 elementos, 81 categorías
3. `list_phases` → 2 fases detectadas
4. `list_loaded_families` → 45 familias
5. `list_materials(limit=100)` → 241 materiales, `limit_hit=true` → iterar
6. `list_schedules` → 2 tablas existentes (Muros)
7. `get_schedule_data(scheduleName="Metrado de Materiales de Muros", startRow=0, maxRows=500)` → 671 filas, truncated
8. `get_schedule_data(startRow=500, maxRows=500)` → 171 filas restantes
9. `send_code_to_revit` → niveles, habitaciones (solo lectura)
10. Generar informe estructurado

## Prueba Mínima de Funcionamiento

```
1. Abrir cualquier modelo .rvt en Revit 2026
2. Activar rvt-mcp (verificar que el plugin esté cargado)
3. Invocar esta skill
4. Resultado esperado mínimo:
   - get_current_view_info retorna viewId válido
   - analyze_model_statistics retorna al menos 1 categoría
   - El informe incluye sección SEMÁFORO con al menos 1 entrada
5. Si analyze_model_statistics retorna error → FALLA DE PRUEBA
```

## Common Pitfalls

1. **Inventar nombres de herramientas.** Solo usar las 15 listadas. Si una operación no es posible, declararla como LIMITACIÓN.
2. **Asumir que `send_code_to_revit` siempre compila.** La API de Revit cambia entre versiones. Siempre manejar `success=false` y continuar.
3. **No paginar `get_schedule_data`.** Si `truncated=true`, SIEMPRE continuar desde `startRow` siguiente hasta `truncated=false`.
4. **Confundir `get_link_elements` con elementos del modelo activo.** Esta herramienta accede a modelos vinculados, no al modelo abierto.
5. **Marcar como error una categoría con count=0.** Es información válida: el elemento simplemente no fue modelado.

## Verification Checklist

- [ ] `get_current_view_info` respondió sin error antes de continuar
- [ ] `analyze_model_statistics` ejecutado y categorías registradas
- [ ] `list_phases` ejecutado y fases documentadas
- [ ] `list_materials` completamente paginado (sin `limit_hit` sin resolver)
- [ ] `get_schedule_data` paginado hasta `truncated=false` para cada tabla relevante
- [ ] Semáforo con las 10 categorías mínimas completado
- [ ] Lista de incidencias incluye IDs cuando fue posible obtenerlos
- [ ] Ninguna herramienta inventada o no listada usada
- [ ] Sin modificaciones al modelo
- [ ] Recomendaciones formuladas sin ordenar al agente que modifique el modelo
