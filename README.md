# afm-cli

A thin, domain-ignorant CLI pass-through to Apple FoundationModels. One prompt in, one text response out.

Designed as a composable building block for GitHub Actions and other automation workflows that need on-device LLM inference without copying Swift source.

---

## Input / Output Contract

```
afm-cli-bin --prompt <text> [--instructions <text>] [--temperature <double>] [--maximum-response-tokens <int>]
```

| Flag | Required | Description |
| :-- | :--: | :-- |
| `--prompt` | ✅ | The user message to send to the model |
| `--instructions` | ❌ | System-level instructions (Apple's term for system prompt) |
| `--temperature` | ❌ | Sampling temperature (Apple default if omitted) |
| `--maximum-response-tokens` | ❌ | Max tokens in the response (Apple default if omitted) |

**stdout:** plain text response from the model  
**stderr:** error messages only  
**exit code:** `0` on success, `1` on any error

---

## Design Principles

1. **No domain knowledge.** This binary knows nothing about release notes, JSON schemas, or output formats. It takes text in, returns text out.
2. **Flag names mirror the FoundationModels API exactly** — no invented vocabulary.
3. **All JSON parsing, prompt assembly, and output formatting belongs in the caller**, not here.

---

## Runtime Requirements

- Apple Silicon Mac, macOS 26+
- Apple Intelligence enabled in System Settings (per-user — may be blocked by MDM)
- No other dependencies — uses only `FoundationModels` (system framework)

---

## Download in a Workflow

```yaml
- name: Download afm-cli
  run: |
    gh release download --repo runbot-hq/afm-cli --tag v1 --pattern 'afm-cli-bin' --dir "$GITHUB_ACTION_PATH"
    chmod +x "$GITHUB_ACTION_PATH/afm-cli-bin"
  env:
    GH_TOKEN: ${{ github.token }}
```

Pin to `--tag v1` (floating tag) for stable patch updates. Each action can pin independently.

---

## Versioning

- `v1` — floating tag, advances on every build (patch fixes, no breaking changes)
- `v1.x.y` — pinned releases for callers that need exact reproducibility

---

## Future Path

When macOS 27 ships the system `fm` CLI (expected GA September 2026), `afm-cli` may become a thin wrapper or be deprecated in favour of the system binary — that's one change in one repo, not N actions.
