---
name: calculo-convectivo-venturi
description: Calcula y simula analíticamente el efecto chimenea, tiro térmico natural, toberas Venturi y ventilación cruzada en cámaras arquitectónicas. Usar cuando se requiera dimensionar aberturas de ventilación, diferencias de presión por temperatura o aceleración de fluidos en conductos pasivos.
---

# Cálculo Convectivo Venturi

## Propósito y Alcance
Modelar cuantitativamente el movimiento natural del aire inducido por diferencias de densidad térmica (Stack Effect) y geometría restrictiva (Efecto Venturi) para maximizar la tasa de renovación de aire sin consumo energético.

**Área de Dominio:** Arquitectura Bioclimática y Sustentable

---

## Protocolo de Ejecución Técnica

### 1. Variables de Entrada Requeridas
- Altura de la columna de aire ($H$ en metros).
- Área de admisión ($A_1$) y área de garganta/salida ($A_2$) en $m^2$.
- Temperatura interior promedio ($T_i$) y temperatura exterior ($T_e$) en °C / K.
- Coeficiente de descarga de aberturas ($C_d$, típicamente 0.60 - 0.65).

### 2. Procedimiento Paso a Paso
- **Paso 1:** Determinar el gradiente térmico $\Delta T = T_i - T_e$ y la diferencia de presiones por tiro térmico $\Delta P = \rho_e g H (1 - T_e/T_i)$.
- **Paso 2:** Calcular la velocidad teórica del flujo convectivo $v = C_d \sqrt{2 g H \frac{T_i - T_e}{T_e}}$.
- **Paso 3:** Aplicar ecuación de continuidad y Bernoulli para modelar la aceleración en la tobera Venturi $v_2 = v_1 (A_1 / A_2)$.
- **Paso 4:** Calcular el caudal volumétrico total $Q (m^3/h)$ y las renovaciones por hora ($ACH$).
- **Paso 5:** Resumir resultados y sugerir ajustes geométricos de aberturas para optimizar la velocidad del aire a la altura de los ocupantes.

### 3. Formato de Salida y Entregables
Memoria de cálculo físico-matemático con ecuaciones resueltas, tabla de caudales para diferentes diferenciales térmicos y dimensionamiento geométrico óptimo de toberas.

---

## Plantillas de Soporte, Prompts y Scripts Base

```python
# Script Base de Verificación de Tiro Convectivo y Venturi
def calcular_tiro_venturi(H, A_in, A_garganta, Ti_C, Te_C, Cd=0.62):
    Ti = Ti_C + 273.15
    Te = Te_C + 273.15
    g = 9.81
    if Ti <= Te:
        return {"error": "Ti debe ser mayor a Te para tiro térmico ascendente"}
    v_in = Cd * (2 * g * H * (Ti - Te) / Te) ** 0.5
    Q = v_in * A_in
    v_garganta = (A_in / A_garganta) * v_in
    return {
        "Delta_T_K": Ti - Te,
        "Velocidad_Entrada_m_s": round(v_in, 2),
        "Velocidad_Garganta_m_s": round(v_garganta, 2),
        "Caudal_m3_s": round(Q, 3),
        "Caudal_m3_h": round(Q * 3600, 1)
    }
```
