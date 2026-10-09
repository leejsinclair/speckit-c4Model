# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Two [Spec Kit](https://github.com/github/spec-kit) packages in one repo, both with
their manifest at the repository root so a single GitHub tag archive installs either:

- **Extension `c4`** (`extension.yml`, `commands/`, `templates/`, `config-template.yml`):
  adds the `speckit.c4.system`, `speckit.c4.feature` and `speckit.c4.validate`
  commands, two optional hooks, and the diagram templates and conventions.
- **Preset `c4-model`** (`preset.yml`, `preset/`): wraps the core `speckit.plan`
  command and appends sections to the core spec and plan templates. It needs the
  extension; `requires.extensions` only warns, it does not install it.

There is no application code, build step, linter or test suite. Every file is a
manifest, or Markdown that an AI agent reads as a prompt or a template. "Correct"
means: Spec Kit installs it, the composed commands read sensibly, and every Mermaid
block renders.

## Commands

One-time setup on a new machine (checks for `specify`, `npx`, `jq` and `claude`, and
registers the Playwright MCP server; safe to re-run):

```bash
./scripts/setup-dev.sh
```

Verify changes in a throwaway project outside this repo (use the scratchpad
directory, never `.specify/` inside this repo):

```bash
specify init <tmp-project> --integration claude
cd <tmp-project>
specify extension add --dev /home/lee/projects/speckit-c4Model
specify preset add --dev /home/lee/projects/speckit-c4Model

specify extension list && specify preset list
specify preset resolve spec-template     # core content + "## System Context"
specify preset resolve plan-template     # core content + "## Architecture"
specify preset resolve speckit.plan      # core command wrapped by preset/commands/speckit.plan.md
```

`--dev` copies the package, so after an edit re-run the extension add with `--force`,
and for the preset run `specify preset remove c4-model` then add it again.

Render every Mermaid block in a Markdown file (one SVG per block; a parse error
fails the command):

```bash
npx -y @mermaid-js/mermaid-cli -i templates/c4-container-template.md -o <tmp>/out.svg
```

Install from the published tag, as a user would:

```bash
specify extension add c4 --from https://github.com/leejsinclair/speckit-c4Model/archive/refs/tags/v0.1.0.zip
specify preset add --from https://github.com/leejsinclair/speckit-c4Model/archive/refs/tags/v0.1.0.zip
```

`specify extension add` cannot install a local zip file; use `--dev` for a local
tree or `--from <url>` for an archive. There is no `specify extension validate`
command; manifest validation only happens during `add`.

## Architecture

**What gets generated in a user's project.** Project-level living documents
(`docs/architecture/context.md`, `containers.md`, `erd.md`) written by
`speckit.c4.system`, and a per-feature `specs/<feature>/architecture.md` (C4
component views, one sequence diagram per flow, the feature ER diagram, and an
Architecture Impact table) written by `/speckit.plan` when the preset is installed,
or by `speckit.c4.feature` without it. The Architecture Impact table is the hand-off:
`speckit.c4.system` merges it into the project-level views after implementation.
The preset's plan wrapper must not edit the project-level views.

**`templates/c4-conventions.md` is the single source of rules.** All three commands
and the preset's plan wrapper tell the agent to read it first. Change a diagram rule
there, not in the individual commands. When a rule changes, update the example
diagrams in the other `templates/c4-*.md` files and in `README.md` so they still
follow it.

**How Spec Kit transforms these files at install time** (this is why paths and names
look the way they do):

- Command names must be exactly three segments, `speckit.<extension-id>.<name>`.
- In extension command files, `templates/<name>.md` is rewritten to
  `.specify/extensions/c4/templates/<name>.md`. The preset's wrapper is not
  rewritten, so it spells out the `.specify/extensions/c4/templates/...` path.
- Refer to other commands with tokens such as `__SPECKIT_COMMAND_C4_SYSTEM__`, never
  a literal slash name. Each agent integration renders the token its own way
  (`/speckit-c4-system` in Claude Code, `/speckit.c4.system` elsewhere).
- `preset/commands/speckit.plan.md` uses `strategy: wrap`. `{CORE_TEMPLATE}` is
  replaced with the core plan command, which keeps the core `scripts:` frontmatter
  and hook blocks. Text above it declares `architecture.md` as an extra Phase 1
  deliverable; text below it is the procedure.
- The two preset templates use `strategy: append`, so their sections always land at
  the end of the core template. Nothing can be inserted mid-document.
- Resolution order is project overrides > presets > extensions > core. The
  extension's templates all use new `c4-*` names and shadow nothing in core.
- Hooks must stay `optional: true` with no `condition`; core command templates skip
  hooks that carry a condition.
- Commands locate the feature with `.specify/scripts/bash/check-prerequisites.sh
  --json --paths-only` and read settings from `.specify/extensions/c4/c4-config.yml`,
  with defaults in `extension.yml` under `config.defaults`.

**Packaging.** `.extensionignore` keeps `preset/` and `preset.yml` out of the
extension install. Presets have no ignore file, so a preset install also copies the
extension's files; they are inert. Any new top-level folder that should not ship
with the extension needs an `.extensionignore` entry.

**Mermaid C4 layout.** Mermaid's C4 renderer has no automatic layout; element
positions follow declaration order. `docs/c4-layout-evaluation.md` records measured
comparisons of orderings for the template examples and found
`UpdateLayoutConfig` row settings had no effect in Mermaid CLI 12.0.0. Read it
before changing the order of declarations in a template example, and judge a layout
change from a rendered image, not from the source.

## Releasing

The extension and preset share one version number.

1. Set the same version in `extension.yml` and `preset.yml`, and add a
   `CHANGELOG.md` entry.
2. Update the two tag URLs in the Install section of `README.md`.
3. Commit, then `git tag vX.Y.Z && git push origin main vX.Y.Z`. GitHub's tag
   archive is the release artifact; there is nothing to build or upload.

## Working rules

These come from a review of past sessions (kept locally in `insights/`, which is
git-ignored).

- **Git.** Commit and push only when asked, and to the branch named. Do not create a
  new branch unless asked; if a branch seems safer, ask first. Check `git status`
  before committing and leave out ignored or runtime files.
- **Verify the real result.** After changing a manifest, command or template,
  reinstall into a throwaway project and read the composed output. After changing
  any Mermaid block, render it and look at the image. Say plainly what was not
  verified; an agent run of `/speckit.specify` then `/speckit.plan` is a separate
  check from an install check.
- **Scope.** Do not add unrequested content or sections, and do not leave out part
  of what was asked without asking first.
- **Research-only requests** ("just research and plan") mean no edits to the repo.
- **Processes.** Never stop a process with a broad `pkill -f <pattern>`; kill by PID
  or port, as its own command.
