---
name: clasificar-partidas-oe3
description: >
  Use when the user wants to map extracted BIM elements to OE.3 Arquitectura partidas from the Reglamento Nacional de Metrados del Peru (RNM). Requires the user to provide the RNM document. Classifies by material, type, function and measurement method — not just by family name. Marks ambiguous results as REQUIERE_CONFIRMACION. Never invents codes, sub-partidas, quantities or normative references. Consults user-provided document and records the page or section used.
version: 1.0.0
author: Antigravity
license: MIT
metadata:
  hermes:
    tags: [metrados, oe3, rnm, arquitectura, partidas, peru, clasificacion]
    related_skills: [extraer-metrados-arquitectura, generar-apu, exportar-presupuesto-excel]
---

# Clasificar Partidas OE.3

## Overview

Relaciona los elementos extraídos por `extraer-metrados-arquitectura` con las partidas OE.3 del Reglamento Nacional de Metrados del Perú (Resolución Ministerial N° 073-2010-VIVIENDA y actualizaciones vigentes). El documento normativo DEBE ser proporcionado por el usuario — el agente no lo tiene incorporado ni lo inventa.

La clasificación considera: material, tipo constructivo, función del elemento, y forma de medición normativa. No clasifica únicamente por nombre de familia Revit.

## When to Use

- Tras completar `extraer-metrados-arquitectura` con registros válidos.
- El usuario necesita codificar elementos BIM en partidas OE.3.
- Antes de generar APU o presupuesto formal.

**No usar para:**
- Clasificar partidas de otras especialidades (OE.1 Estructuras, OE.2 Instalaciones).
- Si el usuario no puede proporcionar el documento RNM.
- Sin haber ejecutado primero `extraer-metrados-arquitectura`.

## Inputs Obligatorios

1. **Registros de metrados** generados por `extraer-metrados-arquitectura`.
2. **Documento RNM OE.3** proporcionado por el usuario (PDF o texto). El agente debe consultarlo, no inventar su contenido.
3. Confirmación: ¿Se aplica a una fase específica del proyecto?

> ⚠️ Si el usuario no entrega el documento RNM, detener y solicitar: "Para clasificar partidas OE.3 necesito el documento Reglamento Nacional de Metrados (RNM) Perú. Por favor proporciónelo."

## Herramientas MCP Permitidas

Ninguna herramienta MCP de Revit se usa en esta skill. La clasificación es un proceso documental-analítico sobre los datos ya extraídos.

Herramientas del agente permitidas:
- Lectura del documento RNM proporcionado por el usuario.
- Lectura de los registros de metrados.
- Generación de tabla de clasificación.

## Cobertura OE.3 (Arquitectura)

Las siguientes partidas se contemplan en la clasificación. El agente consulta el documento RNM para obtener descripción exacta, unidad y forma de medición:

| Código | Sección | Partida general |
|--------|---------|----------------|
| OE.3.1 | — | Muros y tabiques de albañilería |
| OE.3.2 | — | Revoques, enlucidos y molduras |
| OE.3.3 | — | Cielorrasos |
| OE.3.4 | — | Pisos y pavimentos |
| OE.3.5 | — | Zócalos y contrazócalos |
| OE.3.6 | — | Coberturas |
| OE.3.7 | — | Carpintería de madera |
| OE.3.8 | — | Carpintería metálica y herrería |
| OE.3.9 | — | Cerrajería |
| OE.3.10 | — | Vidrios, cristales y similares |
| OE.3.11 | — | Pintura |
| OE.3.12 | — | Varios, limpieza y jardinería |

> El agente NUNCA inventa subpartidas, descripciones ni unidades. Solo usa las que figuran en el documento proporcionado.

## Procedimiento Paso a Paso

### Paso 1 — Verificar inputs
- ¿Existen registros de metrados con campos: `categoria`, `tipo`, `material`, `area_neta_m2`?
- ¿El usuario ha proporcionado el documento RNM OE.3?

Si falta alguno → DETENER y solicitar el faltante.

**Criterio de completitud:** Registros cargados + sección OE.3 del RNM disponible para lectura.

---

### Paso 2 — Construir matriz de correspondencia

Para cada registro de metrado:

1. **Identificar categoría Revit** (`categoria` del registro).
2. **Leer el material** (`material` del registro).
3. **Leer el tipo constructivo** (`tipo` del registro).
4. **Consultar el documento RNM**: buscar la partida cuya descripción y forma de medición coincida con el elemento.
5. **Verificar la unidad normativa** del RNM vs la unidad del registro BIM.
6. **Asignar código OE.3.x** — solo si la correspondencia es clara y documentada.

**Lógica de clasificación (no inventar — aplicar desde el RNM):**

