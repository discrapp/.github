# .github

![GitHub branch status](https://img.shields.io/github/checks-status/discrapp/.github/main)
![GitHub Issues](https://img.shields.io/github/issues/discrapp/.github)
![GitHub last commit](https://img.shields.io/github/last-commit/discrapp/.github)
![GitHub repo size](https://img.shields.io/github/repo-size/discrapp/.github)
![GitHub License](https://img.shields.io/github/license/discrapp/.github)

🎯 **Discr** - Shared GitHub Actions workflows and organization configuration
for the Discr project.

This repository contains reusable GitHub Actions workflows used across all
Discr repositories.

## 🔐 HEIMDALLR_TOKEN Setup

The workflows use a GitHub Personal Access Token (PAT) for the `mosherlabs-heimdallr`
service account to post PR comments with proper attribution.

### Creating/Rotating the Token

1. **Login to GitHub** as `mosherlabs-heimdallr` (credentials in 1Password)
1. Go to **Settings** → **Developer settings** → **Personal access tokens** →
   **Tokens (classic)**
1. Click **Generate new token (classic)**
1. **Token settings:**
   - Note: `HEIMDALLR_TOKEN for discrapp`
   - Expiration: 90 days (recommended)
   - Scopes: `repo` (Full control of private repositories)
1. **Generate token** and copy it immediately
1. **Add to discrapp org:**

   ```bash
   echo "<token>" | gh secret set HEIMDALLR_TOKEN --org discrapp --visibility all
   ```

1. **Store in 1Password** under mosherlabs-heimdallr account for future reference

### Why This Token

- Provides proper attribution (gravatar, username) on PR comments
- Shared across all discrapp repos via organization secret
- Must be rotated every 90 days for security

## 💬 Slack Integration Setup

The Heimdallr workflow can send notifications to Slack when releases are created.
This requires two organization secrets from your Slack workspace.

### Getting Slack Credentials

1. **Go to Slack App Settings**
   - Visit the Slack app at <https://api.slack.com/apps>
   - Select your app (or create a new one for Discr notifications)

1. **Get the Bot User OAuth Token**
   - Navigate to **OAuth & Permissions** in the sidebar
   - Copy the **Bot User OAuth Token** (starts with `xoxb-`)
   - This token allows the bot to post messages

1. **Get the Channel ID**
   - Open Slack and navigate to the channel where you want notifications
   - Click the channel name at the top
   - Scroll down and copy the **Channel ID** (looks like `C01234ABCDE`)

### Setting the Secrets

```bash
# Set the Slack bot token
echo "<xoxb-token>" | gh secret set SLACK_BOT_USER_OAUTH_ACCESS_TOKEN \
  --org discrapp --visibility all

# Set the Slack channel ID
echo "<channel-id>" | gh secret set SLACK_PLATFORM_NOTIFICATIONS_CHANNEL_ID \
  --org discrapp --visibility all
```

### Required Slack App Permissions

Your Slack app needs these **Bot Token Scopes**:

- `chat:write` - Post messages to channels
- `chat:write.public` - Post to public channels without joining

### Why These Secrets

- `SLACK_BOT_USER_OAUTH_ACCESS_TOKEN` - Authenticates the bot to send messages
- `SLACK_PLATFORM_NOTIFICATIONS_CHANNEL_ID` - Specifies which channel receives
  notifications
- Both are organization secrets shared across all Discr repos
- Used by Heimdallr workflow when `enable_slack: true`

## 🔰 Contributing

Upon first clone, install the pre-commit hooks.

```bash
pre-commit install
```

To run pre-commit hooks locally, without a git commit.

```bash
pre-commit run -a --all-files
```

To update pre-commit hooks, this ideally should be ran before a pull request is
merged.

```bash
pre-commit autoupdate
```
