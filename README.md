# AGENTS.md


## Project Overview

- **MUST** design UI for human read, not AI read.
- **ALWAYS** reread lib document, not guess the answer
- **MUST** follow existing coding patterns and styles
- **MUST** NOT name functions with `build...`; use `get...` for retrieval/derivation and `create...` for construction/creation semantics
- **NEVER** keep backward compatibility (this is fresh project, not production ready), try to cleanup old code as much as possible
- **NEVER** create document file after implement if not be asked
- **ALWAYS** research existing libs first - not reinvent the wheel
- **MUST** try to use markdown table or **mermaidjs v11 skill** diagram to explain any user's question

## Tool Usage Guidelines

### Codebase Discovery & Change Analysis

**Core rule:** use `srcwalk` first for discovery/call-graph/deps; use `rg` only for last-mile text confirmation.

- Start with a narrow scope: `--scope apps/server/src` or `--scope apps/web/src`
- Do not start with `grep`, `find`, `cat`, or blind full-file reads
- For full detail (artifact mode, advanced escalation, edge cases), read `.agents/skills/srcwalk/SKILL.md`

#### Quick command map

- **MUST** design UI for human read, not AI read.
- **ALWAYS** reread lib document, not guess the answer
- **MUST** follow existing coding patterns and styles
- **MUST** NOT name functions with `build...`; use `get...` for retrieval/derivation and `create...` for construction/creation semantics
- **NEVER** keep backward compatibility (this is fresh project, not production ready), try to cleanup old code as much as possible
- **NEVER** create document file after implement if not be asked
- **ALWAYS** research existing libs first - not reinvent the wheel
- **MUST** try to use markdown table or **mermaidjs v11 skill** diagram to explain any user's question

| Need | Command |
|---|---|
| Orientation | `srcwalk map --scope <dir>` |
| Find symbol/usage | `srcwalk find <query> --scope <dir>` |
| Find files by glob | `srcwalk files '<glob>' --scope <dir>` |
| Quick bidirectional call slice | `srcwalk flow <symbol> --scope <dir>` |
| Upstream callers | `srcwalk callers <symbol> --scope <dir>` |
| Downstream callees | `srcwalk callees <symbol> --detailed --scope <dir>` |
| File blast radius | `srcwalk deps <file> --scope .` |
| Pre-edit symbol blast radius | `srcwalk impact <symbol> --scope <dir>` |
| Evidence read | `srcwalk <path>:<line-range>` or `srcwalk <path> --section <symbol>` |
| Last-mile literal search | `rg '<pattern>' -n <path>` |

#### Minimal escalation order

1. `srcwalk map --scope <dir>`
2. `srcwalk find <query> --scope <dir>`
3. `srcwalk flow <symbol> --scope <dir>`
4. `srcwalk callers <symbol> --scope <dir>` + `srcwalk callees <symbol> --detailed --scope <dir>`
5. `srcwalk deps <file> --scope .` + `srcwalk impact <symbol> --scope <dir>`
6. `srcwalk <path> --section <symbol>` or `srcwalk <path>:<start>-<end>`
7. `rg` for final literal confirmation

### Meta Skill (`ms`)

`ms` is a local-first skill management platform for agents (SQLite + Git archive, hybrid search, adaptive suggestions, MCP integration).

#### Repo integration status

- This repo is already integrated with `.ms/` and project skill paths (`./.agents/skills`, `./.claude/skills`)
- Quick environment check: `ms --version`, `ms doctor`

#### Daily workflow (important)

1. Before implementing a new pattern: `ms search "<topic>" --machine`
2. Inspect candidates: `ms show <skill-id> --machine`
3. Load the selected guidance: `ms load <skill-id> --machine`
4. If uncertain which skill to use: `ms suggest --machine`
5. After applying a skill: `ms feedback <skill-id> helpful|unhelpful`

#### Maintenance

- Re-index after adding/updating skills: `ms index`
- Health check: `ms doctor`
- Expose as MCP tools when needed: `ms mcp serve`

#### Guardrails

- Prefer machine-readable output: `--machine` or `--robot`
- Do not apply historical guidance blindly; always re-validate against current code
- Avoid interactive TUI mode in automation flows

### Bug Scanner

**Golden Rules (before every commit):**
- `ubs <changed-files>` (Exit 0 = safe. Exit >0 = fix & re-run).

#### Commands

```bash
ubs src/file.ts src/file2.ts
ubs $(but diff --name-only --staged)
ubs --only=typescript,json apps/
ubs --ci --fail-on-warning .
```

#### Output Format

```text
⚠️  Category (N errors)
    file.ts:42:5 – Issue description
    💡 Suggested fix
Exit code: 1
```

Parse: `file:line:col` → location, `💡` → suggested fix, Exit code 0/1 → pass/fail.

#### Fix Workflow

1. Read finding category and suggested fix.
2. Open `file:line:col` context.
3. Verify true issue (not false positive).
4. Fix root cause.
5. Re-run `ubs` until exit 0.

#### Bug Severity

- **Critical (always fix):** SQL injection in raw queries, unhandled promise rejections, `any` at DTO/service boundaries.
- **Important:** Missing `await`, uncaught handler exceptions, interceptor resource leaks.
- **Contextual:** TODO/FIXME/HACK, leftover `console.log`.

### Git

**MUST** use `but` as default version control workflow. **NEVER** use raw `git` for normal commit/push/branch history operations.

#### Daily Flow

- `but status --json`
- `but diff [target]`
- `but commit <branch> -m "J-<id> Type [scope] description" --changes <id|path> --json --status-after`
- `but push [branch] --json --status-after`

#### High-value Commands

- Branching: `but branch new`, `but branch show`
- Surgical changes: `but stage`, `but rub`, `but amend`, `but squash`, `but move`, `but uncommit`
- Sync: `but pull --check --json`, `but pull --json --status-after`
- Conflict handling: `but resolve status|finish|cancel`
- Recovery: `but undo`, `but oplog`
- PR flow: `but pr new`, `but pr set-draft`, `but pr set-ready`, `but pr auto-merge`

#### Conflict Playbook

1. `but status --upstream`
2. `but pull --check --json`
3. `but pull --json --status-after`
4. If conflict: `but resolve` → fix files → `but resolve finish`
5. Wrong direction: `but resolve cancel` or `but undo`

#### GitHub-hosted Operations

- PR: `gh-axi pr view|list|review|comment|merge|close|checkout|edit`
- CI: `gh-axi run list|view --log`
- Issue: `gh-axi issue list|view|comment`
- Release/labels: `gh-axi release create`, `gh-axi label list|create`
