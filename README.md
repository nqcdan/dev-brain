# dev-brain

An AI-first knowledge base and workspace. Developers work at both ends (specification and
verification), AI handles the middle (implementation). This is where artifacts are created and
stored along the work lifecycle, kept separate from product code for traceability and reuse.

## Structure

| Directory | Role |
|---|---|
| `brainstorms/` | Divergent phase: problems, approaches, assumptions to validate (HMW → converge) |
| `openspec/` | Spec: requirements + acceptance + "Done when (test list)", spec-driven style |
| `designs/` | How to build it: file/module boundaries, data model, API, task plan for AI |
| `code/` | Implementation and AI-generated code (or links/submodules to code repos) |

## Workflow lifecycle

```
brainstorming → spec → design → spec-review → implementation → review
   (human)     (human) (human)   (AI critic)     (AI code)       (dev)
                          └──── Ready-for-AI gate ────┘
```

- **Humans** own the first three phases (brainstorming/spec/design); an **AI critic** inspects
  spec + design (`spec-review`) to catch ambiguity and gaps before coding; **AI** implements
  (`implementation`); a **dev** reviews the AI's code (`review`) and holds the merge button.
- **Review symmetry:** AI reviews the *human's design* · dev reviews the *AI's code*.
- **Ready-for-AI gate:** spec + design are handed to AI only when there are no TBDs left,
  acceptance criteria / test list are measurable, and file/module boundaries are settled.

Mapping to the four directories: `brainstorms/` (brainstorming) · `openspec/` (spec) ·
`designs/` (design + spec-review) · `code/` (implementation + review).

For detailed conventions and workflow, see the `knowledge/` set in each project's harness.

## OpenSpec — spec-driven for the spec phase

The `spec` phase (`openspec/` directory) runs on **OpenSpec**, a spec-driven framework for AI
coding assistants: https://github.com/Fission-AI/openspec

OpenSpec's Claude Code integration is **vendored** in this repo (nothing to install just to
read it), under `.claude/`:

- `.claude/skills/openspec-*` — 6 skills: `explore` · `propose` · `apply` · `sync-specs` ·
  `update-change` · `archive-change`.
- `.claude/commands/opsx/*` — 6 matching slash commands: `/opsx:explore`, `/opsx:propose`,
  `/opsx:apply`, `/opsx:sync`, `/opsx:update`, `/opsx:archive`.

### Install the CLI to activate

The skills call the `openspec` CLI (`allowed-tools: Bash(openspec:*)`), so you need the CLI to
**run** them:

```bash
npm install -g @fission-ai/openspec@latest   # or: brew install openspec
openspec init --tools claude                 # creates the openspec/ data dir (specs + changes)
```

`openspec init` regenerates `.claude/` (already present here) and creates `openspec/` with
`specs/` + `changes/`. Then follow the loop: `/opsx:explore` → `/opsx:propose "idea"` →
`/opsx:apply` → `/opsx:archive`.

> The repo only vendors the skills/commands (readable + versioned). The CLI is machine-local
> tooling and is NOT committed to the repo.
