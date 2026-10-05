---
name: auditoria-planos-dwg-rvt
description: Verifica la coherencia dimensional, capas, escalas, cotas, niveles y concordancia entre planos 2D (AutoCAD DWG) y modelos BIM 3D (Revit RVT). Usar al revisar expedientes técnicos y proyectos de fin de carrera.
---

# Auditoría de Planos DWG y Modelos RVT

## Propósito y Alcance
Detectar interferencias espaciales, errores de acotado, incongruencias entre cortes y plantas y malas prácticas de modelado antes de la impresión o entrega final.

**Área de Dominio:** Asesoría Académica y Mediatecas

---

## Protocolo de Ejecución Técnica

### 1. Variables de Entrada Requeridas
- Planos en formato DWG o modelo RVT exportado.
- Nivel de detalle esperado (LOD 200, 300, 350).
- Pliego de estándares gráficos de la institución.

### 2. Procedimiento Paso a Paso
- **Paso 1:** Control de estándares de capas (Layers), grosores de línea y estilos de texto.
- **Paso 2:** Cruce dimensional entre cotas parciales y cotas totales (Detección de errores de cierre).
- **Paso 3:** Verificación de concordancia entre niveles de planta, cortes generales y elevaciones.
- **Paso 4:** Detección de elementos duplicados o líneas sueltas (Purge & Audit).
- **Paso 5:** Redacción de reporte de no conformidades con capturas y llamados a ejes estructurales.

### 3. Formato de Salida y Entregables
Reporte de auditoría de dibujo y modelado técnico con lista de discrepancias referenciadas por lámina y eje.

---

## Plantillas de Soporte, Prompts y Scripts Base

### Informe de Control de Calidad de Planos (QA/QC):
| Lámina | Ejes Afectados | Tipo de Error | Descripción de la Discrepancia | Acción Requerida |
|---|---|---|---|---|
| A-02 (Planta 1) | Eje 3 entre B y C | Cota errónea | La cota indica 3.50 m pero la distancia real medida es 3.42 m | Corregir cota y ajustar vano |
| A-05 (Corte 1-1) | Nivel +3.60 m | Incongruencia de Nivel | El corte muestra losa en +3.40 m y la planta indica +3.60 m | Unificar niveles de entrepiso |
