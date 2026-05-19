# GitKraken + Cursor IDE Integration Guide

## Overview
This guide connects your GitKraken account with Cursor IDE for unified build tracking, version management, and development workflow across all your repositories.

---

## 🔐 Part 1: Authentication Setup

### Step 1: GitKraken Account Configuration

#### 1.1 Link GitHub Accounts
1. Open **GitKraken** → Click profile icon (top-right)
2. Go to **Preferences** → **Integrations**
3. Click **Connect** next to GitHub
4. Authenticate with:
   - Primary: `NextGenAILLC`
   - Secondary: `GrafUX` (add as additional account)
5. Grant permissions for:
   - `repo` (full control)
   - `user:email` (for commits)
   - `admin:repo_hook` (for webhooks)

#### 1.2 SSH Key Setup
```bash
# Generate SSH key in GitKraken
GitKraken → Preferences → SSH Keys → Generate New Key Pair

# Key name: gitkraken-cursor-$(date +%Y%m%d)
# Save passphrase securely

# Add to GitHub
GitHub Settings → SSH and GPG keys → New SSH key
# Paste public key from GitKraken
```

### Step 2: Cursor IDE Configuration

#### 2.1 Install Cursor Extensions
1. Open **Cursor**
2. Go to **Extensions** marketplace
3. Install:
   - `GitLens` (enhanced Git integration)
   - `GitHub Pull Requests` (PR management)
   - `Commit Lint` (conventional commits)
   - `Git Graph` (visual commit history)

#### 2.2 Connect GitHub to Cursor
1. **Cursor** → **Settings** → **GitHub Integration**
2. Authenticate with `NextGenAILLC` account
3. Enable:
   - [ ] Auto-fetch PRs
   - [ ] Auto-commit on save
   - [ ] Sync with remote

#### 2.3 Link GitKraken to Cursor
1. **Cursor** → **Preferences** → **Git**
2. Set Git client path: `/Applications/GitKraken/GitKraken.app/Contents/MacOS/GitKraken` (Mac) or `C:\Program Files\GitKraken\gitkraken.exe` (Windows)
3. Enable "Open in GitKraken" context menu option

---

## 🚀 Part 2: Build Tracking Integration

### Step 3: Build Version Management

#### 3.1 Automated Build Tagging with Git Hooks

Create `.githooks/prepare-commit-msg` in each Tier 1 repository:

```bash
#!/bin/bash
# File: .githooks/prepare-commit-msg

BUILD_DATE=$(date +%Y%m%d)
BUILD_NUM=$(git rev-list --count HEAD)
BUILD_TAG="v${BUILD_DATE}.${BUILD_NUM}"

# Add build reference to commit message
echo "

# Build: $BUILD_TAG" >> "$1"
```

Enable in repository:
```bash
cd NextGenAILLC/agents
git config core.hooksPath .githooks
chmod +x .githooks/*
```

#### 3.2 Conventional Commits in Cursor

Add `.commitlintrc.json` to each repository:

```json
{
  "extends": ["@commitlint/config-conventional"],
  "rules": {
    "type-enum": [
      2,
      "always",
      [
        "feat",
        "fix",
        "docs",
        "style",
        "refactor",
        "perf",
        "test",
        "build",
        "ci",
        "chore",
        "revert"
      ]
    ],
    "scope-enum": [
      2,
      "always",
      [
        "core",
        "api",
        "ui",
        "cli",
        "agent",
        "skill",
        "integration"
      ]
    ]
  }
}
```

### Step 4: GitKraken Build Dashboard

#### 4.1 Create Workspace in GitKraken

1. **GitKraken** → **File** → **Open Workspace**
2. Add all Tier 1 repositories:
   - `NextGenAILLC/agents`
   - `NextGenAILLC/fast-agent`
   - `NextGenAILLC/cline`
   - `NextGenAILLC/openclaw`
   - `NextGenAILLC/firecrawl`
   - `GrafUX/copilot-cli`
   - `GrafUX/agentskills`

3. Save workspace as: `AI-Agents-Master.gks`

#### 4.2 Configure GitKraken Insights

In GitKraken:
1. **View** → **Insights**
2. Enable:
   - [ ] Commit graph visualization
   - [ ] Author statistics
   - [ ] File change tracking
3. Set metrics period: **Last 30 days**

---

## 🔧 Part 3: Cursor Workflow Integration

### Step 5: Cursor Git Commands

#### 5.1 Create Custom Commands in Cursor

Create `.cursor/commands.json`:

