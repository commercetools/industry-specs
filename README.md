# Industry specs

> **Freely available, AS IS and UNSUPPORTED.** This is reference content published by
> commercetools: no SLA, no security patching, not covered by commercetools Support. See
> [SUPPORT.md](SUPPORT.md) before you build on it.

Industry specs for commercetools, as [OpenSpec](https://github.com/Fission-AI/OpenSpec) capability
specs (`@commercetools-labs/industry-specs`).


## Get these specs into your project: use the CLI

The fastest way is the `@commercetools-labs/commercetools-spec-templates` CLI.
It writes the right specs into your project, and you never copy files by hand:

```bash
npx @commercetools-labs/commercetools-spec-templates init
```

Answer a few questions (framework, who the storefront sells to, industry, how much of it). Every
question has a default, nothing is written until you have confirmed the summary, and Ctrl+C
writes nothing.

- **OpenSpec or GitHub Spec Kit.** This repository is the OpenSpec half only; the CLI also
  renders Spec Kit.
- **Only what you need.** Pick your industry and business model, or take a partial set.
- **Preview first.** `plan` shows exactly what would be written, `apply` writes it:

  ```bash
  npx @commercetools-labs/commercetools-spec-templates plan  --industry grocery --model B2C
  npx @commercetools-labs/commercetools-spec-templates apply --industry grocery --model B2C
  ```

- **Safe to change your mind.** It keeps a receipt, so you can check or undo it later, and files
  you have edited are never overwritten or removed without `--force`:

  ```bash
  npx @commercetools-labs/commercetools-spec-templates status
  npx @commercetools-labs/commercetools-spec-templates remove
  ```

- **No gaps invented.** If your industry or model has no published content yet, you get the
  general storefront plus open questions, never made-up requirements.

The CLI is not in this repository. It lives in
[commercetools-spec-templates](https://github.com/commercetools/commercetools-spec-templates), which is also where these specs are authored.

## Browse the specs

Want to read them first? Pick an industry, then a business model, then a capability. `Generic`
is the general storefront every industry builds on.

| Industry | Business models | Capabilities |
|---|---|---|
| [Generic](generic/README.md) | B2C, B2B, B2B2C, B2B2B | up to 50 |
| [Automotive](automotive/README.md) | B2C, B2B, B2B2C | up to 60 |
| [Food & Grocery](grocery/README.md) | B2C, B2B, B2B2C | up to 52 |
| [Healthcare & Life Sciences](healthcare/README.md) | B2C, B2B | up to 60 |
| [Industrials & Manufacturing](manufacturing/README.md) | B2C, B2B, B2B2B | up to 70 |
| [Retail & CPG](retail/README.md) | B2C, B2B, B2B2C | up to 61 |
| [Telecom](telecom/README.md) | B2C, B2B | up to 59 |

## Or copy a spec by hand

Each business model folder has a `specs/` folder. Copy it into your project:

```bash
cp -R retail/b2c/specs/. my-project/openspec/specs/
npx -y @fission-ai/openspec@latest validate --all --strict
```

Every spec carries `[SKILL: commercetools-*]` annotations that point at skills from the
[commercetools AI Plugins](https://github.com/commercetools/commercetools-ai-plugins). Install
those first, or an agent will invent API surface instead of following the playbook.

## Licence

MIT, see `LICENSE`.
