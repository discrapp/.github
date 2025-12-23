# Discr .github - Project Memory

This file contains persistent context for Claude Code sessions on this project.
It will be automatically loaded at the start of every session.

## Project Overview

This repository contains reusable GitHub Actions workflows for Discr projects.
It provides centralized CI/CD workflows that are consumed by other repositories
via `workflow_call`.

**Key Details:**

- **Purpose:** Shared GitHub Actions workflows
- **Consumers:** All Discr repositories (mobile, api, docs)
- **Workflows:** Release, Pre-commit, Stale, Heimdallr (notifications)
- **Pattern:** Reusable workflows with `workflow_call`
- **Forked from:** Mosher-Labs/.github (sync manually as needed)

## Repository Structure

```text
.github/
├── .github/workflows/          # Shared workflows
│   ├── coverage.yml            # Coverage reporting for PRs (uses heimdallr)
│   ├── heimdallr.yml           # Notifications (Slack/PR comments)
│   ├── pre-commit.yml          # Pre-commit hook validation
│   ├── release.yml             # Semantic versioning & releases
│   ├── stale.yml               # Stale issue/PR management
│   ├── terraform.yml           # Terraform plan/apply
│   └── test.yml                # Node.js test runner
├── .pre-commit-config.yaml
└── CLAUDE.md
```

## Workflows

### release.yml

Handles semantic versioning and release creation using Conventional Commits.

**Features:**

- Calculates next semantic version using `ietf-tools/semver-action`
- Runs on both PRs (preview) and main branch (create release)
- Integrates with Heimdallr for notifications
- Supports draft releases and custom configuration

**Usage in consuming repos:**

```yaml
jobs:
  release:
    uses: "discrapp/.github/.github/workflows/release.yml@main"
    secrets: inherit
    permissions:
      contents: write
      issues: write
      pull-requests: write
```

**Note:** Currently using `@main` for simplicity. Consider pinning to specific
version tags in the future for stability.

**On PRs:** Calculates next version, Heimdallr comments with version preview

**On main:** Creates GitHub release, Heimdallr sends Slack notification

### heimdallr.yml

Notification system for Slack messages and PR comments.

**Features:**

- Auto-detects file path vs string content for PR comments
- Sends Slack notifications (when enabled)
- Posts/updates PR comments (when enabled)
- Configurable via inputs

**Inputs:**

- `enable_slack` - Send Slack notification (default: false)
- `enable_comment_on_pr` - Post PR comment (default: false)
- `comment_content` - String or file path for comment body
- `comment_identifier` - HTML comment for identifying/updating comments
- `slack_message` - Message to send to Slack

**Auto-detect Logic:**

The workflow automatically detects whether `comment_content` is a file path
or direct string using `fs.existsSync()`. This allows:

- Release workflow to pass version strings directly
- Terraform workflow to pass plan file paths
- No changes needed in calling repos

### terraform.yml

Terraform plan and apply workflow with PR commenting.

**Features:**

- Runs `terraform plan` on PRs
- Runs `terraform apply` on main (when enabled)
- Posts plan output to PR comments
- Supports multiple Terraform Cloud workspaces

**Note:** Currently uses inline script for PR commenting. Future enhancement
is to use Heimdallr workflow once re-enabled.

## Versioning Strategy

**IMPORTANT:** All consuming repos should pin to **specific version tags**
(e.g., `@v0.10.3`) instead of using `@main`.

### Why Specific Versions

- **Stability:** Changes to `@main` won't unexpectedly break consuming repos
- **Predictability:** Repos control when they upgrade to new versions
- **Testing:** New versions can be tested before rolling out
- **Rollback:** Easy to revert to previous version if needed

### When to Update Versions

**In .github repo (after merge to main):**

- A new version tag is automatically created based on Conventional Commits
- Example: `feat:` commits create minor version (v0.10.0 → v0.11.0)
- Example: `fix:` commits create patch version (v0.10.0 → v0.10.1)

**In consuming repos:**

- Manually update the version reference when ready to adopt new features
- Test the new version in a feature branch first
- Create a PR to update the version reference
- Example:

  ```yaml
  # Update from
  uses: "Mosher-Labs/.github/.github/workflows/release.yml@v0.10.3"
  # To
  uses: "Mosher-Labs/.github/.github/workflows/release.yml@v0.11.0"
  ```

### Version Update Process

1. **Monitor .github releases:** Watch for new versions
1. **Read release notes:** Understand what changed
1. **Test in branch:** Create feature branch in consuming repo
1. **Update version:** Change `@v0.10.3` to `@v0.11.0`
1. **Verify workflow:** Check that PR workflow passes
1. **Create PR:** Review and merge to update production

## Git Workflow

### Standard Development Flow

1. **Create feature branch:** `git checkout -b feature/description`
1. **Make changes** to workflows or documentation
1. **ALWAYS run pre-commit BEFORE committing:** `pre-commit run --all-files`
   - Fix ALL errors (especially YAML and markdown linting)
   - Do NOT commit with `--no-verify` unless absolutely necessary
1. **Commit with conventional format:** `git commit -m "type: description"`
1. **Push and create PR:** `gh pr create --title "feat: description"`
1. **Test with branch reference (critical!):** Before merging, test workflows
   that reference other workflows by temporarily changing `@main` to `@your-branch`
1. **Verify tests pass** in the PR
1. **Change back to `@main`** before merging
1. **Merge to main:** Consumers will use updated workflows

### Testing Workflow Changes

