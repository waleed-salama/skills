# Skills

Reusable Codex skills by Waleed Salama.

## Included Skills

- `linear-flow`: structured Linear workflow for discovery, triage, planning, implementation, and closeout.
- `commit-flow`: commit, push, and pull request delivery workflow with documentation checks.

## Install

Install both skills globally for Codex with:

```bash
npx skills add waleed-salama/skills -g -a codex --skill linear-flow commit-flow
```

Install one skill globally for Codex with:

```bash
npx skills add waleed-salama/skills -g -a codex --skill linear-flow
```

List available skills without installing:

```bash
npx skills add waleed-salama/skills --list
```

## Local Development

For local development, symlink the skill folders into Codex's global skills directory so edits in this repository are immediately reflected in new Codex chats:

```bash
ln -s "/path/to/skills/skills/linear-flow" "$HOME/.codex/skills/linear-flow"
ln -s "/path/to/skills/skills/commit-flow" "$HOME/.codex/skills/commit-flow"
```
