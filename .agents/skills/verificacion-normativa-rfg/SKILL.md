---
name: verificacion-normativa-rfg
description: Valida el cumplimiento de normas de eficiencia energética, confort térmico, acústico y lumínico según el Reglamento Nacional de Edificaciones (RNE - EM.110 / A.010) y estándares internacionales de construcción sustentable. Usar cuando se requiera auditar cumplimiento reglamentario técnico.
---

# Verificación Normativa RFG y RNE

## Propósito y Alcance
Garantizar la conformidad legal y técnica de anteproyectos y proyectos ejecutivos frente a normativas de envolvente térmica, ventilación mínima reglamentaria y factores de luz natural.

**Área de Dominio:** Arquitectura Bioclimática y Sustentable

---

## Protocolo de Ejecución Técnica

### 1. Variables de Entrada Requeridas
- Planos arquitectónicos y cuadro de vanos.
- Zona bioclimática según la norma EM.110 / zonificación térmica nacional.
- Especificaciones técnicas de muros, vidrios y cubiertas.

### 2. Procedimiento Paso a Paso
- **Paso 1:** Identificar la zona climática normativa aplicable al emplazamiento.
- **Paso 2:** Contrastar la transmitancia térmica ($U$) proyectada contra los valores máximos admisibles ($U_{max}$) de la norma EM.110.
- **Paso 3:** Verificar coeficientes de ventilación natural mínima y porcentaje de iluminación natural por ambiente (Norma A.010).
- **Paso 4:** Comprobar el factor solar de acristalamientos ($FS$ / $SHGC$) y puentes térmicos perimetrales.
- **Paso 5:** Elaborar ficha de no conformidades con propuestas de subsanación inmediata.

### 3. Formato de Salida y Entregables
Matriz de cumplimiento normativo estructurada por artículo, exigencia reglamentaria, valor proyectado, estado (CONFORME / NO CONFORME) y plan de acción.

---

## Plantillas de Soporte, Prompts y Scripts Base

| Norma / Artículo | Parámetro Exigido | Valor Máx / Mín Permitido | Valor en Proyecto | Estado | Acción Correctiva |
|---|---|---|---|---|---|
| EM.110 - Art 8 | Transmitancia Techo | $U \le 0.47 W/m^2K$ | $0.38 W/m^2K$ | CONFORME | N/A |
| EM.110 - Art 10 | SHGC Acristalamiento | $SHGC \le 0.40$ | $0.58$ | NO CONFORME | Instalar lámina de control solar o doble vidriado |
| A.010 - Art 51 | Área de Ventilación | $\ge 5\%$ del área útil | $8.2\%$ | CONFORME | N/A |
