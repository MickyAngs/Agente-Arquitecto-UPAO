# GUÍA DE INSTALACIÓN Y CONFIGURACIÓN LOCAL: OMNIROUTE
## Asignatura: Inteligencia Artificial Arquitectura (ARQU-373) — UPAO 2026-20
**Docente Titular:** Mg. Arq. Juan Miguel Angulo García (CAP 6915)  
**Nivel:** Pregrado Profesional · Ciclo 09  

---

## 🎯 ¿QUÉ ES OMNIROUTE Y PARA QUÉ LO USAMOS EN ARQUITECTURA?
**OmniRoute** es una pasarela de Inteligencia Artificial (*AI Gateway*) que corre dentro de tu propia laptop. En lugar de estar abriendo 10 páginas web distintas o pagar costosas suscripciones mensuales, OmniRoute unifica más de **1,200 modelos de IA** detrás de un único endpoint local (`http://localhost:20128/v1`). 

Esto te permite alimentar a tus agentes en **Antigravity**, **SketchUp** y **Revit** con modelos ultra-rápidos de código abierto (como LLaMA 3.3 70B, Qwen Coder o DeepSeek) **sin pagar un solo dólar**.

---

## 🛠️ PASO 1: VERIFICAR O INSTALAR NODE.JS
OmniRoute requiere el motor de ejecución Node.js (versión 18 o superior).

1. Abre tu terminal (**PowerShell** en Windows o **Terminal** en Mac).
2. Escribe y presiona Enter:
   ```bash
   node -v
   ```
   * Si te devuelve algo como `v20.x.x` o superior: **¡Ya lo tienes listo! Pasa al Paso 2.**
   * Si te sale un error de "comando no reconocido": Entra a [https://nodejs.org](https://nodejs.org), descarga la versión **LTS** (Recomendada), instálala con *Siguiente-Siguiente* y reinicia tu terminal.

---

## 🚀 PASO 2: INICIAR OMNIROUTE EN TU COMPUTADORA
No necesitas descargar instaladores raros ni clonar repositorios pesados. Solo ejecuta este comando maestro en tu PowerShell:

```powershell
npx -y omniroute serve --daemon
```

### ¿Qué hace este comando?
1. Descarga e inicia silenciosamente OmniRoute en tu máquina.
2. La bandera `--daemon` lo deja corriendo en segundo plano como un servicio invisible (no te bloquea la consola).
3. Abre automáticamente tu navegador en el panel de control:
   👉 **`http://localhost:20128`**

---

## 🔑 PASO 3: OBTENER TU CLAVE GRATUITA DE GROQ (ALTA VELOCIDAD)
Para que tu enrutador tenga un motor de inteligencia artificial gratuito capaz de responder en menos de 1 segundo (a más de 300 tokens por segundo), usaremos **Groq**:

1. Entra a: **[https://console.groq.com](https://console.groq.com)**
2. Inicia sesión con tu cuenta de Google o correo institucional UPAO.
3. En el menú de la izquierda, haz clic en **`API Keys`** y luego en el botón **`Create API Key`**.
4. Ponle de nombre `MiOmniRoute`, dale a crear y **COPIA** la clave generada (empieza con `gsk_...`).
   > *Guarda esa clave en un bloc de notas temporal, ya que Groq no te la volverá a mostrar por seguridad.*

---

## 🔌 PASO 4: CONECTAR GROQ EN EL DASHBOARD DE OMNIROUTE
1. Ve a la pestaña del navegador donde se abrió OmniRoute (`http://localhost:20128`).
2. En el menú de la izquierda haz clic en **`Proveedores`** (o baja en la pantalla principal hasta la sección *Proveedores de claves API*).
3. Verás una tarjeta roja con el logo de **`Groq`**.
4. Haz clic directamente sobre la tarjeta de **Groq**.
5. Pega tu clave que copiaste (`gsk_...`) y haz clic en **Guardar / Conectar**.
6. Verás un aviso verde que dice: **`14 models imported (100%)`**. ¡Listo! Ya tienes IA en tu PC.

---

## 🤖 PASO 5: CONECTAR OMNIROUTE A TUS AGENTES EN ANTIGRAVITY
Ahora que tu pasarela está corriendo, configuras tu agente de Antigravity o cualquier script con estos datos universales:

| Parámetro | Valor a Configurar |
|---|---|
| **Base URL:** | `http://localhost:20128/v1` |
| **API Key:** | Cualquier texto (ej: `sk-omniroute-local`) |
| **Modelo Rápido:** | `groq/compound` o `groq/qwen/qwen3.8-27b` |
| **Modelo Razonamiento:** | `auto/best-reasoning` |

---

## 🧪 PASO 6: PRUEBA DE FUEGO EN TU TERMINAL
Para comprobar que tu enrutador responde de forma instantánea, escribe en tu terminal:

```bash
npx omniroute chat --model groq/groq/compound "Hola, dime en una sola oracion por que la arquitectura necesita IA"
```

Verás que en **menos de 1 segundo** OmniRoute te devuelve la respuesta en pantalla.

---

## ❓ PREGUNTAS FRECUENTES Y SOLUCIÓN DE PROBLEMAS

* **¿OmniRoute me va a cobrar dinero?**  
  No. OmniRoute es una herramienta open-source 100% gratuita y la cuenta de Groq es libre de coste.
* **Reinicié mi laptop y no abre el dashboard:**  
  Solo vuelve a abrir PowerShell y escribe: `npx -y omniroute serve --daemon`.
* **Me sale error de puerto ocupado (`Port 20128 in use`):**  
  Significa que OmniRoute ya está encendido en segundo plano. Solo entra a `http://localhost:20128` en tu navegador.
* **¿Puedo conectar más proveedores después?**  
  Sí, en el panel puedes conectar OpenRouter, Google Gemini Free o Moonshot/Kimi cuando lo desees.
