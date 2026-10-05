---
name: generar-apu
description: >
  Use when the user wants to generate Analisis de Precios Unitarios (APU) for validated OE.3 partidas in Peru. Uses only user-provided sources: CAPECO methodology, price lists, verified web quotes, and explicitly provided jornal/rendimiento data. Never invents prices, yields, crews or uses historical CAPECO prices as current values. Marks APU as INCOMPLETO if any critical field lacks evidence. Must follow clasificar-partidas-oe3.
version: 1.0.0
author: Antigravity
license: MIT
metadata:
  hermes:
    tags: [apu, precios-unitarios, presupuesto, peru, capeco, arquitectura, costos]
    related_skills: [clasificar-partidas-oe3, exportar-presupuesto-excel]
---

# Generar APU (Análisis de Precios Unitarios)

## Overview

Construye APUs para las partidas OE.3 previamente validadas. Usa exclusivamente fuentes proporcionadas por el usuario o fuentes web verificables y citables. No usa precios históricos de CAPECO como referencia actual. No inventa rendimientos, cuadrillas ni precios. Cada APU generado tiene estado de verificación y lista de supuestos explícitos.

**Prerequisito:** Tabla de clasificación OE.3 con partidas en estado `CLASIFICADO` o `REQUIERE_CONFIRMACION` revisadas por el usuario.

## When to Use

- Tras completar y revisar `clasificar-partidas-oe3`.
- El usuario solicita "construir el APU", "calcular precios unitarios", "armar el presupuesto por partida".
- Se dispone de al menos una fuente de precios: lista de proveedores, cotización, URL verificable, o jornales vigentes publicados.

**No usar para:**
- Partidas sin clasificación OE.3 validada.
- Si el usuario no puede proporcionar ninguna fuente de precios (en ese caso, los APUs quedarán como INCOMPLETO y se debe advertir).
- Presupuestos de especialidades distintas a arquitectura.

## Inputs Obligatorios

Antes de construir cualquier APU, confirmar con el usuario:

1. **Ciudad** donde se ejecuta la obra (afecta jornales, fletes, disponibilidad).
2. **Fecha de referencia** de precios (mes y año).
3. **Moneda** (PEN soles, USD dólares).
4. **IGV** aplicable (18% estándar en Perú, o exonerado según régimen).
5. **Flete** estimado (S/./m³, S/./und, o "incluido en precio proveedor").
6. **Fuentes disponibles:** ¿Tiene el usuario alguna de estas?
   - [ ] CAPECO vigente (solo metodología, no precios históricos como actuales)
   - [ ] Lista de precios de materiales de proveedor
   - [ ] Cotizaciones adjuntas
   - [ ] URLs de páginas de precios
   - [ ] Jornales oficiales (CAPECO, MTPE, contrato colectivo vigente)
   - [ ] Rendimientos propios de experiencia

> ⚠️ Si el usuario no proporciona ninguna fuente, generar APU con todos los campos de precio como `[PENDIENTE]` y estado `INCOMPLETO`. No inventar valores.

## Herramientas MCP Permitidas

No se usan herramientas MCP de Revit en esta skill. Es un proceso de análisis económico.

Herramientas del agente permitidas:
- Búsqueda web para cotizaciones públicas (solo si el usuario autoriza y la URL es verificable).
- Lectura de documentos proporcionados por el usuario (CAPECO, listas de precios).

## Procedimiento Paso a Paso

### Paso 1 — Recopilar configuración APU

Solicitar y registrar:

```
ciudad: ___________
fecha_referencia: ___________
moneda: PEN / USD
igv_pct: 18% / exonerado / otro
flete_incluido: SI / NO / PARCIAL
fuentes_disponibles: [lista]
```

No continuar sin ciudad y fecha de referencia.

**Criterio de completitud:** Todos los campos de configuración respondidos.

---

### Paso 2 — Seleccionar partidas a desarrollar

Presentar al usuario la lista de partidas clasificadas y preguntar:
- ¿Desarrollar todas las partidas?
- ¿Solo las de nivel de confianza ALTO?
- ¿Una partida específica?

Priorizar partidas con mayor impacto económico (mayor cantidad neta).

---

### Paso 3 — Estructura base del APU

Para cada partida seleccionada, crear la estructura:

```json
{
  "codigo_partida": "OE.3.1",
  "descripcion": "[texto del RNM]",
  "unidad": "m²",
  "ciudad": "Lima",
  "fecha_referencia": "2025-07",
  "moneda": "PEN",
  "igv_pct": 18,
  "componentes": {
    "materiales": [],
    "mano_de_obra": [],
    "equipos_herramientas": []
  },
  "costo_directo_unitario": null,
  "supuestos": [],
  "estado_verificacion": "INCOMPLETO",
  "advertencias": []
}
```

