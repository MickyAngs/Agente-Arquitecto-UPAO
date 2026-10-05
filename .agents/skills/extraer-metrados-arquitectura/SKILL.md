---
name: extraer-metrados-arquitectura
description: >
  Use when the user wants to extract architectural quantities (metrados) from an open Revit model using MCP tools, with full traceability to element IDs. Covers walls, floors, ceilings, doors, windows, roofs, rooms, railings, curtain walls and finish elements. Read-only mode: never modifies the model. Requires a passing audit from auditar-modelo-bim first. Outputs structured records with source, validation status, and gross/deduction/net quantities per element.
version: 1.0.0
author: Antigravity
license: MIT
metadata:
  hermes:
    tags: [bim, revit, metrados, arquitectura, peru, oe3, cantidades]
    related_skills: [auditar-modelo-bim, clasificar-partidas-oe3]
---

# Extraer Metrados de Arquitectura

## Overview

Extrae cantidades arquitectónicas de un modelo Revit activo usando las herramientas confirmadas del MCP `rvt-mcp` v0.5.0. Cada registro mantiene el ID del elemento BIM para trazabilidad. La extracción se realiza por paquetes de categoría para evitar respuestas inmanejables. Diferencia datos extraídos directamente del MCP de valores calculados por el agente.

**Prerequisito obligatorio:** La skill `auditar-modelo-bim` debe haberse ejecutado y el informe debe mostrar al menos una categoría con estado LISTO o REQUIERE AJUSTE.

## When to Use

- Tras completar la auditoría BIM con resultado no vacío.
- El usuario pide "extraer metrados", "obtener cantidades", "calcular áreas" desde Revit.
- Se necesitan datos base para clasificación OE.3 o generación de APU.

**No usar para:**
- Metrados de estructuras o instalaciones (disciplinas distintas).
- Modelos sin categorías arquitectónicas presentes.
- Si la auditoría reportó estado VACÍO o sin MCP.

## Inputs Obligatorios

1. Revit abierto con modelo activo y rvt-mcp conectado.
2. Informe de auditoría de `auditar-modelo-bim` (para saber qué categorías procesar).
3. Confirmación del usuario sobre el **paquete a extraer**: todas las categorías o una en particular.
4. ¿Existen tablas de planificación con campos de ID de elemento? (condiciona el método de extracción).

## Herramientas MCP Permitidas

| Herramienta | Uso en esta skill |
|---|---|
| `get_current_view_info` | Verificar conexión activa |
| `list_schedules` | Identificar tablas existentes por categoría |
| `get_schedule_data` | Extraer datos paginados de tablas |
| `activate_view` | Activar tabla antes de leerla si necesario |
| `send_code_to_revit` | Leer instancias, IDs y parámetros (SOLO LECTURA) |
| `list_materials` | Verificar materiales disponibles |

**NO se usan en esta skill:** `show_message`, `capture_view_image` (no aportan cantidades).

## Procedimiento Paso a Paso

### Paso 1 — Verificar conexión y estado de auditoría
```
get_current_view_info → confirmar respuesta válida
```
Si falla → DETENER con mensaje de error.

Revisar informe de auditoría previo. Identificar lista de categorías LISTO y REQUIERE AJUSTE.

**Criterio de completitud:** Lista de categorías a procesar definida.

---

### Paso 2 — Definir estrategia de extracción por categoría

Para cada categoría, determinar método:

| Categoría | Método primario | Método alternativo |
|---|---|---|
| Muros | `get_schedule_data` si existe tabla | `send_code_to_revit` lectura |
| Suelos (Pisos) | `get_schedule_data` si existe tabla | `send_code_to_revit` lectura |
| Techos | `get_schedule_data` si existe tabla | `send_code_to_revit` lectura |
| Puertas | `send_code_to_revit` lectura | — |
| Ventanas | `send_code_to_revit` lectura | — |
| Habitaciones | `send_code_to_revit` lectura | — |
| Barandas | `send_code_to_revit` lectura | — |
| Paneles MC | `send_code_to_revit` lectura | — |
| Cubiertas | `send_code_to_revit` lectura | — |
| Cielorrasos | `get_schedule_data` si existe tabla | `send_code_to_revit` lectura |

