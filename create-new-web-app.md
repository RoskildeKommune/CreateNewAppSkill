# Create New Web Application

Create a complete web application with Azure deployment configuration, ready for local development and future deployment.

## Skill Metadata
- **Trigger**: `/create-new-web-app` or when user asks to create a new web app
- **Scope**: User-level skill (global)
- **Prerequisites**: Git, GitHub CLI (gh)
- **Arguments**: `<app description>` - A brief description of the application (optional, will prompt if not provided)

## Configuration
- **GitHub Organization**: RoskildeKommune
- **Azure Resource Group**: internal_web_applications
- **App Service Plan**: ASP-internal-web-applications

## Template Sources
- **Web App Template**: https://github.com/sorabeRK/WebAppTemplate
- **Deployment Templates**: https://github.com/RoskildeKommune/deployment-templates

All project files are sourced from the WebAppTemplate repository. See its README for the complete directory structure and file contents.

---

## Instructions

When this skill is invoked, follow these steps in order:

### Step 0: Get App Description

**Check if an app description was provided as an argument.**

If the skill was invoked with an argument (e.g., `/create-new-web-app En dashboard til at vise RPA processer` or `/create-new-web-app A dashboard to display RPA processes`), use that as the app description.

If NO argument was provided, ask the user:

> **What would you like to build?**
>
> Please provide a brief description of your application in Danish or English.
>
> Examples:
> - "En intern dashboard til at overvåge RPA processer"
> - "A customer portal for viewing invoices"
> - "REST API til integration med økonomisystemet"

**Do not proceed until you have an app description.** This description will be used to:
- Suggest an appropriate app name
- Write the README and GitHub repo description
- Guide project structure recommendations

Store the description in `{{APP_DESCRIPTION}}` for use in later steps.

### Step 1: Analyze Requirements

Using the app description as context, determine the best project structure. Use **smart decision-making** based on confidence level:

#### Confidence Assessment

Before asking questions, analyze `{{APP_DESCRIPTION}}` and assess confidence for each decision:

| Confidence | When to apply | Action |
|------------|---------------|--------|
| **High** | Description explicitly states the answer | Decide automatically, no question needed |
| **Medium** | Can reasonably infer from context | Propose your assumption and ask user to confirm |
| **Low** | Ambiguous or no clear indicators | Ask the question with options |

#### High-Confidence Indicators (skip question)

**App Type (Q1):**
- "REST API", "API til...", "backend service", "microservice" → **Backend-only**
- "Dashboard", "portal", "web app", "UI til...", "vise/display" → **Web app with UI**
- "Landing page", "static site" → **Static website**

**Backend Needed (Q2):**
- "stores data", "database", "gemme data", "integration med..." → **Yes, needs backend**
- "display data from [external API]", "vise data fra..." → **No, frontend-only**

#### Medium-Confidence Indicators (propose and confirm)

- "Invoice portal" → Likely needs backend for data storage (confirm)
- "Monitoring dashboard" → Likely needs backend for real-time data (confirm)
- "Employee tool" → Could go either way (confirm what data sources)

#### Questions (ask only when needed)

**Q1: What type of application are you building?** *(Skip if high confidence)*

Options:
- A) **Dashboard or web app with UI** - Most common choice for internal tools
- B) **REST API or backend service only** - For microservices or APIs consumed by other apps
- C) **Static website** - For landing pages or simple sites without backend

*Guidance*: If unsure, choose A - you can always remove the backend later.

**Q2: Does your app need its own backend/API?** *(Skip if Q1 = B or high confidence)*

Options:
- A) **Yes** - The app needs to store data, call internal systems, or have custom business logic
- B) **No** - Frontend only (will call external APIs or be static)

*Guidance*: Choose "Yes" if you need:
- Database storage
- Integration with internal systems
- Custom business logic that shouldn't be in the frontend

**Q3: Application name**

Suggest a name based on `{{APP_DESCRIPTION}}`, but let the user confirm or change it.

Requirements:
- Must be globally unique (will be used for Azure Web App URL)
- Lowercase letters, numbers, and hyphens only
- 2-60 characters
- Will become: `https://{{APP_NAME}}.azurewebsites.net`

Examples: `rpa-monitoring`, `invoice-processor`, `customer-portal`

**Q4: Where should I create the project?**

Options:
- A) **Current directory** - Initialize here (directory should be empty)
- B) **New subdirectory** - Create `{{APP_NAME}}/` folder here

### Step 2: Validate Environment