---

### Paso 4 — Completar componentes: Materiales

Para cada material necesario en la partida:

```json
{
  "descripcion": "Ladrillo KK 18 huecos",
  "unidad_comercial": "und",
  "aporte_unitario": 72,
  "rendimiento_comercial": "millar",
  "precio_unitario": null,
  "merma_pct": 5,
  "precio_con_merma": null,
  "parcial": null,
  "fuente": "[PENDIENTE — solicitar cotización]",
  "url_fuente": null,
  "fecha_cotizacion": null,
  "supuesto": "Ladrillo KK estándar 18 huecos 9x13x24cm"
}
```

**Reglas para materiales:**
- `precio_unitario`: solo completar si hay fuente verificable.
- Si hay ≥ 2 fuentes equivalentes para el mismo material, usar la **mediana** de precios.
- Normalizar unidades comerciales (millar → und; saco → kg; galón → litro) con el factor de conversión explícito.
- Registrar merma si la fuente o CAPECO metodológico la especifica; si no, dejar vacío.
- No mezclar materiales técnicamente distintos (ej. ladrillo KK ≠ ladrillo pandereta).

---

### Paso 5 — Completar componentes: Mano de Obra

```json
{
  "categoria": "Operario",
  "cuadrilla": 1.0,
  "jornal_diario": null,
  "rendimiento_diario": null,
  "aporte_unitario": null,
  "costo_unitario": null,
  "fuente_jornal": "[PENDIENTE — proporcionar jornal vigente CAPECO/MTPE]",
  "fuente_rendimiento": "[PENDIENTE — proporcionar rendimiento de experiencia o CAPECO]"
}
```

**Reglas para mano de obra:**
- `jornal_diario`: SOLO usar jornales proporcionados explícitamente por el usuario o publicados por CAPECO/MTPE con fecha vigente.
- `rendimiento_diario`: SOLO usar si el usuario lo proporciona o si está en el documento CAPECO del usuario.
- No usar rendimientos de CAPECO histórico como si fueran actuales.
- Cuadrilla típica en Perú: Capataz + Operarios + Peones — confirmada con el usuario o CAPECO del usuario.
- Si faltan jornal O rendimiento → la mano de obra queda `[PENDIENTE]`.

---

### Paso 6 — Completar componentes: Equipos y Herramientas

```json
{
  "descripcion": "Herramientas manuales",
  "unidad": "global",
  "aporte_porcentaje_mo": 3,
  "costo_unitario": null,
  "fuente": "CAPECO metodológico, proporcionado por usuario"
}
```

**Reglas para equipos:**
- El % de herramientas sobre mano de obra solo si CAPECO metodológico del usuario lo indica.
- Equipos con alquiler: usar precio de mercado local, con fecha y fuente.
- No inventar tasas de alquiler.

---

### Paso 7 — Calcular costo directo unitario

```
costo_directo_unitario = suma(parcial_materiales) + suma(costo_unitario_mo) + suma(costo_unitario_equipos)
```

Solo calcular si **todos los componentes críticos tienen precio**. Si alguno está `[PENDIENTE]`:
- `costo_directo_unitario = null`
- `estado_verificacion = "INCOMPLETO"`

Componentes críticos: materiales estructurales + mano de obra categoría Operario.
Componentes no críticos: herramientas manuales (puede estimarse como % si el usuario lo autoriza).

---

### Paso 8 — Calcular precio unitario total

```
precio_con_igv = costo_directo_unitario × (1 + igv_pct/100)
precio_con_flete = costo_directo_unitario + flete_unitario (si aplica)
```

Solo calcular si `costo_directo_unitario ≠ null`.

---

### Paso 9 — Registrar supuestos y advertencias

Todo supuesto debe quedar registrado explícitamente:

```
supuestos:
- "Rendimiento de asentado de ladrillo asumido de CAPECO 2022 proporcionado por usuario (p.45)"
- "Precio de cemento Portland tipo I tomado de web ferrocalidad.com.pe el 2025-07-01"
- "IGV 18% aplicado al costo directo"
advertencias:
- "Precio de arena fina: pendiente de cotización local"
- "Rendimiento de tarrajeo no proporcionado — APU incompleto"
```

---

### Paso 10 — Tabla resumen de APUs

```markdown
## RESUMEN APUs GENERADOS

| Código | Descripción | Unidad | CD Unitario | IGV | Precio Total | Estado |
|--------|-------------|--------|------------|-----|-------------|--------|
| OE.3.1 | Muros albañilería | m² | S/. 45.20 | S/. 8.14 | S/. 53.34 | VERIFICADO |
| OE.3.2 | Tarrajeo | m² | [PENDIENTE] | — | — | INCOMPLETO |
| OE.3.7 | Puertas madera | und | [PENDIENTE] | — | — | INCOMPLETO |

APUs completos: 1/3
APUs incompletos: 2/3 (faltan fuentes de precios y rendimientos)
```

