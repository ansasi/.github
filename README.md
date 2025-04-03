# .github

This repository contains reusable GitHub workflows that can be applied across all my repositories.

## Available Workflows

| Workflow | Description |
| -------- | ----------- |
| Renovate | Automatically updates dependencies by creating pull requests |

## Renovate

### What it does

Renovate scans your repositories for dependencies (npm, pip, Docker, etc.) and automatically creates pull requests when updates are available.

### Setup Instructions

To set up the Renovate workflow:

1. Create a GitHub App:
   - Go to [GitHub Developer Settings](https://github.com/settings/apps/new)
   - Follow the instructions at [Creating a GitHub App](https://docs.github.com/en/developers/apps/building-oauth-apps/creating-a-github-app)
   - Make sure to grant proper permissions for repository access

2. Configure repository secrets:
   - Add `RENOVATE_BOT_APP_PRIVATE_KEY`: The private key from your GitHub App
   - Add `RENOVATE_BOT_APP_ID`: The numeric ID found in your GitHub App settings

3. Ensure the workflow file exists:
   - The workflow should be defined at `.github/workflows/renovate.yaml`

### Configuration

The Renovate configuration settings are maintained in the repository [ansasi/renovate-config](https://github.com/ansasi/renovate-config).

### Alternative

For a simpler setup, you can use the official [Renovate GitHub App](https://github.com/apps/renovate) instead of self-hosting.
