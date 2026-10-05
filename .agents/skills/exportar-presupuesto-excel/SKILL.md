---
name: exportar-presupuesto-excel
description: >
  Use when the user wants to export the complete 5D BIM quantity survey results to an auditable, editable Excel file. Creates 10 worksheets with formulas (not hardcoded values), preserves BIM element IDs, normative sources and price sources. Detects empty cells, unit mismatches, duplicates and formula errors. Includes QA/QC control sheet. Final step of the 5D workflow after generar-apu.
version: 1.0.0
author: Antigravity
license: MIT
metadata:
  hermes:
    tags: [excel, presupuesto, metrados, apu, peru, oe3, exportacion, 5d]
    related_skills: [generar-apu, clasificar-partidas-oe3, extraer-metrados-arquitectura, auditar-modelo-bim]
---

# Exportar Presupuesto Excel

## Overview

Genera un archivo Excel `.xlsx` auditable con 10 hojas que vinculan metrados BIM, clasificación OE.3, APUs y presupuesto final. Usa fórmulas editables — no valores calculados manualmente. Mantiene trazabilidad de IDs de elementos, fuentes normativas y fuentes de precios. Detecta y reporta inconsistencias antes de entregar el archivo.

**Prerequisito:** Datos de las 4 skills anteriores completados (al menos parcialmente). El archivo puede exportarse con campos `PENDIENTE` — estos quedarán marcados en la hoja QA_QC.

## When to Use

- El usuario solicita "exportar a Excel", "generar el presupuesto", "armar el archivo final".
- Tras completar (total o parcialmente) las skills 1-4.
- Para entrega formal del presupuesto a cliente o revisión interna.

**No usar para:**
- Exportar sin haber completado al menos la extracción de metrados.
- Generar presupuestos de especialidades no cubiertas.

## Inputs Obligatorios

1. Registros de metrados (`extraer-metrados-arquitectura`).
2. Tabla de clasificación OE.3 (`clasificar-partidas-oe3`).
3. APUs generados (`generar-apu`) — pueden ser parciales.
4. Nombre del proyecto y versión del modelo.
5. Ruta de destino para el archivo `.xlsx`.

## Herramientas del Agente Permitidas

Esta skill usa código Python con la librería `openpyxl` para generar el Excel. No usa herramientas MCP de Revit.

Dependencia: `openpyxl` (estándar, sin instalación adicional si Python 3.x disponible).

## Estructura del Archivo Excel

### Hoja 1: Configuración

| Campo | Valor |
|---|---|
| Nombre del proyecto | [dato] |
| Versión del modelo | [dato] |
| Fecha de generación | [fecha ISO] |
| Versión MCP Revit | 0.5.0 |
| Ciudad de referencia | [dato] |
| Fecha de precios | [dato] |
| Moneda | [dato] |
| IGV (%) | [dato] |
| Elaborado por | [usuario] |
| Nota de responsabilidad | [ver advertencia al pie] |

Todas las celdas de valor son editables. El resto del libro referencia estas celdas mediante fórmulas con nombre definido (Named Ranges).

---

### Hoja 2: Auditoría BIM

Tabla exportada del informe de `auditar-modelo-bim`:

| Columna | Descripción |
|---|---|
| Categoría | Nombre de categoría Revit |
| Instancias | Conteo de elementos |
| Estado | LISTO / REQUIERE AJUSTE / NO MODELADO / NO VERIFICABLE |
| Incidencias | Lista de IDs con problema |
| Observaciones | Texto libre |

Incluye resumen del semáforo global al pie de la hoja.

---

### Hoja 3: Mapeo OE.3

Tabla de clasificación completa:

| Col | Campo |
|---|---|
| A | Código OE.3 |
| B | Descripción partida (del RNM) |
| C | Unidad normativa |
| D | Forma de medición |
| E | Categoría BIM |
| F | Familia BIM |
| G | Tipo BIM |
| H | Material BIM |
| I | Regla de correspondencia |
| J | IDs de elementos (separados por coma) |
| K | Cantidad bruta |
| L | Deducciones |
| M | Cantidad neta (fórmula: =K-L, o "REQUIERE_VER." si L vacío) |
| N | Fuente normativa |
| O | Página/sección RNM |
| P | Nivel de confianza |
| Q | Estado |
| R | Observaciones |