```json
{
  "commands": [
    {
      "id": "git.createBuild",
      "title": "Create Build Tag",
      "description": "Create v[YYYYMMDD].[BUILD_NUM] tag",
      "category": "Git",
      "keybinding": "ctrl+shift+b"
    },
    {
      "id": "git.pushBuild",
      "title": "Push Build to Remote",
      "description": "Push tags and create GitHub Release",
      "category": "Git",
      "keybinding": "ctrl+shift+p"
    },
    {
      "id": "git.trackImprovement",
      "title": "Track Improvement Completion",
      "description": "Mark improvement as complete with PR link",
      "category": "Git"
    }
  ]
}
```

#### 5.2 Build Tracking in Cursor

Create task in **Cursor** → **Activity**:

```
📦 Build Management
├── Create build tag (v20260519.1)
├── Update BUILD_TRACKING.md
├── Create GitHub Release
├── Link improvements completed
└── Push to remote
```

### Step 6: GitHub Actions for Build Automation

Create `.github/workflows/build-release.yml`:

```yaml
name: Automated Build Release

on:
  push:
    branches: [main]
    paths-ignore:
      - "docs/**"
      - "*.md"

jobs:
  build-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Generate Build Tag
        id: build
        run: |
          BUILD_DATE=$(date +%Y%m%d)
          BUILD_NUM=$(git rev-list --count HEAD)
          echo "BUILD_TAG=v${BUILD_DATE}.${BUILD_NUM}" >> $GITHUB_OUTPUT

      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ steps.build.outputs.BUILD_TAG }}
          release_name: Build ${{ steps.build.outputs.BUILD_TAG }}
          body: |
            **Build:** ${{ steps.build.outputs.BUILD_TAG }}
            **Date:** $(date)
            **Commits:** $(git rev-list --count HEAD)
```

---

## 📊 Part 4: Integrated Monitoring Dashboard

### Step 7: Create Build Monitor Script

Create `scripts/build-monitor.sh`:

```bash
#!/bin/bash
# Build Monitoring Dashboard

REPOS=(
  "NextGenAILLC/agents"
  "NextGenAILLC/fast-agent"
  "NextGenAILLC/cline"
  "NextGenAILLC/openclaw"
  "NextGenAILLC/firecrawl"
  "GrafUX/copilot-cli"
  "GrafUX/agentskills"
)

echo "🚀 BUILD STATUS DASHBOARD"
echo "=========================="
echo "Generated: $(date)"
echo ""

for repo in "${REPOS[@]}"; do
  echo "📦 $repo"
  cd "$repo" 2>/dev/null || continue
  
  # Latest tag
  LATEST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "no-tags")
  
  # Commit count
  COMMITS=$(git rev-list --count HEAD)
  
  # Last commit date
  LAST_COMMIT=$(git log -1 --format="%ai" 2>/dev/null)
  
  # Branch status
  BRANCH=$(git rev-parse --abbrev-ref HEAD)
  
  echo "  Latest Build: $LATEST_TAG"
  echo "  Total Commits: $COMMITS"
  echo "  Last Updated: $LAST_COMMIT"
  echo "  Branch: $BRANCH"
  echo ""
done
```

Run from Cursor terminal:
```bash
bash scripts/build-monitor.sh
```

### Step 8: Real-time Build Notifications

Create `scripts/setup-notifications.sh`:

```bash
#!/bin/bash
# Setup GitKraken + Cursor notifications

# For each repository, enable webhooks
for repo in agents fast-agent cline openclaw firecrawl; do
  echo "Setting up notifications for $repo..."
  
  # GitHub webhook for Cursor notifications
  curl -X POST \
    -H "Authorization: token $GITHUB_TOKEN" \
    -H "Accept: application/vnd.github.v3+json" \
    https://api.github.com/repos/NextGenAILLC/$repo/hooks \
    -d '{
      "name": "web",
      "events": ["push", "pull_request", "release"],
      "config": {
        "url": "http://localhost:3000/webhook",
        "content_type": "json"
      }
    }'
done
```

---

## 💾 Part 5: Local Development Setup

### Step 9: Clone All Tier 1 Repos

Create `scripts/clone-all.sh`:

```bash
#!/bin/bash
# Clone all essential repositories

BASE_DIR="$HOME/Projects/AI-Agents"
mkdir -p "$BASE_DIR"
cd "$BASE_DIR"

REPOS=(
  "NextGenAILLC/agents"
  "NextGenAILLC/fast-agent"
  "NextGenAILLC/cline"
  "NextGenAILLC/openclaw"
  "NextGenAILLC/firecrawl"
  "GrafUX/copilot-cli"
  "GrafUX/agentskills"
)

for repo in "${REPOS[@]}"; do
  echo "Cloning $repo..."
  git clone git@github.com:$repo.git
done

echo "✅ All repositories cloned!"
echo "Open in GitKraken: $BASE_DIR"
```