---

### Paso 3 — Extracción por paquete: Muros

**Vía tabla (si existe):**
```
list_schedules(categoryFilter="Muros")
activate_view(view_name="<nombre tabla>")
get_schedule_data(scheduleName="<nombre>", startRow=0, maxRows=500)
```
Paginar hasta `truncated=false`.

**Vía C# de solo lectura:**
```csharp
// SOLO LECTURA — extracción de muros
var walls = new FilteredElementCollector(doc)
    .OfCategory(BuiltInCategory.OST_Walls)
    .WhereElementIsNotElementType()
    .Cast<Wall>()
    .Select(w => new {
        Id = w.Id.Value,
        Familia = w.get_Parameter(BuiltInParameter.ELEM_FAMILY_PARAM)?.AsString() ?? "NO DISPONIBLE",
        Tipo = w.get_Parameter(BuiltInParameter.ELEM_TYPE_PARAM)?.AsString() ?? "NO DISPONIBLE",
        Nivel = w.LevelId != null ? (doc.GetElement(w.LevelId) as Level)?.Name ?? "NO DISPONIBLE" : "NO DISPONIBLE",
        Longitud = w.get_Parameter(BuiltInParameter.CURVE_ELEM_LENGTH)?.AsDouble() ?? -1,
        Area = w.get_Parameter(BuiltInParameter.HOST_AREA_COMPUTED)?.AsDouble() ?? -1,
        Volumen = w.get_Parameter(BuiltInParameter.HOST_VOLUME_COMPUTED)?.AsDouble() ?? -1
    });
// -1 indica parámetro no disponible
return string.Join("\n", walls.Select(w => 
    $"{w.Id}|{w.Familia}|{w.Tipo}|{w.Nivel}|{w.Longitud:F4}|{w.Area:F4}|{w.Volumen:F4}"));
```

> Las unidades en la API de Revit están en pies. El agente convierte a metros: 1 pie = 0.3048 m; áreas: × 0.0929; volúmenes: × 0.0283.

---

### Paso 4 — Extracción por paquete: Pisos y Techos

Código C# análogo al de muros, usando:
- `OST_Floors` para pisos
- `OST_Roofs` para techos/coberturas
- `OST_Ceilings` para cielorrasos

Parámetros clave:
- `HOST_AREA_COMPUTED` → Área
- `HOST_VOLUME_COMPUTED` → Volumen
- `LEVEL_PARAM` → Nivel

---

### Paso 5 — Extracción por paquete: Puertas y Ventanas

```csharp
// SOLO LECTURA — puertas
var doors = new FilteredElementCollector(doc)
    .OfCategory(BuiltInCategory.OST_Doors)
    .WhereElementIsNotElementType()
    .Cast<FamilyInstance>()
    .Select(d => new {
        Id = d.Id.Value,
        Familia = d.Symbol?.FamilyName ?? "NO DISPONIBLE",
        Tipo = d.Symbol?.Name ?? "NO DISPONIBLE",
        Nivel = (doc.GetElement(d.LevelId) as Level)?.Name ?? "NO DISPONIBLE",
        Ancho = d.Symbol?.get_Parameter(BuiltInParameter.DOOR_WIDTH)?.AsDouble() ?? -1,
        Alto = d.Symbol?.get_Parameter(BuiltInParameter.DOOR_HEIGHT)?.AsDouble() ?? -1
    });
```
Misma estructura para ventanas con `OST_Windows`, `WINDOW_WIDTH`, `WINDOW_HEIGHT`.

Cantidad = 1 unidad por instancia. No hay área geométrica directa: registrar Ancho × Alto como área referencial con fuente `[CALCULADO]`.

---

### Paso 6 — Extracción por paquete: Habitaciones

