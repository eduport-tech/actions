# Centralized Vercel Deployment

This repository hosts a centralized GitHub Actions workflow to deploy multiple Vercel projects from a single location. This approach allows for a unified deployment strategy and easier secret management.

## 🚀 How it Works

1.  **Target Repo**: A "Trigger Workflow" runs on push/PR, gathers context, and sends a `repository_dispatch` event.
2.  **Central Actions Repo**: The "Deploy Workflow" listens for this event, checks out the code, and deploys it to Vercel using the provided Project IDs.

---

## 🛠 Setup Guide for New Repositories

To enable auto-deployment for a new Next.js / Vercel project:

### 1. Add Secrets to the New Repository
Go to your repository's **Settings -> Secrets and variables -> Actions** and add the following:

| Secret Name | Description | How to find it |
| :--- | :--- | :--- |
| `VERCEL_PROJECT_ID` | **(Required per Repo)** The ID of the project on Vercel. | Project Settings -> General -> Project ID. |
| `VERCEL_ORG_ID` | **(Org Secret)** The ID of your Vercel Team. | *Already configured globally.* |

### 2. Install the GitHub App
Ensure the **"Vercel Deploy Bot"** (or your equivalent GitHub App) is installed on your new repository.
*   *Permissions needed:* Read access to code, Write access to Pull Requests (for commenting).

### 3. Add the Trigger Workflow
Create a file named `.github/workflows/vercel-trigger.yml` in your new repository with the following content:

```yaml
name: Request Central Vercel Deployment

on:
  push:
    branches: [main, master]
  # pull_request: # Optional: Un-comment to deploy PRs

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Find open PR for this branch
        id: pr
        uses: jwalton/gh-find-current-pr@v1

      - name: Dispatch to central repo
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.CENTRAL_DISPATCH_TOKEN }}
          repository: eduport-tech/actions
          event-type: vercel-commit
          client-payload: >-
            {
              "repo": ${{ toJSON(github.repository) }}, 
              "branch": ${{ toJSON(github.head_ref || github.ref_name) }},
              "pr_number": ${{ toJSON(steps.pr.outputs.number) }},
              "vercel_project_id": "${{ secrets.VERCEL_PROJECT_ID }}",
              "vercel_org_id": "${{ secrets.VERCEL_ORG_ID }}"
            }
```

### 4. Configure `CENTRAL_DISPATCH_TOKEN`
Ensure your organization secrets or repository secrets have `CENTRAL_DISPATCH_TOKEN`.
*   This is a **Classic PAT** with `repo` scope that has write access to the `eduport-tech/actions` repository.

---

## ⚙️ Central Repository Configuration (`eduport-tech/actions`)

These secrets must be configured in this `actions` repository for the deployment to work.

| Secret Name | Description |
| :--- | :--- |
| `VERCEL_TOKEN` | Vercel Account Token (Full Account scope). |
| `CI_APP_ID` | App ID of the GitHub App used for cloning/commenting. |
| `CI_APP_PRIVATE_KEY` | Private Key of the GitHub App. |

---

## 🐛 Troubleshooting

*   **"Repository not found" in Trigger**: Check if `CENTRAL_DISPATCH_TOKEN` has access to `eduport-tech/actions`.
*   **"Project not found" in Vercel**: Verify `VERCEL_PROJECT_ID` matches exactly.
*   **"Author not in team"**: This workflow automatically removes `.git` to bypass this check, treating deployments as manual uploads by the token owner.