---

### Hoja 4: Metrados

Registros individuales de elementos BIM:

| Col | Campo |
|---|---|
| A | ID Elemento |
| B | Categoría |
| C | Familia |
| D | Tipo |
| E | Nivel |
| F | Fase |
| G | Ambiente |
| H | Material |
| I | Longitud (m) |
| J | Anchura (m) |
| K | Altura (m) |
| L | Área bruta (m²) |
| M | Deducciones (m²) |
| N | Área neta (m²) — fórmula: =L-M |
| O | Volumen (m³) |
| P | Cantidad |
| Q | Unidad original |
| R | Fuente del dato |
| S | Estado de validación |
| T | Observaciones |

Subtotales por categoría con `SUBTOTAL(9,rango)` para que funcionen con filtros.

---

### Hoja 5: Precios y Fuentes

Registro de todas las fuentes de precios usadas en los APUs:

| Col | Campo |
|---|---|
| A | Código partida |
| B | Insumo |
| C | Unidad |
| D | Precio 1 |
| E | Fuente 1 |
| F | URL 1 |
| G | Fecha 1 |
| H | Precio 2 |
| I | Fuente 2 |
| J | URL 2 |
| K | Fecha 2 |
| L | Precio 3 |
| M | Fuente 3 |
| N | Mediana precios (fórmula: =MEDIANA(D,H,L)) |
| O | Precio adoptado |
| P | Estado verificación |

---

### Hoja 6: APU

Un bloque por partida. Cada bloque:

```
[A1] Código: OE.3.1     [B1] Descripción: Muros de albañilería
[A2] Unidad: m²         [B2] Ciudad: Lima    [C2] Fecha: 2025-07
[A3] Moneda: PEN        [B3] IGV: =Config!IGV_pct

COMPONENTES
[A5] Tipo | [B5] Descripción | [C5] Unidad | [D5] Aporte | [E5] P.Unit | [F5] Parcial
[A6] MAT  | Ladrillo KK     | und         | 72          | ='Precios'!O2 | =D6*E6
...
[A_N] MO  | Operario        | hh          | ...
...
[A_M] EQ  | Herramientas    | %MO         | 3%
...
[Fórmula CD] =SUMA(parciales_mat) + SUMA(parciales_mo) + SUMA(parciales_eq)
[Fórmula IGV] =CD * (1 + Config!IGV_pct/100)
[Total] =CD + IGV
[Estado] [VERIFICADO / INCOMPLETO]
```

Todos los precios referencian la hoja **Precios y Fuentes**. No se escriben valores hardcodeados en APU.

---

### Hoja 7: Presupuesto

Resumen ejecutivo del presupuesto:

| Col | Campo |
|---|---|
| A | Código OE.3 |
| B | Descripción |
| C | Unidad |
| D | Cantidad neta (referencia a hoja Mapeo OE.3) |
| E | Precio unitario (referencia a hoja APU) |
| F | Costo parcial (fórmula: =D*E) |

Subtotales por sección OE.3 (OE.3.1, OE.3.2, etc.) con `SUBTOTAL`.

Al pie:

| Concepto | Valor |
|---|---|
| Costo directo total | =SUMA(F:F) |
| Gastos generales (%) | [celda editable] |
| Gastos generales (S/.) | =CD*GG_pct |
| Utilidad (%) | [celda editable] |
| Utilidad (S/.) | =CD*UT_pct |
| Sub-total | =CD+GG+UT |
| IGV | =Subtotal * Config!IGV_pct |
| Contingencia (%) | [celda editable] |
| PRESUPUESTO TOTAL | =Subtotal+IGV+Cont |

> Gastos generales, utilidad y contingencia son **celdas editables sin fórmula predefinida** — el usuario ingresa los porcentajes según el proyecto.