```csharp
// SOLO LECTURA — habitaciones
var rooms = new FilteredElementCollector(doc)
    .OfCategory(BuiltInCategory.OST_Rooms)
    .Cast<SpatialElement>()
    .Select(r => new {
        Id = r.Id.Value,
        Nombre = r.get_Parameter(BuiltInParameter.ROOM_NAME)?.AsString() ?? "NO DISPONIBLE",
        Numero = r.get_Parameter(BuiltInParameter.ROOM_NUMBER)?.AsString() ?? "NO DISPONIBLE",
        Nivel = r.LevelId != null ? (doc.GetElement(r.LevelId) as Level)?.Name ?? "NO DISPONIBLE" : "NO DISPONIBLE",
        Area = r.get_Parameter(BuiltInParameter.ROOM_AREA)?.AsDouble() ?? -1,
        Perimetro = r.get_Parameter(BuiltInParameter.ROOM_PERIMETER)?.AsDouble() ?? -1,
        Volumen = r.get_Parameter(BuiltInParameter.ROOM_VOLUME)?.AsDouble() ?? -1,
        Cerrada = (r.get_Parameter(BuiltInParameter.ROOM_VOLUME)?.AsDouble() ?? 0) > 0
    });
```

---

### Paso 7 — Procesar y estructurar registros

Para cada elemento extraído, crear un registro con estos campos exactos:

```json
{
  "id_elemento": 456789,
  "categoria": "Muros",
  "familia": "Muro básico",
  "tipo": "Ladrillo KK 15cm",
  "nivel": "Nivel 1",
  "fase": "Nueva Construcción",
  "ambiente": "NO DISPONIBLE",
  "material": "LADRILLO KK",
  "longitud_m": 4.25,
  "ancho_m": 0.15,
  "alto_m": 2.80,
  "area_bruta_m2": 11.90,
  "area_deducciones_m2": "NO DISPONIBLE",
  "area_neta_m2": 11.90,
  "volumen_m3": 1.785,
  "cantidad": 1,
  "unidad_original": "m²",
  "fuente_dato": "[MCP:get_schedule_data]",
  "estado_validacion": "EXTRAIDO",
  "observaciones": "Deducciones de vanos no disponibles por herramienta MCP"
}
```

**Fuentes de dato posibles:**
- `[MCP:get_schedule_data]` — dato directo de tabla Revit
- `[MCP:send_code_to_revit]` — dato de API C# solo lectura
- `[CALCULADO]` — derivado matemáticamente por el agente (ej. ancho × alto)
- `[NO DISPONIBLE]` — no fue posible obtener el dato

**Estados de validación:**
- `EXTRAIDO` — dato completo
- `PARCIAL` — faltan algunos campos
- `NO DISPONIBLE` — no se pudo extraer
- `REQUIERE_REVISION` — dato inconsistente

---

### Paso 8 — Detectar y registrar deducciones

Las deducciones (vanos en muros, huecos en pisos) **no están directamente disponibles** en las herramientas MCP actuales sin C# avanzado o tablas específicas.

Registrar:
- `area_deducciones_m2`: "NO DISPONIBLE" si no existe tabla con deducciones
- `observacion`: "Las deducciones de vanos deben verificarse manualmente o con tabla de planificación específica"

**NO inventar valores de deducción.**

---

### Paso 9 — Resumen de extracción

Generar tabla de resumen antes de pasar a la siguiente skill:

```markdown
## RESUMEN DE EXTRACCIÓN

| Categoría    | Elementos | Registros | Fuente       | Campos vacíos |
|-------------|-----------|-----------|--------------|---------------|
| Muros        | 73        | 73        | schedule+C#  | 0             |
| Pisos        | 22        | 22        | C#           | 5 (material)  |
| Puertas      | 23        | 23        | C#           | 0             |
| Ventanas     | 41        | 41        | C#           | 0             |
| Habitaciones | 51        | 51        | C#           | 3 (cerradas)  |

Advertencias:
- 5 pisos sin material asignado → usar "NO DISPONIBLE"
- 3 habitaciones no cerradas → área no confiable
- Deducciones de muros: NO DISPONIBLE en todas las entradas
```

