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
| Dry Run             | Log what Renovate would do without opening branches or pull requests, and what the 7-day step would set without setting it |
| Autodiscover Filter | Limit the run to matching repositories, e.g. `homelab` or `docker_*`            |
| Log Level           | `debug` or `info`                                                               |
| Repository cache    | `enabled`, `disabled`, or `reset` to discard the cache and rebuild it from zero |

The repository cache is saved between runs with `actions/cache`, so Renovate only re-extracts dependencies from repositories that changed.

### The 7-day wait for undated PRs

Images on GHCR, Quay and other registries (Helm charts stored as OCI images too) give Renovate no release date, so their PRs carry the `undated` label. Before each Renovate run, the step *Set the 7-day status on undated PRs* sets a commit status, `undated-age`, on every open `undated` PR in every repository: `pending` until the newest commit on the PR's branch is 7 days old, then `success`. The shared preset automerges the ones from trusted publishers with plain versions, and Renovate merges only when every check is green, so they wait those 7 days ([Trusted publishers](https://github.com/ansasi/renovate-config#trusted-publishers)). The other `undated` PRs are still merged by hand; the status says when their week is up.

If the step fails, the job stops before Renovate runs, so no PR is merged without the status. The step uses the same app token as Renovate (*Commit statuses: write*).

### Configuration

The Renovate configuration settings are maintained in the repository [ansasi/renovate-config](https://github.com/ansasi/renovate-config). This repository extends it from `.renovaterc.json5`, and onboarding pull requests in other repositories are pre-filled to extend it too.

### Alternative

For a simpler setup, you can use the official [Renovate GitHub App](https://github.com/apps/renovate) instead of self-hosting.
