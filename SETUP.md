# Setup

How to turn dev-brain into the control room for your own projects. The repos you work on stay
where they are, each with its own git history. dev-brain only mounts them with symlinks, so the
AI can read every repo, plan one change across several of them, and still commit each change in
the repo it belongs to.

## Target layout

```
dev-brain/
├── .claude/              # skills, commands, settings (keep at the root)
├── CLAUDE.md             # routing + safety rules the AI loads every session
├── brainstorms/          # entry point: goals and TODOs, one file per topic
├── openspec/             # specs (opt-in, for serious work)
├── designs/              # designs and task plans
├── code/                 # throwaway demos that belong to no project
├── knowledge/
│   └── projects.md       # registry: which repos exist, where, what stack
└── apps/                 # symlinks to the real repos (git-ignored)
    ├── api -> ~/code/my-api
    ├── web -> ~/code/my-web
    └── shared-lib -> ~/code/shared-lib
```

## 1. Clone and add machine-local files

```bash
git clone https://github.com/nqcdan/dev-brain.git ~/dev-brain
cd ~/dev-brain
cp secrets.example.env .claude/secrets.env
cp infrastructure.example.md .claude/infrastructure.md
chmod 600 .claude/secrets.env
```

Both files are already in `.gitignore`. Credentials never go into a tracked file.

## 2. Mount your project repos

Clone each project wherever you normally keep code, then link it into `apps/`:

```bash
mkdir apps
ln -s ~/code/my-api apps/api
ln -s ~/code/my-web apps/web
ln -s ~/code/shared-lib apps/shared-lib
```

Keep the mounted repos out of dev-brain's history by adding this to `.gitignore`:

```gitignore
apps/*
!apps/.gitkeep
```

Each symlink points to an independent repo. Branches, commits and pushes happen inside that repo,
never in dev-brain.

The same works for reference material such as a notes vault or a docs repo:

```bash
mkdir knowledge
ln -s ~/notes knowledge/notes
```

Add `knowledge/notes` to `.gitignore` too.

## 3. Write the registry

Create `knowledge/projects.md` with one row per mounted repo. This tells the AI where things live,
so it does not have to scan every repo to find out:

```markdown
# Project registry

| App | Symlink | Real path | Default branch | Stack | Depends on | Description |
|---|---|---|---|---|---|---|
| `api` | `apps/api` | `~/code/my-api` | `main` | Java/Spring | `shared-lib` | Backend API |
| `web` | `apps/web` | `~/code/my-web` | `main` | React/TS | `api` | Web frontend |
| `shared-lib` | `apps/shared-lib` | `~/code/shared-lib` | `main` | Java | — | Library used by `api` |

## Keywords → app
- billing, invoice, payment → `api`
- page, form, UI → `web`
```

The **Depends on** column is what makes cross-repo work reliable. When `shared-lib` changes,
the AI can see right away that `api` has to follow.

## 4. Add the routing rules

Add a root `CLAUDE.md` that the AI loads every session. The MVP version needs only a few rules:

```markdown
# dev-brain

Coordination workspace, not a product repo. Real code lives in `apps/<app>` (symlinks to
separate git repos).

## Routing
- Before touching any app, read `knowledge/projects.md` to scope the task: which repos,
  which paths, which order. Do not scan every repo blindly.
- If a task touches 2+ repos, list them in dependency order before starting.

## Working in apps
- Code, test and commit inside `apps/<app>`. Never copy app code into dev-brain.
- Branch off the app's default branch. Push and PRs need confirmation.
- `code/` is only for demos that belong to no app.

## Pipeline
- Default: small, reversible work goes straight from `brainstorms/` to the app.
- Opt-in for large or correctness-critical work: design → spec → implementation → review.
```

Add per-repo coding conventions and migration notes ("use the new HTTP client, not the old one")
here, or in each app's own `CLAUDE.md`. The AI follows whichever rules it can see.

## 5. Run a session

Start the AI from `brainstorms/`, one session per topic:

```bash
cd ~/dev-brain/brainstorms
claude -n add-invoice-export
```

The project root is still `dev-brain/` (where `.git` and `.claude/` are), so skills, commands
and both `CLAUDE.md` files load. Write what you want in `brainstorms/<topic>.md`:

```markdown
## TODO (dev)
- [ ] Add CSV export to invoices (api + web)

## Summary (AI)
<!-- the AI keeps this section up to date -->
```

## 6. Day-to-day flow

1. Drop a goal into `brainstorms/<topic>.md`.
2. The AI reads the registry, picks the repos involved and decides how heavy the process
   should be.
3. For larger changes it writes a design in `designs/` and a spec through `/opsx:propose`
   (see [README](README.md#openspec--spec-driven-for-the-spec-phase)), then you review them.
4. The AI implements in each affected repo on its own branch, in dependency order.
5. You review the code and merge in each repo.

The decision trail (topic → design → spec) stays in dev-brain. The code stays in the app repos.

## Checklist

- [ ] `.claude/secrets.env` and `.claude/infrastructure.md` exist and are git-ignored
- [ ] Every repo is linked under `apps/`, and `apps/*` is git-ignored
- [ ] `knowledge/projects.md` lists every app with its path, stack and dependencies
- [ ] Root `CLAUDE.md` has the routing and safety rules
- [ ] `git status` in dev-brain shows nothing from `apps/`
