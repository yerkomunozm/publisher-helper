# publisher-helper

Convierte notas de voz en contenido publicado en LinkedIn usando Claude Code.

**Flujo:** nota de voz → transcripción local → newsletter + 3 posts LinkedIn → revisión automática de marca → programación en Metricool

---

## Requisitos

- [Claude Code](https://claude.ai/code)
- Python 3.10+
- MCP de **Metricool** instalado y autenticado en Claude Code
- MCP de **Higgsfield** instalado y autenticado en Claude Code (solo para carruseles)

## Setup

**1. Instala dependencias Python**

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install faster-whisper
```

**2. Configura tu voz de marca**

Edita `brand_voice.md` con tu tono, reglas de escritura y ejemplos reales de tus mejores textos. Este archivo es la fuente de verdad que usan todos los agentes para redactar y evaluar.

Edita `brand_design.md` con tu paleta, tipografía y layouts (solo necesario para carruseles).

Las plantillas de referencia están en `brand_voice — plantilla.md` y `brand_design — plantilla.md`.

**3. Crea las carpetas de trabajo**

```bash
mkdir audio output
```

## Uso

```bash
# Pipeline completo: audio → publicado
/publica-esto audio/mi-grabacion.m4a

# Generar carrusel desde una newsletter existente
/carrusels output/newsletter-2026-06-01.md
```

## Cómo funciona

El comando `/publica-esto` orquesta cuatro agentes en secuencia:

1. **Transcripción** — `tools/transcribe.py` convierte el audio a texto localmente con faster-whisper (modelo `large-v3`, sin API key).
2. **Redactor** — genera una newsletter y 3 posts de LinkedIn (ángulos conceptual, práctico y personal) respetando `brand_voice.md`.
3. **Revisor** — evalúa cada pieza contra la voz de marca y devuelve un JSON de aprobación. Si alguna pieza falla, el pipeline para.
4. **Publicador** — programa los 3 posts en Metricool (martes, jueves y sábado a las 9:00). La newsletter queda en `output/` para publicación manual.

```
output/
├── transcripcion-YYYY-MM-DD-{nombre}.txt
├── newsletter-YYYY-MM-DD.md        ← publicar manualmente
├── linkedin-YYYY-MM-DD.md          ← programado en Metricool
└── carrusel-estructura-YYYY-MM-DD.md
```

## Estructura del proyecto

```
.claude/
├── agents/          ← redactor, revisor, publicador, carrusel-designer
└── skills/          ← /publica-esto y /carrusels
tools/
└── transcribe.py    ← transcripción local con faster-whisper
brand_voice.md       ← TU VOZ (editar antes de usar)
brand_design.md      ← TU DISEÑO (editar antes de usar)
```