- Un muro de ladrillo KK tipo soga → consultar RNM OE.3.1 para muros de albañilería.
- El acabado de ese mismo muro (tarrajeo) → consultar RNM OE.3.2.
- Los datos de material del BIM (ej. "CHAPISCO", "MASSA ÚNICA") deben relacionarse con la descripción normativa del RNM.
- Si el material BIM no permite determinar la partida con certeza → `REQUIERE_CONFIRMACION`.

---

### Paso 3 — Clasificar partidas que no provienen de geometría directa

Algunas partidas OE.3 **no pueden obtenerse directamente de la geometría BIM**. Identificarlas y documentarlas:

| Partida | Razón | Acción |
|---------|-------|--------|
| OE.3.9 Cerrajería | No modelada como elemento independiente en Revit estándar | Solicitar planilla al usuario |
| OE.3.11 Pintura | Depende de área de muro, no siempre distinguible del acabado | Derivar de OE.3.1 + OE.3.2 con confirmación |
| OE.3.5 Zócalos | Raramente modelados en Revit como familia independiente | Solicitar o marcar NO MODELADO |
| OE.3.12 Limpieza | Por partida general, no geométrica | Solicitar rendimiento y área |

---

### Paso 4 — Calcular cantidades netas por partida

Para cada partida asignada:

```
cantidad_bruta = suma de area_bruta_m2 (o longitud/volumen según unidad normativa)
deducciones = suma de area_deducciones_m2 (si disponible, si no: "NO DISPONIBLE")
cantidad_neta = cantidad_bruta - deducciones (si deducciones disponibles)
              = "REQUIERE_VERIFICACION" (si deducciones = "NO DISPONIBLE")
```

**No sumar elementos de distintos niveles o fases sin indicarlo.**

---

### Paso 5 — Generar tabla de clasificación

Cada registro de clasificación debe contener:

```json
{
  "codigo_oe3": "OE.3.1",
  "descripcion_partida": "[texto exacto del RNM, página X]",
  "unidad_normativa": "m²",
  "forma_medicion": "[texto exacto del RNM]",
  "categoria_bim": "Muros",
  "familia_bim": "Muro básico",
  "tipo_bim": "Ladrillo KK 15cm",
  "material_bim": "LADRILLO KK",
  "regla_correspondencia": "Material LADRILLO KK → OE.3.1 muros de albañilería (RNM p.XX)",
  "ids_elementos": [456789, 456790, 456795],
  "cantidad_bruta": 145.30,
  "deducciones": "NO DISPONIBLE",
  "cantidad_neta": "REQUIERE_VERIFICACION",
  "fuente_normativa": "RNM OE.3 Arquitectura, p.XX, sección X.X",
  "nivel_confianza": "ALTO",
  "estado": "CLASIFICADO",
  "observaciones": "Deducciones de vanos no extraídas del BIM"
}
```

**Niveles de confianza:**
- `ALTO` — Material, tipo y forma de medición coinciden sin ambigüedad con el RNM.
- `MEDIO` — Coincidencia probable pero con supuesto (ej. material no especificado en BIM).
- `BAJO` — Clasificación inferida; requiere confirmación de especialista.
- `REQUIERE_CONFIRMACION` — No puede asignarse sin información adicional.

**Estados:**
- `CLASIFICADO` — Partida asignada con evidencia.
- `REQUIERE_CONFIRMACION` — Ambigüedad o falta de datos.
- `SIN_PARTIDA` — No existe partida OE.3 equivalente para este elemento.
- `NO_DISPONIBLE_EN_BIM` — Partida necesaria pero elemento no modelado.

---

### Paso 6 — Resumen de clasificación

```markdown
## RESUMEN DE CLASIFICACIÓN OE.3

| Código | Descripción | Unidad | Cantidad Neta | Confianza | Estado |
|--------|-------------|--------|--------------|-----------|--------|
| OE.3.1 | Muros de albañilería | m² | REQUIERE_VER. | ALTO | CLASIFICADO |
| OE.3.2 | Tarrajeo interior | m² | REQUIERE_VER. | MEDIO | CLASIFICADO |
| OE.3.4 | Pisos de cerámica | m² | 25.21 | ALTO | CLASIFICADO |
| OE.3.7 | Puertas de madera | und | 23 | ALTO | CLASIFICADO |
| OE.3.9 | Cerrajería | pto | NO DISPONIBLE | — | NO_DISPONIBLE_EN_BIM |
| OE.3.11 | Pintura | m² | REQUIERE_VER. | BAJO | REQUIERE_CONFIRMACION |

Elementos sin clasificar: [lista]
Partidas sin equivalente BIM: [lista]

## ELEMENTOS QUE REQUIEREN CONFIRMACIÓN HUMANA
[lista detallada con razón]
```

