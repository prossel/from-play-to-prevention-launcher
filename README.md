# From Play to Prevention Launcher

Specification starter project, consolidated from the design discussion on 24 November 2026.

This folder brings together the current specification, API, theme proposals and image assets. No launcher application has been implemented. Continue developing the specification before creating runtime files or choosing a framework.

## Start here

1. Read [decisions](docs/decisions.md) for the agreed direction.
2. Read the [specification](docs/specification.md) and [architecture](docs/architecture.md).
3. Review the [API](docs/api.md), [experience integration](docs/experience-integration.md) and [theme guide](docs/theme-development.md).
4. Use [open questions](docs/open-questions.md) to decide what to refine next.
5. Browse the [12 theme proposals](docs/design/theme-proposals.md) or the [visual dossier PDF](docs/design/Theme_Explorations.pdf).

## Important distinctions

- The launcher is modular and themeable; it is not committed to a phone presentation.
- The theme and experience both declare display support. The core mediates all display changes.
- Navigation hide/show is independent of display mode.
- Source structure, module hooks, metadata and protocol examples are proposals, not implemented code.
- Development documentation and identifiers are English. Interface controls/content target French-speaking users. Keep the supplied project titles unchanged.
- The visual dossier predates display-mode negotiation. It is a visual exploration, not the authoritative behaviour specification; the Markdown contracts take precedence.

## Continue in Codex

Open this extracted folder as the workspace. AGENTS.md gives the working constraints. A ready-to-use continuation prompt is in [CODEX_START.md](CODEX_START.md).

## Git and GitHub

Local Git is already initialised on `main` and contains an initial commit. At the workspace review on 24 November 2026, no Git remote was configured. Repository owner, name and visibility remain undecided.

Further commits, remote repository creation and publication require separate user authorisation. Use the actual repository URL once selected; do not infer it from the project name.

Review the included images and PDF against your repository's asset policy before publication. No license is assumed or granted for the project by this starter package.

## Folder map

| Path | Purpose |
| --- | --- |
| docs/specification.md | Product, lifecycle and acceptance criteria |
| docs/architecture.md | Core/theme boundaries and proposed future source layout |
| docs/api.md | Navigation, display negotiation, messages and examples |
| docs/experience-integration.md | Metadata, publication, embedding and team checklist |
| docs/theme-development.md | Theme authoring responsibilities |
| docs/decisions.md | Confirmed direction |
| docs/open-questions.md | Choices still requiring review |
| docs/design/ | Twelve themes, PDF and historical phone explorations |
| docs/references.md | Technical links already used in the discussion |
| CHANGELOG.md | Specification baseline history |

Use Markdown as the editable source. Avoid duplicating API definitions in other documents; link to the authoritative contract instead.
