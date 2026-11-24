# .github

## Available Workflows

| Workflow | Description |
| -------- | ----------- |
| Renovate | Automatically updates dependencies by creating pull requests |

## Renovate

### What it does

Renovate scans every repository owned by `ansasi` for dependencies (npm, pip, Docker, GitHub Actions, etc.) and opens pull requests when updates are available. It runs every 12 hours, can be started manually from the Actions tab, and opens an onboarding pull request in any repository that has no Renovate config yet.

### Setup Instructions

1. Create a GitHub App:
   - Go to [GitHub Developer Settings](https://github.com/settings/apps/new)
   - Follow the instructions at [Creating a GitHub App](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app)
   - Grant these repository permissions, which the workflow requests explicitly when it mints a token:

     | Permission        | Access         |
     | ----------------- | -------------- |
     | Checks            | Read and write |
     | Commit statuses   | Read and write |
     | Contents          | Read and write |
     | Dependabot alerts | Read-only      |
     | Issues            | Read and write |
     | Pull requests     | Read and write |
     | Workflows         | Read and write |

   - Install the App on all repositories

2. Configure repository secrets:
   - Add `RENOVATE_BOT_CLIENT_ID`: The Client ID shown on your GitHub App's settings page (starts with `Iv`)
   - Add `RENOVATE_BOT_APP_PRIVATE_KEY`: The private key from your GitHub App
   - Add `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`: a Docker Hub account and a personal access token with *Public Repo Read-only* access. Renovate logs in to the Docker Hub registry with them, which raises its rate limits.

3. Ensure the workflow file exists:
   - The workflow should be defined at `.github/workflows/renovate.yaml`

### Manual runs

Run the workflow from the Actions tab with these inputs:

| Input               | Purpose                                                                         |
| ------------------- | ------------------------------------------------------------------------------- |
| Dry Run             | Log what Renovate would do without opening branches or pull requests, and which undated PRs would be merged |
| Autodiscover Filter | Limit the run to matching repositories, e.g. `homelab` or `docker_*`            |
| Log Level           | `debug` or `info`                                                               |
| Repository cache    | `enabled`, `disabled`, or `reset` to discard the cache and rebuild it from zero |

The repository cache is saved between runs with `actions/cache`, so Renovate only re-extracts dependencies from repositories that changed.

### Merging undated PRs after 7 days

Images on GHCR, Quay and other registries (Helm charts stored as OCI images too) give Renovate no release date, so the shared preset never automerges them: their PRs get the `undated` label. After each Renovate run, the step *Merge undated PRs that waited 7 days* merges such a PR (squash, as Renovate does) when all of these hold:

- it has the labels `undated` and `trusted`, and one of `type/patch`, `type/minor` or `type/digest`;
- it has none of `type/major` (0.x minors get it too), `calver`, `database` or `manual`;
- the newest commit on its branch is 7 days old: a newer version pushes a new commit and the wait starts again;
- every check run and commit status on that commit passed, there is at least one, and GitHub says the PR can be merged.

It looks only at labels, so it names no app. Renovate adds the labels: the shared preset adds `undated`, `type/*`, `calver` and `database`, and each repository decides what is `trusted` or `manual` in its own config ([Undated images](https://github.com/ansasi/renovate-config#undated-images)). A repository that never adds `trusted` keeps merging its undated PRs by hand. In a grouped PR the labels of all its updates add up, so one `manual` or `calver` update keeps the whole PR for review.

The labels to look for are set in the step's environment (`REQUIRED_LABELS`, `ANY_LABELS`, `BLOCKING_LABELS`). With **Dry Run** on, it logs what it would merge without merging. A failure in this step fails the run but doesn't stop Renovate, which runs first.

### Configuration

The Renovate configuration settings are maintained in the repository [ansasi/renovate-config](https://github.com/ansasi/renovate-config). This repository extends it from `.renovaterc.json5`, and onboarding pull requests in other repositories are pre-filled to extend it too.

### Alternative

For a simpler setup, you can use the official [Renovate GitHub App](https://github.com/apps/renovate) instead of self-hosting.
