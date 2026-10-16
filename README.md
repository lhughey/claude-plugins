# claude-plugins

A private Claude Code plugin marketplace. Every plugin here is **copied in and
reviewed**, never installed from someone else's repository, so an upstream
change reaches us only when we choose to pull it and read the diff.

## Plugins

| Plugin | Kind | Upstream | Vendored at | License |
|---|---|---|---|---|
| `savvy-flow` | skill + 5 subagents (Markdown only, no code) | [Phalegethon/sgs-kit](https://github.com/Phalegethon/sgs-kit) `plugins/savvy-flow`, itself a fork of johnnyvizz/claude-kit | `27f096e22402455630c2866d5d0bb29136bfeb97` | MIT (`plugins/savvy-flow/LICENSE`) |

Deliberately **not** vendored: `savvy-progress` from the same upstream. It is a
code mod that hooks prompts, tool calls and model steps, and has not been
reviewed.

## Enabling

Org-wide, in claude.ai **Organization settings → Claude Code → Managed settings**:

```json
{
  "extraKnownMarketplaces": {
    "lhughey-plugins": {
      "source": { "source": "github", "repo": "lhughey/claude-plugins" }
    }
  },
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "lhughey/claude-plugins" }
  ],
  "enabledPlugins": { "savvy-flow@lhughey-plugins": true }
}
```

Adding a plugin to this repo does not enable it anywhere; `enabledPlugins` does.

## Updating a vendored plugin

1. Clone the upstream at its latest commit.
2. `diff -ru` it against `plugins/<name>/` and **read every line**. For a
   skill or agent, the risk is instructions to the model (exfiltration,
   pushing, disabling checks), not executable code.
3. Copy in, update the *Vendored at* SHA above, and commit.
