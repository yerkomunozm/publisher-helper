# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Sistema de marketing personal: convierte notas de voz en newsletter + posts de LinkedIn publicados en Metricool.

**Flujo:** audio → transcripción local (faster-whisper) → redacción → revisión de marca → publicación

## Comandos de uso

```bash
# Pipeline completo desde audio
/publica-esto audio/grabacion.m4a

# Generar carrusel desde newsletter ya redactada
/carrusels output/newsletter-2026-06-01.md

# Transcribir audio suelto
.venv/bin/python tools/transcribe.py audio/grabacion.m4a
```

El entorno Python está en `.venv/`. Activar con `source .venv/bin/activate` o usar `.venv/bin/python` directamente.

## Archivos de configuración críticos

Antes de usar el pipeline, el usuario debe editar estos dos archivos con su información real:

- **`brand_voice.md`** — voz, tono, ejemplos de textos reales, reglas por canal (LinkedIn y newsletter). Es la fuente de verdad para el redactor y el revisor. Mejora añadiendo fragmentos que hayan funcionado bien.
- **`brand_design.md`** — paleta, tipografía, layouts por tipo de slide. Lo usa el carrusel-designer antes de llamar a Higgsfield.

Los archivos `brand_voice — plantilla.md` y `brand_design — plantilla.md` en la raíz son plantillas de referencia. No borrarlos.

## Arquitectura de agentes

El pipeline `/publica-esto` orquesta cuatro agentes en secuencia:

| Agente | Modelo | Condición de activación |
|---|---|---|
| `redactor` | sonnet | Siempre. Genera `output/newsletter-YYYY-MM-DD.md` y `output/linkedin-YYYY-MM-DD.md` |
| `revisor` | haiku | Siempre después del redactor. Devuelve JSON con `aprobado_global` |
| `publicador` | haiku | Solo si `aprobado_global: true`. Programa los 3 posts en Metricool |
| `carrusel-designer` | sonnet | Solo desde `/carrusels`. Lee `brand_design.md` y llama a Higgsfield MCP |

El revisor devuelve **únicamente JSON** — si hay texto adicional, es un error del agente. El publicador programa LinkedIn en Metricool; la newsletter queda en `output/` para publicación manual (no hay integración con Substack/email).

## Outputs generados

Todos los archivos se guardan en `output/` con fecha en el nombre:

- `transcripcion-YYYY-MM-DD-{stem}.txt` — texto crudo del audio
- `newsletter-YYYY-MM-DD.md` — newsletter lista para publicar manualmente
- `linkedin-YYYY-MM-DD.md` — 3 posts con ángulos conceptual, práctico y personal
- `carrusel-estructura-YYYY-MM-DD.md` — estructura de slides antes de generar con Higgsfield

## MCPs requeridos

- **Metricool** — el publicador lo usa para programar posts. Sin él, el Paso 4 falla.
- **Higgsfield** — el carrusel-designer lo usa para generar imágenes. Sin él, `/carrusels` falla.

## Reglas invariables

- Leer `brand_voice.md` antes de redactar cualquier texto (redactor y revisor).
- Leer `brand_design.md` antes de generar o estructurar carruseles.
- No inventar datos, métricas ni citas que no estén en el audio original.
- No modificar `brand_voice.md` ni `brand_design.md` sin confirmación explícita del usuario.
- El revisor debe aprobar (`aprobado_global: true`) antes de cualquier publicación.