Run:
```bash
bash scripts/clone-all.sh
```

Then open in GitKraken:
```bash
gitkraken -p "$HOME/Projects/AI-Agents"
```

### Step 10: Cursor Workspace Setup

Create `.code-workspace` file:

```json
{
  "folders": [
    {
      "path": "../agents",
      "name": "agents (Core Framework)"
    },
    {
      "path": "../fast-agent",
      "name": "fast-agent (Builder)"
    },
    {
      "path": "../cline",
      "name": "cline (Coding Agent)"
    },
    {
      "path": "../openclaw",
      "name": "openclaw (Personal AI)"
    },
    {
      "path": "../firecrawl",
      "name": "firecrawl (Web API)"
    }
  ],
  "settings": {
    "git.autofetch": true,
    "git.autorefresh": true,
    "git.pullBeforeCommit": true,
    "extensions.recommendations": [
      "eamodio.gitlens",
      "github.vscode-pull-request-github",
      "conventional-commits.conventional-commits",
      "donjayamanne.githistory"
    ]
  }
}
```

Open in Cursor:
```bash
cursor ai-agents-workspace.code-workspace
```

---

## 🔄 Part 6: Daily Workflow

### Standard Build Development Cycle

#### Morning: Check Build Status
```bash
# In Cursor terminal
bash scripts/build-monitor.sh

# In GitKraken UI
# View → Insights → Last 24h commits
```

#### Development: Make Changes
```bash
# In Cursor
# 1. Make code changes
# 2. Auto-format on save
# 3. Conventional commit message (enforced by commitlint)
# 4. GitKraken shows changes in real-time

git add .
git commit -m "feat(core): add new agent capability"
```

#### Testing: Create Feature Branch
```bash
# GitKraken: Right-click main → Create feature branch
# Name: feature/improvement-name

git checkout -b feature/improvement-name
git push -u origin feature/improvement-name
```

#### Review: Create Pull Request
```bash
# In Cursor → GitHub PR extension
# Click "Create Pull Request"
# Link improvement from BUILD_TRACKING.md
# Add tags: enhancement, verified
```

#### Release: Build Tag & Release
```bash
# When merged to main, GitHub Actions auto-tags
# Check: git tag -l | tail -5
# GitKraken: Graph shows new tag
# GitHub: Releases page shows new build
```

---

## 📋 Quick Reference

### GitKraken Shortcuts
- `Cmd+Shift+H` (Mac) / `Ctrl+Shift+H` (Windows): Open in new tab
- `Cmd+K` / `Ctrl+K`: Command palette
- `Cmd+J` / `Ctrl+J`: Toggle file list

### Cursor Shortcuts
- `Cmd+Shift+P` / `Ctrl+Shift+P`: Command palette
- `Cmd+K Cmd+O` / `Ctrl+K Ctrl+O`: Open workspace
- `Ctrl+`` : Toggle terminal

### Git Commands (from Cursor terminal)
```bash
# View build tags
git tag -l "v*" | sort -V | tail -10

# View recent commits
git log --oneline -20

# Check repository status
git status

# Push tags to remote
git push origin --tags

# View branch status
git branch -vv
```

---

## ✅ Setup Checklist

- [ ] GitHub accounts linked in GitKraken
- [ ] SSH keys generated and added to GitHub
- [ ] Cursor extensions installed
- [ ] GitKraken workspace created with all Tier 1 repos
- [ ] Git hooks configured for build tagging
- [ ] Cursor workspace file created
- [ ] GitHub Actions workflow deployed
- [ ] Build monitor script tested
- [ ] Notifications enabled
- [ ] All 7 Tier 1 repositories cloned locally

---

## 🆘 Troubleshooting

### GitKraken Won't Connect to GitHub
```bash
# Re-authenticate
rm ~/.gitkraken/config
# Restart GitKraken and log in again
```

### SSH Key Issues
```bash
# Test SSH connection
ssh -T git@github.com

# If fails, add key to SSH agent
ssh-add ~/.ssh/id_rsa
```

### Cursor Git Integration Not Working
```bash
# Check git executable location
which git

# Verify in Cursor settings:
# Preferences → Git → Path: /usr/bin/git (or similar)
```

---

## 📞 Support Resources

- [GitKraken Documentation](https://support.gitkraken.com/)
- [Cursor IDE Docs](https://cursor.com/docs)
- [GitHub SSH Setup](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [Conventional Commits](https://www.conventionalcommits.org/)

---

**Last Updated:** 2026-05-19  
**Next Review:** 2026-06-19
**Status:** 🟢 Ready for Implementation