---

### Paso 7 — Solicitar revisión humana

Antes de cerrar, presentar al usuario:
1. Tabla completa de clasificación.
2. Lista de elementos `REQUIERE_CONFIRMACION` con propuesta del agente.
3. Partidas `NO_DISPONIBLE_EN_BIM` que deben completarse manualmente.
4. Permitir que el usuario corrija o confirme la clasificación.

**No avanzar a `generar-apu` sin que el usuario haya revisado al menos las partidas de nivel de confianza BAJO y REQUIERE_CONFIRMACION.**

## Reglas de Validación

- **No clasificar solo por nombre de familia Revit.** El nombre "Puerta de madera" no garantiza que sea OE.3.7 sin verificar material y función.
- **No inventar códigos.** Solo usar códigos que existan en el documento RNM proporcionado.
- **No inventar subpartidas.** Si el RNM no tiene la subpartida, no crearla.
- **Registrar siempre la página o sección** del RNM consultada.
- **No mezclar cantidades de distintas fases** sin indicarlo explícitamente.
- Materiales con nombres en otros idiomas (ej. "Bois", "Liquid") deben evaluarse antes de clasificar; si no corresponden a materiales constructivos peruanos, marcar como `REQUIERE_CONFIRMACION`.

## Condiciones para Detenerse

1. El usuario no proporciona el documento RNM → DETENER.
2. Registros de metrados vacíos o corruptos → DETENER.
3. Ningún elemento puede clasificarse (todos `REQUIERE_CONFIRMACION`) → emitir advertencia y solicitar orientación al usuario antes de continuar.

## Limitaciones Conocidas

| Limitación | Declaración |
|---|---|
| Zócalos y contrazócalos | Raramente modelados en Revit; casi siempre NO_DISPONIBLE_EN_BIM |
| Cerrajería | No es elemento geométrico BIM estándar; requiere planilla manual |
| Pintura por color | BIM no distingue colores de pintura; clasificación requiere confirmación |
| Partidas de limpieza | No extraíbles de geometría; requieren datos de área y rendimiento |
| Elementos de acabado en familias compuestas | Material puede estar en capas no accesibles sin C# avanzado |

## Ejemplo de Uso

**Input:**
- 73 registros de muros con material "CHAPISCO", "MASSA ÚNICA", "TINTA ACRÍLICA"
- RNM OE.3 p.45: "OE.3.2 Revoques — medición m² de área neta"

**Resultado:**
```json
{
  "codigo_oe3": "OE.3.2",
  "descripcion_partida": "Tarrajeo con mortero 1:5 — RNM p.45",
  "unidad_normativa": "m²",
  "material_bim": "MASSA ÚNICA",
  "regla_correspondencia": "MASSA ÚNICA = mortero de revoque (RNM OE.3.2, p.45)",
  "cantidad_bruta": 1494.18,
  "nivel_confianza": "MEDIO",
  "observaciones": "Material inferido por nombre en portugués del BIM; requiere confirmación"
}
```

## Prueba Mínima de Funcionamiento

```
1. Tomar 3 registros de metrado (1 muro, 1 piso, 1 puerta)
2. Proporcionar sección OE.3 del RNM con al menos 3 partidas
3. Resultado esperado:
   - 3 registros de clasificación con campo codigo_oe3 completado
   - fuente_normativa incluye página o sección del RNM
   - Ningún código inventado
4. Si algún código no existe en el RNM proporcionado → FALLA DE PRUEBA
```

## Common Pitfalls

1. **Clasificar por nombre de familia.** "Ventana de vidrio" puede ser OE.3.10, OE.3.8 o OE.3.7 dependiendo del tipo.
2. **Inventar páginas del RNM.** Registrar solo páginas reales del documento proporcionado.
3. **Sumar cantidades sin verificar deducciones.** Si deducciones = "NO DISPONIBLE", la cantidad neta es "REQUIERE_VERIFICACION".
4. **No solicitar revisión humana.** Sin aprobación del usuario, los resultados ambiguos no deben avanzar al APU.

## Verification Checklist

- [ ] Documento RNM OE.3 proporcionado por usuario y accesible
- [ ] Todos los registros de metrado procesados
- [ ] Ningún código OE.3 inventado o inferido sin evidencia normativa
- [ ] Página o sección del RNM registrada en cada clasificación
- [ ] Elementos `REQUIERE_CONFIRMACION` listados y presentados al usuario
- [ ] Partidas sin equivalente BIM identificadas como `NO_DISPONIBLE_EN_BIM`
- [ ] Usuario revisó y confirmó o corrigió clasificaciones ambiguas
- [ ] Tabla de resumen generada con cantidad bruta, deducciones y neta
