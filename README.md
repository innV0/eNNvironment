# eNNvironment

Synopsis y análisis del ecosistema iNNv0. Este repo documenta cómo se relacionan los proyectos del ecosistema, con foco en la integración entre **cogNNitive** e **iNNv0_skills**.

---

## cogNNitive — Motor de iNNfo

**Propósito**: Hub de especificaciones, tooling y motor del ecosistema [iNNfo](https://github.com/innV0/cogNNitive).

- Define las especificaciones iNNfo (niveles 0-2: defiNNe, iNNfo, templates)
- Implementa `@innv0/innfo-core` — parser, validador, resolvedor de cadenas de specs
- Provee `@innv0/innfo-mcp` — servidor MCP que expone tools determinísticas para AI agents
- Incluye **iNNfo Modeler** — editor Vue 3 SPA para modelos iNNfo
- Pipeline de validación CI (`pipeline-gates`)

## iNNv0_skills — Skills de Agente

**Propósito**: Colección modular de skills para [OpenCode](https://github.com/innV0/iNNv0_skills) que enseñan al AI agent a trabajar con el ecosistema iNNfo.

- `innv0-innfo` — crear, editar, validar modelos iNNfo (delega al MCP de cogNNitive)
- `innv0-trannsform` — pipeline de importación/exportación de documentos
- `innv0-router` — entry point que deriva al skill correcto
- `innv0-workflow-orchestrator` — orquestación multi-skill
- `innv0-skills-lifecycle` — instalar, auditar, mantener skills
- `innv0-site-generator` — generar sites de documentación
- `innv0-design-presets` — tokens de diseño visual

---

## Cómo se Integran

```
AI Agent → iNNv0_skills (instrucciones) → innfo-mcp (MCP server) → @innv0/innfo-core
                                                                       ↓
                                                           Specs en GitHub RAW
                                                           (cogNNitive/specs/latest/)
```

### Punto de integración principal: innfo-mcp

El MCP server es el puente. Vive en cogNNitive (`packages/innfo-mcp/`) y se distribuye como bundle compilado. iNNv0_skills lo descarga bajo demanda via `scripts/update-mcp.js` y lo registra como MCP server local en `.opencode/opencode.json`.

### URLs de especificaciones

Los skills referencian specs canónicas desde cogNNitive:

```
https://raw.githubusercontent.com/innV0/cogNNitive/main/specs/latest/level0/defiNNe_NN.md
https://raw.githubusercontent.com/innV0/cogNNitive/main/specs/latest/level1/iNNfo_NN.md
https://raw.githubusercontent.com/innV0/cogNNitive/main/specs/latest/level2/{template}/{template}_NN.md
```

### Version sync

`iNNv0_skills/scripts/update-mcp.js` fetchea la versión remota desde cogNNitive y descarga el bundle actualizado cuando hay una versión nueva.

---

## División de Responsabilidades

| Capa | cogNNitive | iNNv0_skills |
|------|-----------|--------------|
| **Rol** | Motor + especificaciones | Instrucciones al agente |
| **Contenido** | TypeScript, specs, tests, editor Vue | SKILL.md declarativos, scripts auxiliares |
| **Testing** | Vitest + Playwright (full suite) | Tests zero-dependency para CLI tools |
| **Output** | Editor web, MCP bundle, docs site | Skills instalables en `~/.agents/skills/` |
| **Versiona** | Specs iNNfo + paquetes npm | Skills individualmente (V_x-y-z) |

La separación es limpia: **cogNNitive define y ejecuta**, iNNv0_skills **instruye al agente** para usar lo que cogNNitive expone. El MCP server es la interfaz estable entre ambos.

---

## Decisiones de Arquitectura

- **MCP como interfaz**: mientras la API del MCP no cambie, ambos lados evolucionan independientemente
- **Single source of truth**: las specs viven solo en cogNNitive, referenciadas por URL
- **Bundle bajo demanda**: el MCP no se trackea en git — `update-mcp.js` lo descarga al iniciar el workspace
- **Zero-dependency tests**: los tests del CLI tool usan `assert` nativo de Node, no Vitest
- **Sin npm todavía**: el MCP cambia frecuentemente — GitHub RAW + update script es suficiente hasta v1.0 estable

---

## Documentación Relacionada

- [`ecosystem-analysis_cogNNitive-vs-iNNv0_skills.md`](ecosystem-analysis_cogNNitive-vs-iNNv0_skills.md) — Análisis detallado de integración con propuestas de mejora
