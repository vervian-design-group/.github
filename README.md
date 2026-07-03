# vervian-design-group / .github

Org-wide shared configuration and reusable GitHub Actions.

## OpenWiki documentation service

`/.github/workflows/openwiki-update.yml` is a **reusable workflow** that keeps a repo's
agent-facing docs fresh with [OpenWiki](https://github.com/langchain-ai/openwiki), using
OpenAI. It is managed here once and referenced by every other repo.

### One-time org setup
- Set an **organization secret** named `OPENAI_API_KEY` (Settings > Secrets and variables >
  Actions > New organization secret), scoped to the repos that should have docs.
- Enable "Allow GitHub Actions to create and approve pull requests" for the org.
- If this repo stays private, enable its workflows for org use
  (this repo > Settings > Actions > General > Access > accessible from repositories in the org).

### Add OpenWiki to a repo
Drop this stub at `.github/workflows/openwiki.yml` in the consumer repo:

```yaml
name: OpenWiki
on:
  schedule:
    - cron: "0 8 * * *"     # daily 08:00 UTC. Stagger the minute across repos.
  workflow_dispatch:
jobs:
  docs:
    uses: vervian-design-group/.github/.github/workflows/openwiki-update.yml@main
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    # with:
    #   model_id: gpt-4o
    #   node_version: "22"
```

The stub's presence is the per-repo on/off switch. Remove it to stop doc automation for that repo.
