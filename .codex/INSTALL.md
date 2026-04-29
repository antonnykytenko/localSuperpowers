# Installing localSuperpowers for Codex

Enable localSuperpowers skills in Codex via native skill discovery. This plugin
is a local fork of [obra/superpowers](https://github.com/obra/superpowers); the
source already lives in this checkout, so installation is just a symlink.

## Prerequisites

- A local checkout of this repository (referred to below as `<plugin-root>`,
  e.g. `~/agents/plugins/localSuperpowers`)

## Installation

1. **Create the skills symlink:**
   ```bash
   mkdir -p ~/.agents/skills
   ln -s <plugin-root>/skills ~/.agents/skills/localSuperpowers
   ```

   **Windows (PowerShell):**
   ```powershell
   New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills"
   cmd /c mklink /J "$env:USERPROFILE\.agents\skills\localSuperpowers" "<plugin-root>\skills"
   ```

2. **Restart Codex** (quit and relaunch the CLI) to discover the skills.

## Verify

```bash
ls -la ~/.agents/skills/localSuperpowers
```

You should see a symlink (or junction on Windows) pointing to this plugin's
`skills/` directory.

## Updating

This plugin tracks an upstream `superpowers` checkout. To pull upstream changes:

```bash
cd <plugin-root>
git fetch origin
git merge origin/main      # expect conflicts on namespace renames
```

Skills update instantly through the symlink — no Codex restart needed for
content changes.

## Uninstalling

```bash
rm ~/.agents/skills/localSuperpowers
```