## Esquema de Salida Estructurado

Lista de objetos APU según la estructura del Paso 3-8, más tabla resumen del Paso 10.

## Reglas de Validación

- **Nunca usar precios de CAPECO histórico como actuales.** CAPECO solo se acepta como referencia metodológica (estructura del APU, porcentajes, cuadrillas) si el documento es proporcionado por el usuario.
- **Nunca inventar rendimientos.** Si no hay fuente, campo queda `[PENDIENTE]`.
- **Nunca mezclar materiales técnicamente diferentes** (ladrillo artesanal ≠ industrial, cemento Tipo I ≠ Tipo V).
- Si hay ≥ 2 cotizaciones equivalentes → usar mediana. Registrar todas las fuentes.
- Todo precio debe tener fecha, fuente y URL (si aplica).
- Un APU sin precio de material crítico es `INCOMPLETO`, no `VERIFICADO`.
- Los campos vacíos representan dato ausente, no cero.

## Condiciones para Detenerse

1. El usuario no proporciona ciudad ni fecha → no proceder. Sin contexto económico el APU no tiene validez.
2. El usuario no proporciona ninguna fuente de precios → generar estructura vacía con todos los campos `[PENDIENTE]` y advertir claramente.
3. Si todos los APUs resultan INCOMPLETO → presentar lista de datos faltantes al usuario antes de avanzar a `exportar-presupuesto-excel`.

## Limitaciones Conocidas

| Limitación | Declaración |
|---|---|
| Precios de mercado | No tengo acceso en tiempo real a precios de ferreterías peruanas. Requiero que el usuario proporcione cotizaciones o URLs |
| Jornales vigentes | Solo disponibles si el usuario proporciona el documento CAPECO o MTPE vigente |
| Rendimientos regionales | Varían por zona geográfica. Sin dato del usuario, no se estiman |
| Equipos especializados | Alquiler de mezcladora, vibrador, andamios: requieren cotización local |
| Flete a obra | Depende de distancia proveedor-obra: requiere dato del usuario |
| Factor prestaciones sociales (leyes sociales) | Solo aplicar si el usuario proporciona el porcentaje vigente |

## Ejemplo de Uso

**Input del usuario:**
- Ciudad: Lima
- Fecha: Julio 2025
- Moneda: PEN
- IGV: 18%
- Proporciona: Lista de precios de materiales (PDF adjunto) + jornales CAPECO 2025

**Resultado:**
- APU OE.3.1: Muro ladrillo KK tipo soga — CD S/. 47.50/m² → con IGV S/. 56.05/m²
- Fuentes: Lista de precios proveedor (fecha), jornales CAPECO 2025 (proporcionados), rendimiento CAPECO 2025 p.78

## Prueba Mínima de Funcionamiento

```
1. Tener 1 partida clasificada (ej. OE.3.7 Puerta de madera 1 und)
2. Usuario proporciona: ciudad Lima, fecha julio 2025, precio puerta S/. 450/und, colocación S/. 80/und
3. Resultado esperado:
   - 1 APU con costo directo = S/. 530/und
   - IGV = S/. 95.40
   - Precio total = S/. 625.40
   - Estado = VERIFICADO
   - Fuente registrada
4. Si el APU retorna precio sin fuente o con precio inventado → FALLA DE PRUEBA
```

## Common Pitfalls

1. **Usar CAPECO como fuente de precios vigentes.** CAPECO publica precios que se desactualizan. Solo como metodología.
2. **Calcular precio total aunque falte algún componente.** Si hay un `[PENDIENTE]`, el total debe ser `null`.
3. **No registrar supuestos.** Un APU sin supuestos explícitos no es auditable.
4. **Normalizar unidades sin registrar el factor.** Siempre explicitar la conversión (ej. "1 saco 42.5 kg → precio/kg = precio_saco/42.5").

## Verification Checklist

- [ ] Ciudad, fecha, moneda, IGV y flete confirmados antes de calcular
- [ ] Ningún precio sin fuente explícita
- [ ] Ningún rendimiento inventado
- [ ] APUs con componentes faltantes marcados como INCOMPLETO
- [ ] Supuestos registrados en cada APU
- [ ] Al menos 2 cotizaciones comparadas cuando sea posible
- [ ] Tabla resumen con estado de cada APU generada
- [ ] Usuario informado de APUs incompletos antes de exportar
