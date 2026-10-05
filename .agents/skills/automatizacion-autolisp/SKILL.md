---
name: automatizacion-autolisp
description: Diseña, depura y genera rutinas y scripts en AutoLISP / Visual LISP para automatizar tareas repetitivas de dibujo, acotado paramétrico y extracción de datos en AutoCAD. Usar cuando se requiera acelerar la producción de planos técnicos.
---

# Automatización con Scripts AutoLISP

## Propósito y Alcance
Eliminar tareas manuales de delineación mediante scripts ligeros y altamente eficientes que interactúan con la base de datos DXF de AutoCAD.

**Área de Dominio:** Asesoría Académica y Mediatecas

---

## Protocolo de Ejecución Técnica

### 1. Variables de Entrada Requeridas
- Tarea manual a automatizar (ej. numerar puertas automáticamente, sumar áreas de polilíneas, acotar ejes).
- Entidades de AutoCAD involucradas (Lines, LWPolylines, Blocks, Text).
- Variables de usuario requeridas.

### 2. Procedimiento Paso a Paso
- **Paso 1:** Definición del comando personalizado (ej. `c:AutoCota` o `c:SumarAreas`).
- **Paso 2:** Selección de entidades con filtros de selección `(ssget '((0 . "LWPOLYLINE")))`.
- **Paso 3:** Procesamiento matemático y extracción de propiedades con funciones `entget` y `assoc`.
- **Paso 4:** Creación o modificación de entidades en el dibujo.
- **Paso 5:** Gestión de variables de sistema (`CMDECHO`, `OSMODE`) y manejo de errores limpio con `*error*`.

### 3. Formato de Salida y Entregables
Código fuente `.lsp` comentado con instrucciones de carga (`APPLOAD`) y sintaxis de uso en la línea de comandos.

---

## Plantillas de Soporte, Prompts y Scripts Base

```lisp
;;; =========================================================================
;;; Rutina AutoLISP: Sumar longitud total de polilíneas seleccionadas
;;; Comando: SUMARLEN
;;; =========================================================================
(defun c:SUMARLEN ( / ss i ent len total)
  (vl-load-com)
  (setq total 0.0)
  (princ "
Seleccione las polilíneas o líneas a medir: ")
  (if (setq ss (ssget '((0 . "LINE,LWPOLYLINE,POLYLINE"))))
    (progn
      (setq i 0)
      (while (< i (sslength ss))
        (setq ent (vlax-ename->vla-object (ssname ss i)))
        (setq len (vlax-get-property ent 'Length))
        (setq total (+ total len))
        (setq i (1+ i))
      )
      (alert (strcat "Longitud total calculada: " (rtos total 2 3) " metros."))
    )
    (princ "
No se seleccionaron entidades válidas.")
  )
  (princ)
)
```
