# Contributing

This repository holds a Spec Kit extension (`c4`) and a preset (`c4-model`). There
is no application code and no build step: every file is a manifest, or Markdown that
an AI agent reads as a command or a template.

## Setup

You need the [Spec Kit CLI](https://github.com/github/spec-kit) (`specify`) 1.0.0 or
later, Node.js (for `npx`), and `jq`. [Claude Code](https://claude.com/claude-code)
is optional; the repository includes a project configuration for it.

```bash
git clone https://github.com/leejsinclair/speckit-c4Model
cd speckit-c4Model
./scripts/setup-dev.sh
```

The script checks that `specify`, `npx`, `jq` and `claude` are installed and
registers the Playwright MCP server with Claude Code. It is safe to re-run. If you
do not use Claude Code, skip it and install the first three tools yourself.

## Where things are

| Path | Contents |
|------|----------|
| `extension.yml`, `commands/`, `templates/`, `config-template.yml` | The `c4` extension |
| `preset.yml`, `preset/` | The `c4-model` preset |
| `templates/c4-conventions.md` | The rules every diagram follows. Change a rule here, then update the example diagrams in the other templates and in `README.md` to match |
| `docs/c4-layout-evaluation.md` | Measured comparison of declaration orders for the Mermaid C4 examples |
| `.claude/`, `CLAUDE.md` | Claude Code skills, hooks and guidance for this repository |
| `scripts/` | Development scripts; not shipped with the extension |

Both manifests sit at the repository root so that one GitHub tag archive installs
either package. A new top-level folder that should not be copied into users'
projects needs an entry in `.extensionignore`.

## Checking a change

Install both packages into a throwaway Spec Kit project outside this repository and
read what Spec Kit composes:

```bash
specify init /tmp/c4-check --integration claude
cd /tmp/c4-check
specify extension add --dev /path/to/speckit-c4Model
specify preset add --dev /path/to/speckit-c4Model

specify extension list && specify preset list
specify preset resolve spec-template
specify preset resolve plan-template
specify preset resolve speckit.plan
```

`--dev` copies the package, so after another edit re-run the extension command with
`--force`, and for the preset run `specify preset remove c4-model` before adding it
again.

If you changed a Mermaid block, render the file. A parse error fails the command:

```bash
npx -y @mermaid-js/mermaid-cli -i templates/c4-container-template.md -o /tmp/c4-check/out.svg
```

Look at the rendered image as well. Mermaid's C4 layout follows declaration order,
so a diagram can parse and still be hard to read.

## Pull requests

- Keep a change to one concern, and say in the description which of the checks above
  you ran.
- Add an entry to `CHANGELOG.md` under an `Unreleased` heading.
- Do not bump the version or tag a release in a pull request. The maintainer does
  that; the steps are in the Releasing section of `README.md`.
- Command names must stay in the form `speckit.c4.<name>`, and commands refer to
  each other with `__SPECKIT_COMMAND_*__` tokens, not literal slash names.

## License

This project is released under the [MIT License](LICENSE). By contributing, you agree
that your contributions are licensed under the same terms.
