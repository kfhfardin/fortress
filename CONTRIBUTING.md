# Contributing to Fortress

Fortress is built and maintained by its **core team**. To protect the stealth
engine's invariants and keep the supply chain trustworthy, contributions are
accepted **only from authorized core-team members**.

## Policy

- **Unsolicited external pull requests are not accepted and will be closed.**
  This explicitly includes branding / attribution / "brand refresh" PRs,
  dependency-bump PRs, and CI / GitHub Actions workflow PRs from non-team accounts.
- **No third-party attribution, badges, or brand strings** may be added to this
  repository. Fortress is not affiliated with, and is not "built" or "maintained
  by," any outside party.
- **CI / GitHub Actions from forks is never approved to run**, and workflow files
  are only added by the core team.
- **Core-team members** coordinate with the maintainer before opening a PR. Any
  test change needing elevated permissions (CI secrets, publishing, infra) must be
  authorized in advance.
- Changes to `patches/`, the SDK, packaging, and `.github/workflows/` require
  maintainer review — see [.github/CODEOWNERS](.github/CODEOWNERS).

## Bug reports

Anyone may open an **issue** for a genuine bug or question. Code, however, lands
only through the core team.
