# vervian-design-group / .github

Org-wide shared configuration and reusable GitHub Actions.

## OpenWiki documentation service

`/.github/workflows/openwiki-update.yml` is a **reusable workflow** that keeps a repo's
agent-facing docs fresh with [OpenWiki](https://github.com/langchain-ai/openwiki), using
OpenAI. It is managed here once and referenced by every other repo. This repo is public so
any repo (personal or org, private or public) can call the workflow. The workflow contains
no secrets.

Provider config lives in the reusable workflow itself: `OPENWIKI_PROVIDER=openai` and model
`gpt-5.4-mini` by default (override with the `model_id` input; OpenWiki also supports `gpt-5.5`).

### Onboarding a repo (three per-repo steps)

The free org plan cannot share one org secret to private repos, so each repo carries its own.

1. **Add the OpenAI key** as a repo secret named `OPENAI_API_KEY`
   (repo > Settings > Secrets and variables > Actions > New repository secret).
2. **Allow the Actions token to write and open PRs**
   (repo > Settings > Actions > General > Workflow permissions:
   set "Read and write permissions" and check "Allow GitHub Actions to create and approve pull requests").
   Or by CLI: `gh api -X PUT repos/<owner>/<repo>/actions/permissions/workflow -f default_workflow_permissions=write -F can_approve_pull_request_reviews=true`
3. **Add the stub** at `.github/workflows/openwiki.yml`:

```yaml
name: OpenWiki
on:
  schedule:
    - cron: "17 8 * * *"     # daily, staggered. Vary the minute across repos.
  workflow_dispatch:
jobs:
  docs:
    uses: vervian-design-group/.github/.github/workflows/openwiki-update.yml@main
    secrets:
      OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
    # with:
    #   model_id: gpt-5.4-mini
    #   node_version: "22"
```

Then trigger the first run manually: `gh workflow run openwiki.yml --repo <owner>/<repo>`.
It opens a PR (branch `openwiki/update`) with generated docs under `openwiki/`.

The stub's presence is the per-repo on/off switch. Remove it to stop doc automation for that repo.

### Onboard a repo with an agent (paste-and-go)

Open an agent (Claude Code, etc.) inside the repo you want to onboard and paste:

> Onboard THIS repository to the Vervian OpenWiki documentation service. The central reusable
> workflow lives at `vervian-design-group/.github`. Do all of this for the current repo:
> 1. Get this repo's `owner/name`: `gh repo view --json nameWithOwner -q .nameWithOwner`.
> 2. API key: check `gh secret list --repo <owner/name>` for `OPENAI_API_KEY`. If missing, STOP
>    and tell me to run `gh secret set OPENAI_API_KEY --repo <owner/name>` and paste my key, then
>    continue. Never enter, hardcode, or print the key yourself.
> 3. Permissions: `gh api -X PUT repos/<owner/name>/actions/permissions/workflow -f default_workflow_permissions=write -F can_approve_pull_request_reviews=true`.
> 4. Stub: create `.github/workflows/openwiki.yml` (below) on a new branch with a PR, not on main.
>    Pick a random unique cron minute 0-59 so repos do not all fire at once.
> 5. Give me the stub PR link and wait for me to merge it.
> 6. Once merged, `gh workflow run openwiki.yml --repo <owner/name>`, watch it, and confirm it
>    opened a PR on branch `openwiki/update` with docs under `openwiki/`. If it fails, read the
>    failed step log and fix it.
> 7. Report: secret existed or added, stub PR link, run result, generated-docs PR link.
>
> Stub contents:
> ```yaml
> name: OpenWiki
> on:
>   schedule:
>     - cron: "23 8 * * *"
>   workflow_dispatch:
> jobs:
>   docs:
>     uses: vervian-design-group/.github/.github/workflows/openwiki-update.yml@main
>     secrets:
>       OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
> ```

### Verified working
Piloted on `pjeremylist/vervian-api` on 2026-07-03: generated accurate `architecture.md`,
`services.md`, `operations.md`, `quickstart.md`, `testing.md` and opened them as a PR.
