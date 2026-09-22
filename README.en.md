# Enhanced EasyEDA Schematic Skill

[简体中文](README.md) | English

This Skill guides AI agents through EasyEDA schematic design, requirement expansion, component and package evidence, local circuit drawing, functional partitioning, readable presentation, protection chains, ECO handling, and schematic-to-PCB synchronization. The current version is **2.1.4**.

## Design intent

- Turn incomplete product requirements into a concrete circuit direction while asking the user only when a decision genuinely changes the architecture.
- Use real library symbols, pin definitions, and footprints rather than invented mappings.
- Prefer short local wires for circuit topology. Use net labels and ports for cross-page, cross-block, and dense controller boundaries instead of replacing every connection with labels.
- Merge valid user changes into the latest design and apply all relevant differences to the same board during ECO work.
- Keep the schematic reviewable: functional blocks, interface meaning, power paths, protection, and annotations must remain clear.

Experimental high-power, high-speed, and FOC guidance is explicitly marked as experimental and must not be represented as production-proven without a matching validated reference design.

## Boundaries

This repository owns schematic design strategy and ECO behavior. `easyeda-api` owns generic Bridge and API operation. `easyeda-pcb-layout-routing` owns physical PCB placement and routing decisions. Live PCB mutations are executed by `easyeda-pcb-mcp`.

Read [SKILL.md](SKILL.md) first, then load only the relevant files under `references/` for component selection, schematic layout, API/ECO behavior, collaboration, or experimental design.

## Origin

The local version retains the `easyeda-schematic-net-fanout` identity and integrates experience derived from the EasyEDA enhanced schematic and API Skill projects. Upstream attribution remains documented in the Chinese README and Skill source. This private repository does not claim upstream work as original local authorship.

