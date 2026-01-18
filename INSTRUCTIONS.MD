# Instructions for AI Agents

This repo documents and dissects Hytale server and modding data. Follow these instructions when working here.

## Start Here
- Read `README.md` for orientation and pointers.
- Follow `AGENTS.md` if the task is split across agents.

## Scope and Sources
- Primary sources: `HytaleServer.jar` (in `serverexample/Server/`) and local sample mods (not included in this repo).
- When using external sources, clearly label them as external/unverified.
- Do not claim official support for inferred behavior.

## Mod Analysis Guidelines
- Treat `.jar` mods as code plugins and `.zip` mods as data/asset packs unless evidence shows otherwise.
- Look for `manifest.json` at the archive root; record `Group`, `Name`, `Version`, `Main`, and dependencies.
- For content packs, inspect `Server/Item/...`, `Server/Item/Recipes/...`, `Server/Item/Groups/...`, `Server/Item/Interactions/...`, and `Common/UI/...`.
- Summarize patterns without copying full content or code verbatim.
- Note mod-to-mod dependencies and ensure load order is documented.

## Documentation Updates
- Prefer updating these files:
  - `HytaleServer.md` for CLI/config/runtime notes
  - `HytaleServer-Dissection.md` for JAR maps and inferred guidance
  - `plugins/HytaleServer-Plugin-API.md` for API surface notes
  - `plugins/Structures-Guide.md` for prefab/structure guidance
  - `README.md` for high-level summaries and pointers
- Keep changes small and cite evidence (file paths or class names).

## Doc Update Checklist
- State the source category (JAR, sample mod, external) when adding new facts.
- Add file/path anchors for inferred guidance.
- Avoid copying content verbatim from mods; summarize patterns.
- Keep scratch artifacts local for evidence but do not treat them as published docs.
- Event docs: use `registerGlobal(...)` for player events and
  `EntityEventSystem` registered via `getEntityStoreRegistry().registerSystem(...)`
  for ECS events. Do not show ECS events registered via `EventRegistry`.

## Safety and Style
- Use ASCII only.
- Prefer `rg` for search.
- Do not modify the server JAR.
- Do not remove user changes without explicit instruction.
- Mark inferred guidance explicitly as unverified.
