---
name: auditoria-sustentable-muba
description: Evalúa y audita envolventes modulares, cámaras convectivas y comportamiento bioclimático pasivo en Módulos Urbanos Bioclimáticos Autosuficientes (MUBA). Usar cuando el usuario solicite auditar diseño bioclimático, confort térmico pasivo, balances de calor o eficiencia de envolvente en módulos urbanos.
---

# Auditoría Sustentable MUBA

## Propósito y Alcance
Proporcionar un marco analítico riguroso para la evaluación bioclimática y energética de módulos MUBA, verificando balances térmicos, tasas de ventilación natural y desempeño de envolventes termoactivas.

**Área de Dominio:** Arquitectura Bioclimática y Sustentable

---

## Protocolo de Ejecución Técnica

### 1. Variables de Entrada Requeridas
- Modelo arquitectónico o planos de envolvente (dimensiones, capas, materiales).
- Ubicación geográfica, latitud y condiciones climáticas del sitio.
- Metadatos térmicos de materiales (valores U, transmitancia, emisividad, capacidad calorífica).

### 2. Procedimiento Paso a Paso
- **Paso 1:** Identificación de la geometría, orientación cardinal y exposición solar de cada fachada y cubierta.
- **Paso 2:** Cálculo de transmitancia térmica global (U-value) de las capas multicapa y verificación de puentes térmicos.
- **Paso 3:** Estimación del flujo convectivo interno y estratificación térmica en la cámara de aire.
- **Paso 4:** Evaluación de ganancias solares directas vs disipación nocturna pasiva.
- **Paso 5:** Emisión de reporte de auditoría con matriz de semáforo (Cumple / Requiere Optimización / No Conforme) y recomendaciones técnicas.

### 3. Formato de Salida y Entregables
Informe técnico en Markdown con tablas de transmitancia térmica, balance térmico estacional, diagrama de flujos convectivos y lista priorizada de mejoras constructivas.

---

## Plantillas de Soporte, Prompts y Scripts Base

### Formato de Salida Obligatorio:
# Informe de Auditoría Bioclimática MUBA
**Proyecto:** [Nombre] | **Fecha:** [Fecha] | **Ubicación:** [Ciudad/Coordenadas]

## 1. Caracterización de Envolvente y Materiales
| Elemento | Material / Capas | Espesor (mm) | Conductividad $\lambda$ (W/mK) | Valor U (W/m²K) |
|---|---|---|---|---|
| Muro Exterior | ... | ... | ... | ... |
| Cubierta | ... | ... | ... | ... |

## 2. Balance Térmico y Desempeño Pasivo
- Ganancia Solar Máxima ($W/m^2$): [...]
- Tasa de Renovación Convectiva ($ACH$): [...]
- Riesgo de Sobrecalentamiento: [Bajo / Moderado / Crítico]

## 3. Dictamen y Recomendaciones de Optimización
- [ ] Optimización 1: ...
- [ ] Optimización 2: ...
