# dev-brain

An AI-first knowledge base and workspace. Developers work at both ends (specification and
verification), AI handles the middle (implementation). This is where artifacts are created and
stored along the work lifecycle, kept separate from product code for traceability and reuse.

## Why this workspace

When most code is written with AI, writing it is no longer the hard part. Keeping it
understandable, consistent and reviewable is. These are the problems this repo addresses:

| Problem | How dev-brain handles it |
|---|---|
| **Decisions are lost.** A few weeks later nobody remembers why something changed or what the original requirement was. | Every change leaves a trail: brainstorm → spec → design live next to each other, versioned in git, so the "why" sits beside the "what". |
| **AI writes more code than humans can review.** One person cannot read every AI-generated PR line by line. | Review moves earlier. Humans review the spec and design, and an AI critic checks them before any code exists. Acceptance criteria and test lists make the code review a check against an agreed contract. |
| **Inconsistent code.** Formatting, naming and design patterns drift between people and sessions. | Shared conventions and rules are written once and loaded by the AI for every task, so all code follows the same style. |
| **Old and new patterns get mixed after a migration.** When a library or framework changes, AI keeps copying the old way it sees in the codebase. | Rules are updated when the migration happens, so the AI can tell the current pattern from the deprecated one. |
| **Changes span multiple repos.** Upgrading a shared library or a core service ripples into the apps that depend on it. Doing it repo by repo is slow and easy to get wrong. | One spec and one design cover the whole change across repos, and implementation is planned from there, so no dependent repo is missed. |

## Structure

To connect your own project repos, see [SETUP.md](SETUP.md).

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
