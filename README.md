# Create New Web App Skill

A Claude Code skill for scaffolding web applications with Azure deployment configuration.

## Installation

Download the skill file to your Claude Code commands directory using GitHub CLI.

> **Note:** Requires GitHub CLI authenticated with access to the RoskildeKommune organization. Run `gh auth login` if not already authenticated.

### Windows (PowerShell)

```powershell
# Create commands directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\commands"

# Download the skill from private repo
$content = gh api repos/RoskildeKommune/CreateNewAppSkill/contents/create-new-web-app.md --jq '.content'
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($content)) | Out-File -Encoding UTF8 "$env:USERPROFILE\.claude\commands\create-new-web-app.md"
```

### macOS/Linux (Bash)

```bash
# Create commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Download the skill from private repo
gh api repos/RoskildeKommune/CreateNewAppSkill/contents/create-new-web-app.md --jq '.content' | base64 -d > ~/.claude/commands/create-new-web-app.md
```

## Update

To update to the latest version, re-run the download command for your platform above.

## Usage

In any Claude Code session, run:

```
/create-new-web-app
```

Claude will guide you through creating a new web application with:
- React + Vite frontend
- FastAPI backend (optional)
- Azure deployment configuration
- GitHub repository setup

## Dependencies

This skill requires:
- [Git](https://git-scm.com/)
- [GitHub CLI](https://cli.github.com/) (authenticated)
- Access to [WebAppTemplate](https://github.com/RoskildeKommune/WebAppTemplate) repository

## Project Structures

The skill supports three structures:

| Structure | Use Case |
|-----------|----------|
| **Full-stack** | Dashboard/web app with backend API (React + FastAPI) |
| **Frontend-only** | Static site or app calling external APIs (React + Vite) |
| **Backend-only** | REST API or microservice (FastAPI) |