Before creating files, check these prerequisites:

```bash
# Check Git is installed
git --version

# Check GitHub CLI is installed and authenticated
gh auth status
```

If GitHub CLI is not authenticated, guide the user:
```bash
gh auth login
```

### Step 3: Determine Project Structure

Based on answers, determine the structure:

| Q1 Answer | Q2 Answer | Structure |
|-----------|-----------|-----------|
| Dashboard (A) | Yes (A) | **Full-stack** (React + FastAPI) |
| Dashboard (A) | No (B) | **Frontend-only** (React + Vite) |
| API only (B) | - | **Backend-only** (FastAPI) |
| Static (C) | - | **Frontend-only** (React + Vite) |

### Step 4: Validate Plan

Before proceeding with implementation, present the complete plan to the user for approval.

**Display the following summary:**

```
## Project Plan Summary

| Setting | Value |
|---------|-------|
| **App Name** | {{APP_NAME}} |
| **Description** | {{APP_DESCRIPTION}} |
| **Structure** | {{STRUCTURE_TYPE}} (Full-stack / Frontend-only / Backend-only) |
| **Project Location** | {{PROJECT_DIR}} |
| **GitHub Repository** | RoskildeKommune/{{APP_NAME}} |
| **Azure URL** | https://{{APP_NAME}}.azurewebsites.net |

{{IF FULL-STACK}}
**Deployment Approach:** Single Python app serving static frontend (recommended for prototypes)
{{END IF}}

---

Does this look correct? Reply **yes** to proceed or tell me what to change.
```

**Wait for user confirmation before proceeding to Step 5.**

If the user requests changes, update the relevant values and show the summary again.

### Step 5: Clone Template Repository

Clone from the WebAppTemplate repository based on structure type:

**Full-stack (React + FastAPI):**
```bash
gh repo clone sorabeRK/WebAppTemplate {{PROJECT_DIR}}
cd {{PROJECT_DIR}}
rm -rf .git
```

**Frontend-only:**
```bash
gh repo clone sorabeRK/WebAppTemplate {{PROJECT_DIR}}
cd {{PROJECT_DIR}}
rm -rf .git backend docker-compose.yml

# Move frontend files to root level
mv frontend/* .
mv frontend/.* . 2>/dev/null || true
rm -rf frontend
```

**Backend-only:**
```bash
gh repo clone sorabeRK/WebAppTemplate {{PROJECT_DIR}}
cd {{PROJECT_DIR}}
rm -rf .git frontend docker-compose.yml templates docs

# Move backend files to root level
mv backend/* .
mv backend/.* . 2>/dev/null || true
rm -rf backend
```

### Step 6: Customize for Your Project

After cloning, update files with project-specific values:

**Required replacements:**

| Find | Replace with |
|------|-------------|
| `rpa-dashboard` | `{{APP_NAME}}` |
| `WebAppTemplate` | `{{APP_NAME}}` |

**Files to update:**

1. **package.json** - Update `"name"` field
2. **README.md** - Update title, description, and project structure section
3. **CLAUDE.md** - Update project name and description
4. **index.html** - Update `<title>` tag (if frontend)
5. **backend/main.py** - Update FastAPI title and description (if backend)

**For full-stack apps**, also update:
- `frontend/package.json`
- `frontend/index.html`

Use find-and-replace in your editor or:
```bash
# macOS/Linux
find . -type f \( -name "*.json" -o -name "*.md" -o -name "*.html" -o -name "*.py" -o -name "*.tsx" \) -exec sed -i '' 's/rpa-dashboard/{{APP_NAME}}/g' {} +

# Windows (PowerShell)
Get-ChildItem -Recurse -Include *.json,*.md,*.html,*.py,*.tsx | ForEach-Object { (Get-Content $_.FullName) -replace 'rpa-dashboard', '{{APP_NAME}}' | Set-Content $_.FullName }
```

### Step 7: Set Up GitHub Deployment Workflow

Fetch the appropriate workflow from deployment-templates:

**Backend-only (Python):**
```bash
mkdir -p .github/workflows
curl -o .github/workflows/azure-deploy.yml https://raw.githubusercontent.com/RoskildeKommune/deployment-templates/main/.github/workflow-templates/azure-webapp-python.yml
# Replace {{APP_NAME}} in the workflow file
```

**Frontend-only (Node.js):**
```bash
mkdir -p .github/workflows
curl -o .github/workflows/azure-deploy.yml https://raw.githubusercontent.com/RoskildeKommune/deployment-templates/main/.github/workflow-templates/azure-webapp-node.yml
# Replace {{APP_NAME}} in the workflow file
```

