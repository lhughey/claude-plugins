# claude-plugins

A private Claude Code plugin marketplace. Every plugin here is **copied in and
reviewed**, never installed from someone else's repository, so an upstream
change reaches us only when we choose to pull it and read the diff.

## Plugins

| Plugin | Kind | Upstream | Vendored at | License |
|---|---|---|---|---|
| `savvy-flow` | skill + 7 subagents (Markdown only, no code) | [Phalegethon/sgs-kit](https://github.com/Phalegethon/sgs-kit) `plugins/savvy-flow`, itself a fork of johnnyvizz/claude-kit | `27f096e22402455630c2866d5d0bb29136bfeb97` | MIT (`plugins/savvy-flow/LICENSE`) |
| `savvy-progress` | mod (TypeScript hooks module, runs code) | same repository, `plugins/savvy-progress` | `27f096e22402455630c2866d5d0bb29136bfeb97` | MIT (`plugins/savvy-progress/LICENSE`) |

### savvy-progress review notes

Read in full (`hooks/register.tsx`, `types/index.d.ts`). Its upstream `tests/`
were not vendored: they only run under `claude plugin test` and were not
reviewed line by line.

- **What it calls** (confirmed by `claude plugin validate`): UI, clock, its own
  plugin state, `$.session.usage`, `$.agent.list`, `$.settings.read` (only the
  `language` setting), `$.env.get` (only `LANG`, `LC_ALL`, `LC_MESSAGES`),
  `$.config.set` (only its own `savvy-progress.*` options), and it registers two
  tools and two slash commands. **No network, filesystem, process or model
  calls.**
- **What it hooks:** prompts and turns (to time them; it never rewrites a
  prompt), every model step (reads token usage only), `Agent` calls and
  subagent spawns (to draw them), and its own two tools. It does not touch the
  system prompt.
- **The one behaviour change:** `agent.spawn` can set the model of an Explore
  or general-purpose subagent that names none. Off by default (`inherit`);
  turned on only by `/agent-routing` or the plugin's options.
- **Robustness, not security:** `claude plugin validate` notes several hooks on
  the `Agent` and prompt paths have no `.catch`, so a bug in the mod could fail
  those calls rather than just losing the drawing. Disable the plugin if
  subagent launches start failing.

### Local changes (not upstream)

Versions carry a `-lh.N` suffix when this repo diverges from the vendored SHA.

- **savvy-flow `-lh.1`:** two cheaper worker tiers, `savvy-sonnet` (Sonnet,
  medium effort) and `savvy-haiku` (Haiku, low effort), added to the agents
  and to the skill's tier table.
- **savvy-progress `-lh.1`:** the panel knows the two new tiers (colour,
  planned-row model, costume, `tier` enum on the `progress` tool), and the
  price table is ported from johnnyvizz/claude-kit 1.2.1: 5.5 model rows,
  Haiku 5.5 long-prompt pricing, cache writes at the 1-hour rate.

Keep these when pulling a new upstream.

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