---

### Hoja 8: Sensibilidad

Tabla de análisis de sensibilidad con variación de precio de materiales y rendimientos:

| Variable | -20% | -10% | Base | +10% | +20% |
|---|---|---|---|---|---|
| Precio ladrillo | =... | =... | =Base | =... | =... |
| Jornal operario | =... | =... | =Base | =... | =... |
| Presupuesto total | =... | =... | =Base | =... | =... |

Todas las celdas son fórmulas que referencian la hoja Config y APU.

---

### Hoja 9: Control QA_QC

Verificaciones automáticas mediante fórmulas:

| N° | Verificación | Resultado | Detalle |
|---|---|---|---|
| 1 | Celdas vacías en Metrados (col. críticas) | =CONTAR.SI.CONJUNTO(...) | [lista de IDs] |
| 2 | Unidades incompatibles OE.3 vs APU | [fórmula de comparación] | [lista] |
| 3 | IDs duplicados en Metrados | =CONTAR.SI(A:A,A2)>1 | [lista] |
| 4 | APUs marcados INCOMPLETO | =CONTAR.SI(APU!Estado,"INCOMPLETO") | [N] |
| 5 | Partidas sin cantidad neta | =CONTAR.SI(Mapeo!M:M,"REQUIERE*") | [N] |
| 6 | Precios sin fuente | =CONTAR.SI(Precios!P:P,"[PENDIENTE]") | [N] |
| 7 | Fórmulas con error (#¿NOMBRE?, #¡REF!) | [función de auditoría] | [lista] |
| 8 | Cantidad neta negativa | =CONTAR.SI(Mapeo!M:M,"<0") | [N] |

Al pie: **RESUMEN DE ADVERTENCIAS CRÍTICAS** con total de incidencias por tipo.

---

### Hoja 10: Registro de Ejecución

Log de las acciones realizadas por el sistema:

| Timestamp | Skill | Acción | Resultado | Notas |
|---|---|---|---|---|
| 2025-07-06T15:00 | auditar-modelo-bim | Auditoría completada | LISTO parcial | 5 incidencias |
| 2025-07-06T15:30 | extraer-metrados | Extracción muros | 73 registros | Deducciones N/D |
| 2025-07-06T16:00 | clasificar-oe3 | Clasificación | 8 partidas | 2 REQUIERE_CONF |
| 2025-07-06T16:30 | generar-apu | APU | 3 VERF / 5 INCOMP | Faltan jornales |
| 2025-07-06T17:00 | exportar-excel | Exportación | EXITOSO | ver QA_QC |

## Procedimiento Paso a Paso

### Paso 1 — Validar datos de entrada
Verificar que existan datos mínimos:
- Al menos registros de metrados (Hoja 4).
- Sin datos de APU → hojas APU y Presupuesto generadas con campos `[PENDIENTE]`.

**Criterio de completitud:** Datos cargados en memoria del agente.

### Paso 2 — Generar el archivo con openpyxl

```python
# Script de generación — esquema
import openpyxl
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter
from datetime import datetime

wb = openpyxl.Workbook()

# Hoja 1: Configuración
ws_config = wb.active
ws_config.title = "Configuracion"
# ... poblar con datos de configuración

# Hojas 2-10: crear y poblar con datos
# Todas las celdas de cálculo usan fórmulas Excel, no valores Python calculados

wb.save("Presupuesto_5D_[proyecto]_[fecha].xlsx")
```

> El script completo se encuentra en `scripts/generar_excel.py` (ver referencia).

### Paso 3 — Ejecutar verificaciones QA_QC

Antes de entregar el archivo:
1. Revisar hoja QA_QC para verificaciones fallidas.
2. Si hay errores de fórmula o IDs duplicados → corregir antes de entregar.
3. Si hay APUs INCOMPLETO → reportar al usuario (no es error, es información).

### Paso 4 — Entregar el archivo y resumen

Proporcionar al usuario:
- Ruta del archivo generado.
- Resumen de advertencias críticas de QA_QC.
- Lista de acciones pendientes del usuario (celdas `[PENDIENTE]`).

## Reglas de Validación

- **Usar fórmulas, no valores calculados.** Las celdas de resultado deben contener fórmulas referenciando otras hojas.
- **No calcular GG, utilidad ni contingencia** si el usuario no ha definido los porcentajes.
- **Preservar IDs** de elementos en todas las hojas donde corresponda.
- **Detectar duplicados** mediante fórmulas en QA_QC, no manualmente.
- **Fecha de generación automática** en hoja Configuración.
- **No borrar campos `[PENDIENTE]`** — son marcadores para que el usuario complete.

## Condiciones para Detenerse

1. Sin registros de metrados → no generar archivo. El Excel vacío no tiene valor.
2. Error de escritura en la ruta de destino → reportar y solicitar ruta alternativa.

## Limitaciones Conocidas

| Limitación | Declaración |
|---|---|
| Fórmulas de auditoría de errores | `CELDA("tipo"...)` y funciones de auditoría avanzada pueden no funcionar en todas las versiones de Excel/LibreOffice |
| Hoja de sensibilidad | Solo cubre variables que el usuario haya definido en APU; no genera escenarios automáticos |
| Macros | El archivo no incluye macros VBA por seguridad; todo es fórmulas estándar |
| Idioma Excel | Las fórmulas se generan en español; si el usuario tiene Excel en otro idioma, puede necesitar adaptar nombres de funciones |

## Ejemplo de Uso

**Input:** 73 registros de muros + 3 APUs verificados + clasificación OE.3 de 5 partidas

**Output:** `Presupuesto_5D_Casa2Pisos_2025-07.xlsx` con:
- Hoja Metrados: 73 filas con IDs, áreas en m², fuentes
- Hoja APU: 3 bloques con fórmulas vinculadas a Precios
- Hoja Presupuesto: 5 partidas + subtotales + celdas de GG, utilidad, IGV editables
- Hoja QA_QC: 2 advertencias (5 pisos sin material, 2 APUs INCOMPLETO)

## Prueba Mínima de Funcionamiento

```
1. Proporcionar 3 registros de metrado + 1 APU verificado + 1 partida OE.3
2. Ejecutar la skill
3. Resultado esperado:
   - Archivo .xlsx creado en la ruta especificada
   - Hoja Metrados con 3 filas y columna ID_Elemento poblada
   - Hoja Presupuesto con fórmula =D2*E2 en columna F (no valor hardcodeado)
   - Hoja QA_QC generada con al menos 8 verificaciones
4. Si alguna celda de cálculo tiene valor hardcodeado (no fórmula) → FALLA DE PRUEBA
```

## Common Pitfalls

1. **Hardcodear resultados de Python en Excel.** Todas las sumas, productos y totales deben ser fórmulas Excel, no el resultado calculado por Python.
2. **Omitir la hoja QA_QC.** Sin control de calidad el archivo no es auditable.
3. **No vincular APU con Precios y Fuentes.** Si los precios del APU están hardcodeados, se pierden al actualizar cotizaciones.
4. **Ignorar APUs INCOMPLETO.** Deben marcarse claramente en la hoja Presupuesto para que el usuario no los use sin revisar.

## Verification Checklist

- [ ] Las 10 hojas creadas con nombres correctos
- [ ] Hoja Configuración con Named Ranges para IGV, ciudad, fecha
- [ ] Hoja Metrados con columna ID_Elemento y fuentes del dato
- [ ] Hoja APU con fórmulas referenciando Precios y Fuentes
- [ ] Hoja Presupuesto con fórmulas =cantidad*precio en columna costo parcial
- [ ] Gastos generales, utilidad y contingencia como celdas editables (sin fórmula predefinida)
- [ ] Hoja QA_QC con las 8 verificaciones implementadas
- [ ] Hoja Registro de Ejecución con log de las 5 skills
- [ ] Sin fórmulas con errores (#¡REF!, #¿NOMBRE?)
- [ ] Resumen de advertencias críticas entregado al usuario