## Esquema de Salida Estructurado

Lista de registros JSON + tabla de resumen como se describe en el Paso 7 y Paso 9. Los registros deben poder pasarse directamente a `clasificar-partidas-oe3`.

## Reglas de Validación

- **Nunca reemplazar un valor ausente por 0.** Usar `"NO DISPONIBLE"`.
- **Nunca modificar el modelo.** Todo C# enviado debe ser de solo lectura.
- Longitud, área y volumen en API de Revit están en pies. **Siempre convertir a metros SI antes de registrar.**
- Si una tabla tiene `truncated=true`, **siempre paginar** hasta `truncated=false`.
- Si `send_code_to_revit` falla por error de compilación, registrar el error, marcar elemento como `NO DISPONIBLE`, continuar con otros.
- **No duplicar elementos**: si un elemento aparece en tabla Y en C#, usar el dato de tabla como primario.
- Separar cantidad bruta, deducción y neta en campos distintos.

## Condiciones para Detenerse

1. MCP no conectado.
2. Categoría seleccionada con `count=0` en la auditoría (no procesar lo que no existe).
3. Todos los intentos de extracción de una categoría fallaron (schedule + C#) → marcar categoría como NO EXTRAÍBLE, continuar con siguiente.

## Limitaciones Conocidas

| Limitación | Declaración |
|---|---|
| Deducciones de vanos | No accesibles sin tabla específica o C# avanzado con geometría. NO DISPONIBLE por defecto |
| Material por capa | `get_schedule_data` retorna material global o primer material. Sin desglose por capa sin C# avanzado |
| Fase por elemento | Solo disponible vía C# `BuiltInParameter.PHASE_CREATED` |
| Ambiente (habitación) por elemento | Requiere C# de superposición geométrica — costoso. Usar si se solicita explícitamente |
| Altura libre de habitación | Requiere geometría 3D no disponible directamente |
| Barandas: longitud | Disponible vía C# `CURVE_ELEM_LENGTH` |
| Paneles MC: área | Disponible vía C# `HOST_AREA_COMPUTED` |

## Prueba Mínima de Funcionamiento

```
1. Abrir modelo con al menos 1 muro en Revit
2. Ejecutar la skill seleccionando categoría "Muros"
3. Resultado esperado:
   - Al menos 1 registro con id_elemento, categoria="Muros", area_bruta_m2 ≠ "NO DISPONIBLE"
   - fuente_dato incluye "[MCP:*]"
   - Sin errores de conexión
4. Si todos los registros tienen area_bruta_m2="NO DISPONIBLE" → FALLA DE PRUEBA
```

## Common Pitfalls

1. **No paginar `get_schedule_data`.** Si `truncated=true` y no se pagina, se pierden elementos → duplicación o metrado incompleto.
2. **No convertir unidades de pies a metros.** La API de Revit usa pies internamente. Factor: longitud ×0.3048; área ×0.0929; volumen ×0.0283.
3. **Usar 0 para valores ausentes.** Un muro con `area=0` es muy diferente a uno con `area="NO DISPONIBLE"`.
4. **Asumir que `ELEM_FAMILY_PARAM` siempre retorna string.** Puede ser `null` si el parámetro no aplica al tipo de elemento.
5. **No registrar la fuente del dato.** Sin fuente, el resultado no es auditable.

## Verification Checklist

- [ ] Auditoría previa completada con al menos 1 categoría procesable
- [ ] Todas las tablas existentes paginadas hasta `truncated=false`
- [ ] Conversión de unidades aplicada (pies → metros)
- [ ] Ningún campo ausente reemplazado por 0
- [ ] Fuente de dato registrada en cada registro
- [ ] Registros estructurados listos para `clasificar-partidas-oe3`
- [ ] Sin modificaciones al modelo
- [ ] Resumen de extracción generado con tabla de categorías
