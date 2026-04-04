# Rebuilds

Each rebuild is an independent repository linked as a git submodule under `rebuilds/`.

## Adding a Rebuild

To add a new rebuild:

```bash
git submodule add https://github.com/MarcoAAlmeida/<game-name> rebuilds/<game-name>
git commit -m "Add <game-name> as submodule"
```

Each rebuild is expected to have its own `AGENTS.md` extending the
[base knowledge base](../AGENTS.md) with game-specific agent context.

## Current Rebuilds

| Game | Repository | Status |
|---|---|---|
| Karriers | [MarcoAAlmeida/karriers](https://github.com/MarcoAAlmeida/karriers) | In Progress |