**CRITICAL:** When modifying reusable workflows (like `heimdallr.yml`), you MUST
test them before merging to main.

#### Example: Testing Heimdallr changes

If you modify `heimdallr.yml` and also update `release.yml` to use the new version:

1. Create branch: `git checkout -b fix-heimdallr`
1. Make changes to `heimdallr.yml`
1. Update `release.yml` to reference your branch temporarily:

   ```yaml
   heimdallr:
     uses: discrapp/.github/.github/workflows/heimdallr.yml@fix-heimdallr
   ```

1. Commit and push
1. Verify the PR workflow runs and tests your changes
1. **Before merging:** Change back to `@main`:

   ```yaml
   heimdallr:
     uses: discrapp/.github/.github/workflows/heimdallr.yml@main
   ```

1. Commit the change back to `@main` reference
1. Merge the PR
1. Now `@main` contains your tested changes

**Why this matters:**

- Workflows that reference `@main` will break if you merge untested code
- All consuming repos use `@main`, so bugs affect everything
- Testing with branch reference catches issues before they go live

### Commit Format

Conventional Commits (enforced by pre-commit hook):

- `feat:` - New feature or workflow
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `chore:` - Maintenance
- `refactor:` - Code refactoring
- `test:` - Temporary test changes (like branch references)

## Pre-commit Hooks

**Installed hooks:**

- YAML linting (yamllint)
- Markdown linting (markdownlint)
- GitHub Actions workflow linting (actionlint)
- Conventional commit format
- File hygiene (trailing whitespace, EOF, etc.)
- Checkov (security scanning)

**Setup:**

```bash
pre-commit install              # One-time setup
pre-commit run --all-files      # Run manually
pre-commit autoupdate           # Update hook versions
```

**Note:** If actionlint-docker fails with Docker errors, you can use
`--no-verify` as a last resort, but ONLY after manually validating workflow syntax.

## Consuming Repositories

The following Discr repos use these shared workflows:

- **mobile** - React Native mobile app
- **api** - Backend API services
- **docs** - Documentation site

## Important Notes

### CRITICAL: Always Use Reusable Workflows

**MANDATORY:** When implementing any CI/CD functionality that will be used
across multiple repositories (mobile, api, web, docs), you MUST create a
reusable workflow in this `.github` repo first.

**Do NOT:**

- Implement the same workflow logic in multiple repos
- Copy/paste workflow code between repos
- Create repo-specific workflows for shared functionality

**Do:**

- Create a reusable workflow here with `workflow_call` trigger
- Have consuming repos call the shared workflow
- Pass repo-specific configuration via inputs

**Example - Coverage Reporting:**

```yaml
# In .github repo: .github/workflows/coverage.yml
on:
  workflow_call:
    inputs:
      coverage_threshold:
        type: number
        default: 50

# In consuming repo:
jobs:
  coverage:
    uses: "discrapp/.github/.github/workflows/coverage.yml@main"
    with:
      coverage_threshold: 40
      pr_statements: "75.5"
      pr_branches: "60.0"
      pr_functions: "80.0"
      pr_lines: "75.0"
    secrets:
      HEIMDALLR_TOKEN: ${{ secrets.HEIMDALLR_TOKEN }}
```

**Why this matters:**

- Single source of truth for workflow logic
- Updates automatically propagate to all repos
- No duplicate code to maintain
- Consistent behavior across all projects

### Code Quality Standards

**CRITICAL:** All code must adhere to linter rules from the start. Do NOT write
code that needs fixing after running pre-commit hooks.

**Markdown (markdownlint):**

Configuration: `.markdownlint.yaml` (allows 2-space indent, 120 char lines)

- Nested lists under unordered items: Use 2-space indentation
- Nested lists under ordered items: Use 2-space indentation
- Line length: 120 characters max (code/tables excluded)
- Use proper heading hierarchy

**YAML (yamllint):**

- Maximum line length: 80 characters
- Use 2-space indentation
- No trailing whitespace
- Proper quoting for strings containing special characters

**GitHub Actions (actionlint):**

- Use proper syntax for expressions
- Quote strings in `with:` parameters when needed
- Use `${{ }}` for all expressions
- Reference actions with specific versions (not `@latest`)

### When Working on This Repo

1. **Write linter-compliant code from the start** - Don't fix after the fact
1. **Test workflow changes** with branch references before merging
1. **Never skip testing** - "Option 2: Just merge it" is only for break-glass
1. **Run pre-commit hooks** BEFORE committing (fix all errors!)
1. **Consider impact** - Changes affect ALL consuming repos (even with version pinning)
1. **Document breaking changes** - Update this file and create migration guide
1. **After merge:** Update consuming repos to new version (create PRs for each)
1. **Communicate:** Notify team about new versions and required updates

### Common Pitfalls

- **Don't merge workflow changes without testing** - Use branch references
- **Don't use `@latest` for actions** - Pin to specific versions
- **Don't skip pre-commit hooks** - They catch workflow syntax errors
- **Don't forget to update consuming repos** - Version pins mean manual updates
- **Don't skip release notes** - Consumers need to know what changed

## References

- @README.md - Repository overview
- GitHub Actions Docs: <https://docs.github.com/en/actions>
- Reusable Workflows: <https://docs.github.com/en/actions/using-workflows/reusing-workflows>

---

**Last Updated:** 2025-11-18

This file should be updated whenever:

- New workflows are added
- Workflow patterns change
- Testing procedures change
- Important context is discovered
