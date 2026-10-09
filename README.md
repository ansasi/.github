# .github

## Available Workflows

| Workflow | Description |
| -------- | ----------- |
| Renovate | Automatically updates dependencies by creating pull requests |

## Renovate

### What it does

Renovate scans every repository owned by `ansasi` for dependencies (npm, pip, Docker, GitHub Actions, etc.) and opens pull requests when updates are available. It runs every 6 hours, can be started manually from the Actions tab, and opens an onboarding pull request in any repository that has no Renovate config yet.

### Setup Instructions

Renovate runs as the GitHub App [`self-hosted-renovate-ansasi`](https://github.com/settings/apps/self-hosted-renovate-ansasi). To set it up again from scratch:

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

   - Install the App on all repositories. When you add permissions later, accept them under
     [Installed GitHub Apps](https://github.com/settings/installations), or the workflow can't mint its token

2. Configure the App credentials in this repository:
   - [Variable](https://github.com/ansasi/.github/settings/variables/actions) `RENOVATE_BOT_CLIENT_ID`: The Client ID shown on the App's settings page (starts with `Iv`). It isn't sensitive, so it's a variable rather than a secret, as [create-github-app-token](https://github.com/actions/create-github-app-token#usage) recommends
   - [Secret](https://github.com/ansasi/.github/settings/secrets/actions) `RENOVATE_BOT_APP_PRIVATE_KEY`: A private key generated on the App's settings page (the whole `.pem` file)

3. Ensure the workflow file exists:
   - The workflow should be defined at `.github/workflows/renovate.yaml`

### Manual runs

Run the workflow from the Actions tab with these inputs:

| Input               | Purpose                                                                         |
| ------------------- | ------------------------------------------------------------------------------- |
| Dry Run             | Log what Renovate would do without opening branches or pull requests            |
| Autodiscover Filter | Limit the run to matching repositories, e.g. `homelab` or `docker_*`            |
| Log Level           | `debug` or `info`                                                               |
| Repository cache    | `enabled`, `disabled`, or `reset` to discard the cache and rebuild it from zero |

The repository cache is saved between runs with `actions/cache`, so Renovate only re-extracts dependencies from repositories that changed.

### Configuration

The Renovate configuration settings are maintained in the repository [ansasi/renovate-config](https://github.com/ansasi/renovate-config). This repository extends it from `.renovaterc.json5`, and onboarding pull requests in other repositories are pre-filled to extend it too.

### Alternative

For a simpler setup, you can use the official [Renovate GitHub App](https://github.com/apps/renovate) instead of self-hosting.
