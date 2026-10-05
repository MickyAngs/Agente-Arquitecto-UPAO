---
name: optimizacion-radiacion-solar
description: Evalúa cargas de radiación solar directa y difusa sobre fachadas y cubiertas, dimensiona dispositivos de sombreado (brise-soleil, voladizos), puentes térmicos y capas de sacrificio. Usar cuando el usuario diseñe protecciones solares pasivas o analice ganancias térmicas críticas.
---

# Optimización de Radiación Solar

## Propósito y Alcance
Minimizar la ganancia térmica indeseada en épocas cálidas y maximizar la captación en periodos fríos mediante el diseño paramétrico de pantallas, capas de sacrificio ventiladas y optimización de ángulos de sombra (HSA y VSA).

**Área de Dominio:** Arquitectura Bioclimática y Sustentable

---

## Protocolo de Ejecución Técnica

### 1. Variables de Entrada Requeridas
- Latitud, acimut y ángulo de inclinación de la superficie.
- Horas solares críticas y cartas solares estereográficas del emplazamiento.
- Propiedades ópticas de materiales (reflectancia solar $\rho$, absortancia $\alpha$, emisividad $\epsilon$, SHGC de vidrios).

### 2. Procedimiento Paso a Paso
- **Paso 1:** Análisis de trayectorias solares anuales para el acimut y latitud dados.
- **Paso 2:** Determinación de los Ángulos de Sombra Horizontal (HSA) y Vertical (VSA) requeridos para bloqueo solar en horas punta.
- **Paso 3:** Dimensionamiento de lamas, voladizos o celosías paramétricas.
- **Paso 4:** Análisis del desacople térmico mediante fachadas ventiladas (capa de sacrificio con disipación por convección).
- **Paso 5:** Generación de recomendaciones de materiales reflectantes y coeficientes de sombra.

### 3. Formato de Salida y Entregables
Reporte de diseño de protección solar con cálculo de coeficientes de sombra (Shading Factor), tablas de ángulos críticos y especificación técnica de capas de sacrificio.

---

## Plantillas de Soporte, Prompts y Scripts Base

### Checklist de Auditoría de Protección Solar:
- [ ] Ángulo de incidencia solar en solsticio de verano calculado.
- [ ] Factor de Ganancia Solar Térmica (SHGC) de acristalamientos < [0.35 para climas cálidos].
- [ ] Capa de sacrificio exterior ventilada con separación mínima de 50 mm.
- [ ] Acabado exterior con Índice de Reflectancia Solar (SRI) > 78 en cubiertas.