**Full-stack:**

Full-stack apps require special deployment consideration. Choose one approach:

| Approach | When to use | Setup |
|----------|-------------|-------|
| **Single Python app** (Recommended) | Prototypes, simple apps | Backend serves frontend as static files |
| **Two separate apps** | Production, independent scaling | `{{APP_NAME}}-api` + `{{APP_NAME}}-web` |
| **Docker Compose** | Local development only | Keep for local, use option 1/2 for prod |

**Option A: Single Python app** (Recommended for prototypes)

1. Add `aiofiles` to `backend/requirements.txt`
2. Update `backend/main.py` to serve static files:
```python
from fastapi.staticfiles import StaticFiles
# Add after route registration:
app.mount("/", StaticFiles(directory="static", html=True), name="static")
```
3. Use Python workflow - it will build frontend and copy to `backend/static` in CI

**Option B: Two separate apps**

Create two Azure Web Apps and deploy separately:
- `{{APP_NAME}}-api` with `azure-webapp-python.yml`
- `{{APP_NAME}}-web` with `azure-webapp-node.yml`

### Step 8: Initialize Git and Create GitHub Repository

```bash
cd {{PROJECT_DIR}}

# Install frontend dependencies (generates package-lock.json)
# For full-stack:
cd frontend && npm install && cd ..
# For frontend-only:
npm install

# Initialize git
git init

# Add all files
git add -A

# Initial commit
git commit -m "Initial project scaffolding

Created with /create-new-web-app skill.
Structure: {{STRUCTURE_TYPE}}

To run locally:
- Frontend: cd frontend && npm install && npm run dev
- Backend: cd backend && pip install -r requirements.txt && uvicorn main:app --reload

To deploy: Run /deploy-prototype

Co-Authored-By: Claude <noreply@anthropic.com>"

# Create GitHub repository
gh repo create RoskildeKommune/{{APP_NAME}} --private --description "{{APP_DESCRIPTION}}" --source=. --remote=origin --push
```

### Step 9: Show Next Steps

After successful creation, display:

```
Project created successfully!

Location: {{PROJECT_DIR}}
GitHub: https://github.com/RoskildeKommune/{{APP_NAME}}

---

## IMPORTANT: Start a New Session for Development

This skill has finished creating your project scaffolding.

For further development, open a NEW Claude Code session inside the project:

    cd {{PROJECT_DIR}}
    claude

This ensures Claude has full context of your project structure and can read CLAUDE.md for project-specific instructions.

---

## Next Steps

### 1. Start Local Development

**Frontend:**
cd {{PROJECT_DIR}}/frontend
npm install
npm run dev
# Opens at http://localhost:5173

**Backend (if full-stack):**
cd {{PROJECT_DIR}}/backend
pip install -r requirements.txt
uvicorn main:app --reload
# API at http://localhost:8000
# Docs at http://localhost:8000/docs

**Or use Docker:**
cd {{PROJECT_DIR}}
docker-compose up --build
# Frontend: http://localhost:3000
# Backend: http://localhost:8000

### 2. Deploy to Azure (when ready)

Run: /deploy-prototype

This will create Azure infrastructure and enable automatic deployments.

### 3. Useful Commands

# View in browser
gh repo view --web

# Check deployment status
gh run list

# View project structure
tree -L 3

---

Read CLAUDE.md for development instructions
Read docs/ARCHITECTURE.md for project structure
```

---

## Troubleshooting

### "Directory not empty"

If creating in current directory and it's not empty:
- Choose "New subdirectory" option instead
- Or manually empty the directory first

### "GitHub repo already exists"

Choose a different app name or use the existing repo.

### "gh not authenticated"

Run: `gh auth login`

### "npm install fails"

Ensure Node.js 20.x is installed: `node --version`

### "uvicorn not found"

Ensure Python 3.11+ is installed and requirements are installed:
```bash
pip install -r requirements.txt
```

### "Clone failed"

Check you have access to the WebAppTemplate repository:
```bash
gh repo view sorabeRK/WebAppTemplate
```

---

## Notes

- All apps use port 8000 for Azure App Service compatibility
- GitHub workflows are configured but won't run until Azure secrets are set
- Use `/deploy-prototype` to complete Azure deployment setup
- Shell scripts use LF line endings (required for Azure Linux containers)
- Template source: https://github.com/sorabeRK/WebAppTemplate
