# SysML v2 Modeling Skill

A concise, specification-grounded skill for working in **SysML v2** (the OMG Systems Modeling Language, built on KerML), with the textual notation as the primary, normative form. It is written as an AI agent skill, but reads cleanly as a human reference too.

## What this is

`SKILL.md` is an agent skill: a guidance file an AI assistant loads when authoring, editing, migrating, or reviewing SysML v2 models. It teaches v2 as v2 (the definition-usage pattern, requirements as evaluable constraints, model libraries plus metadata for extension) and explicitly steers away from SysML v1 / UML habits (stereotypes, BDD vs IBD, "block", PlantUML).

## Repository layout

```
sysml-v2-modeling-skill/
├── README.md                   this file
├── LICENSE                     CC BY 4.0
└── sysml-v2-modeling/
    └── SKILL.md                the skill
```

## Install and use

**Option A - as an agent skill.** Copy the `sysml-v2-modeling/` folder into your agent's skills directory (for example, a Claude Code or Cowork skills location). The folder name matches the skill's `name:` field, which is what most skill loaders expect. The agent loads it when SysML v2 work is detected.

**Option B - as context.** Open `sysml-v2-modeling/SKILL.md` and paste its contents into your assistant's system or context window before a SysML v2 task.

## Specification baseline

This skill targets OMG SysML v2.0 and its KerML foundation, with notation following the SysML v2 release train as of 2026-01. SysML v2 notation still evolves between releases; for release-sensitive constructs, verify against the current OMG specification and monthly release.

## Contributing

Issues and pull requests are welcome: corrections, clearer examples, and coverage of constructs not yet included (for example, use cases, individuals and time slices, and variation modeling).

## License

Released under [CC BY 4.0](LICENSE). You are free to share and adapt with attribution.

SysML v2 and KerML are standards of the Object Management Group (OMG). Code examples follow the OMG SysML v2 / KerML specifications and the OMG "Introduction to the SysML v2 Language." SysML is a registered trademark of OMG. This is an independent reference and is not an OMG publication.
