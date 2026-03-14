# Git Workflow & Repository Best Practices

This guide provides step-by-step instructions for setting up and maintaining your `terraform-and-grafana` repository on GitHub.

---

## Initial Repository Setup on GitHub

### Step 1: Create Repository on GitHub

1. Go to https://github.com/new
2. **Repository name**: `terraform-and-grafana`
3. **Description**: "Learning documentation and code for Grafana, Terraform, and Python web frameworks"
4. **Visibility**: Select `Public` (as shown in your screenshot)
5. **Initialize with**:
   - ✅ Add a README.md
   - ✅ Add .gitignore (select Python)
   - ✅ Add a license (select MIT)
6. Click **Create repository**

### Step 2: Clone Repository Locally

```bash
# Clone the repository
git clone https://github.com/yourusername/terraform-and-grafana.git

# Navigate into the directory
cd terraform-and-grafana

# Configure git user (if not already configured)
git config user.email "your.email@example.com"
git config user.name "Your Full Name"

# Verify configuration
git config --list
```

---

## Directory Setup and Initial Commit

### Step 3: Create Folder Structure

```bash
# Create main folders
mkdir terraform grafana fastapi-app django-app flask-app docs scripts

# Create subfolders
mkdir -p terraform/{environments,modules}
mkdir -p terraform/environments/{dev,staging,prod}
mkdir -p terraform/modules/{vpc,rds,ec2,security}

mkdir -p grafana/{dashboards,alerts,provisioning,scripts}
mkdir -p grafana/provisioning/{datasources,dashboards}

mkdir -p fastapi-app/app/{models,schemas,routes,services,utils,middleware}
mkdir -p fastapi-app/{tests,scripts,alembic}

mkdir -p django-app/config/{settings}
mkdir -p django-app/apps/{items,users}
mkdir -p django-app/{tests,scripts,logs}

mkdir -p flask-app/app/{models,schemas,routes,services,utils,middleware}
mkdir -p flask-app/{tests,migrations,scripts,logs}

# Create placeholder files
touch terraform/.gitkeep
touch grafana/.gitkeep
touch docs/.gitkeep
touch scripts/setup.sh
```

### Step 4: Add Documentation Files

**Copy all the markdown files we created:**

```bash
# Copy from outputs (these should already exist)
cp /path/to/README.md .
cp /path/to/REPOSITORY_STRUCTURE.md docs/
cp /path/to/FASTAPI_SETUP.md docs/
cp /path/to/DJANGO_SETUP.md docs/
cp /path/to/FLASK_SETUP.md docs/
```

### Step 5: Create Root .gitignore

```bash
# Root level .gitignore (already created, verify it has everything)
cat > .gitignore << 'EOF'
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg
MANIFEST

# Virtual environments
.venv
venv/
env/
.env
.env.local
.env.*.local

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store
.project
.pydevproject
.settings/
*.sublime-project
*.sublime-workspace

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/
.hypothesis/

# Logs
*.log
logs/

# Database
*.db
*.sqlite
*.sqlite3

# Terraform
.terraform/
.terraform.lock.hcl
terraform.tfstate
terraform.tfstate.backup
*.tfvars
!terraform.tfvars.example
crash.log

# Grafana
grafana/data/
grafana/logs/

# Docker
docker-compose.override.yml

# Environment files
.env
.env.local
.env.*.local

# OS
.DS_Store
Thumbs.db

# Instance folders
instance/
EOF
```

### Step 6: Add README.md to Root

The main `README.md` should already be in the repository root.

### Step 7: First Commit

```bash
# Check status
git status

# Add all files
git add .

# Create initial commit
git commit -m "docs: initial repository setup with documentation"

# Push to GitHub
git push -u origin main
```

---

## Creating Individual Framework Folders

### For FastAPI App

```bash
cd fastapi-app

# Create structure
mkdir -p app/{models,schemas,routes,services,utils,middleware}
mkdir -p tests/routes
mkdir scripts
mkdir alembic/versions

# Create Python files (we'll populate them later)
touch main.py
touch requirements.txt
touch .env.example
touch .gitignore
touch Dockerfile
touch docker-compose.yml
touch pytest.ini

touch app/__init__.py
touch app/config.py
touch app/database.py
touch app/dependencies.py
touch app/main.py

touch app/models/__init__.py
touch app/models/base.py
touch app/models/item.py

touch app/schemas/__init__.py
touch app/schemas/item.py
touch app/schemas/response.py

touch app/routes/__init__.py
touch app/routes/health.py
touch app/routes/items.py

touch app/services/__init__.py
touch app/services/item_service.py

touch app/utils/__init__.py
touch app/utils/logger.py
touch app/utils/exceptions.py

touch app/middleware/__init__.py
touch app/middleware/cors.py

touch tests/__init__.py
touch tests/conftest.py
touch tests/test_main.py
touch tests/routes/__init__.py
touch tests/routes/test_items.py

touch scripts/__init__.py
touch scripts/init_db.py

cd ..
```

### For Django App

```bash
cd django-app

# Create structure
mkdir -p config/settings
mkdir -p apps/items/migrations
mkdir -p apps/users/migrations
mkdir -p tests
mkdir -p scripts
mkdir -p logs
mkdir -p static
mkdir -p media

# Create files
touch manage.py
touch requirements.txt
touch .env.example
touch .gitignore
touch Dockerfile
touch docker-compose.yml
touch pytest.ini

touch config/__init__.py
touch config/settings/__init__.py
touch config/settings/base.py
touch config/settings/development.py
touch config/settings/production.py
touch config/settings/testing.py
touch config/urls.py
touch config/asgi.py
touch config/wsgi.py

touch apps/__init__.py

touch apps/items/__init__.py
touch apps/items/models.py
touch apps/items/views.py
touch apps/items/serializers.py
touch apps/items/urls.py
touch apps/items/admin.py
touch apps/items/apps.py
touch apps/items/tests.py
touch apps/items/migrations/__init__.py

touch apps/users/__init__.py
touch apps/users/models.py
touch apps/users/views.py
touch apps/users/serializers.py
touch apps/users/urls.py
touch apps/users/admin.py
touch apps/users/apps.py
touch apps/users/tests.py
touch apps/users/migrations/__init__.py

touch tests/__init__.py
touch tests/conftest.py
touch tests/test_api.py
touch tests/factories.py

touch scripts/__init__.py
touch scripts/manage_db.py
touch scripts/seed_data.py

touch .gitkeep
cd static && touch .gitkeep && cd ..
cd media && touch .gitkeep && cd ..
cd logs && touch .gitkeep && cd ..

cd ..
```

### For Flask App

```bash
cd flask-app

# Create structure
mkdir -p app/{models,schemas,routes,services,utils,middleware}
mkdir -p tests/routes
mkdir -p migrations/versions
mkdir -p scripts
mkdir -p logs
mkdir instance

# Create files
touch wsgi.py
touch requirements.txt
touch .env.example
touch .gitignore
touch Dockerfile
touch docker-compose.yml
touch pytest.ini

touch app/__init__.py
touch app/config.py
touch app/extensions.py

touch app/models/__init__.py
touch app/models/item.py

touch app/schemas/__init__.py
touch app/schemas/item.py

touch app/routes/__init__.py
touch app/routes/health.py
touch app/routes/items.py

touch app/services/__init__.py
touch app/services/item_service.py

touch app/utils/__init__.py
touch app/utils/logger.py
touch app/utils/decorators.py
touch app/utils/exceptions.py

touch app/middleware/__init__.py
touch app/middleware/cors.py

touch tests/__init__.py
touch tests/conftest.py
touch tests/test_main.py
touch tests/routes/__init__.py
touch tests/routes/test_items.py

touch scripts/__init__.py

touch logs/.gitkeep
touch instance/.gitkeep

cd ..
```

### For Terraform

```bash
cd terraform

# Create files
touch README.md
touch main.tf
touch variables.tf
touch outputs.tf
touch provider.tf
touch backend.tf
touch terraform.tfvars.example

# Environment configs
mkdir -p environments/{dev,staging,prod}
touch environments/dev/terraform.tfvars
touch environments/dev/backend.tf
touch environments/staging/terraform.tfvars
touch environments/staging/backend.tf
touch environments/prod/terraform.tfvars
touch environments/prod/backend.tf

# Modules
mkdir -p modules/{vpc,rds,ec2,security}
for module in vpc rds ec2 security; do
  touch modules/$module/main.tf
  touch modules/$module/variables.tf
  touch modules/$module/outputs.tf
done

cd ..
```

### For Grafana

```bash
cd grafana

# Create structure
mkdir -p dashboards
mkdir -p alerts
mkdir -p provisioning/datasources
mkdir -p provisioning/dashboards
mkdir -p scripts

# Create files
touch README.md
touch docker-compose.yml
touch .env.example

touch dashboards/system-monitoring.json
touch dashboards/application-metrics.json

touch alerts/alert-rules.yaml

touch provisioning/datasources/prometheus.yml
touch provisioning/dashboards/dashboard-provider.yml

touch scripts/setup-grafana.sh

cd ..
```

---

## Git Workflow - Creating Feature Branches

### Standard Git Workflow

```bash
# 1. Update main branch
git checkout main
git pull origin main

# 2. Create feature branch
# Format: feature/short-description or fix/issue-name
git checkout -b feature/add-fastapi-models

# 3. Make changes
# ... edit files ...

# 4. Check what changed
git status
git diff

# 5. Stage changes
git add app/models/
git add app/schemas/

# Or add specific files
git add app/models/item.py
git add app/schemas/item.py

# Or add all changes
git add .

# 6. Create commit with descriptive message
git commit -m "feat: add item model and schema for FastAPI"

# Commit message format:
# feat: new feature
# fix: bug fix
# docs: documentation changes
# refactor: code refactoring
# test: test additions/modifications
# style: code style changes (formatting)
# chore: maintenance tasks

# 7. Push branch to GitHub
git push -u origin feature/add-fastapi-models

# 8. Create Pull Request on GitHub
# Go to https://github.com/yourusername/terraform-and-grafana/pulls
# Click "New Pull Request"
# Select your branch
# Add description and click "Create Pull Request"
```

---

## Common Git Commands

### View Repository Status

```bash
# Current status
git status

# View recent commits
git log --oneline -10

# View branches
git branch -a

# View all changes
git diff

# View staged changes
git diff --cached
```

### Working with Branches

```bash
# Create and switch to new branch
git checkout -b feature/my-feature

# Or using newer syntax
git switch -c feature/my-feature

# Switch to existing branch
git checkout feature/my-feature

# Delete branch locally
git branch -d feature/my-feature

# Delete branch on GitHub
git push origin --delete feature/my-feature

# List all branches
git branch -a

# Rename branch
git branch -m old-name new-name
```

### Undoing Changes

```bash
# Unstage file
git restore --staged filename.py

# Discard changes in file
git restore filename.py

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# View history
git log --oneline
```

### Pulling Latest Changes

```bash
# Pull main branch updates
git pull origin main

# Fetch without merging
git fetch origin

# Rebase your branch on main
git rebase origin/main
```

---

## Commit Message Conventions

Use the following format for commit messages:

```
<type>: <short description>

<optional longer description>

Closes #<issue-number> (if applicable)
```

### Types

- **feat**: A new feature
- **fix**: A bug fix
- **docs**: Documentation only changes
- **refactor**: Code change that neither fixes a bug nor adds a feature
- **test**: Adding missing tests or correcting existing tests
- **style**: Changes that do not affect the meaning of the code (formatting, etc.)
- **chore**: Changes to build process, dependencies, or other maintenance

### Examples

```bash
# Feature addition
git commit -m "feat: add CORS middleware to FastAPI application"

# Bug fix
git commit -m "fix: resolve database connection timeout in Django"

# Documentation
git commit -m "docs: update README with installation instructions"

# Multiple file changes
git commit -m "refactor: reorganize Flask app structure for better scalability"

# With issue reference
git commit -m "fix: resolve pagination bug in item listing

Fixes #42"
```

---

## Pull Request Workflow

### Creating a Pull Request

1. **Push your branch to GitHub**
   ```bash
   git push -u origin feature/my-feature
   ```

2. **Go to GitHub and create PR**
   - Navigate to your repository
   - Click "Pull requests" tab
   - Click "New pull request"
   - Select your branch
   - Fill in title and description
   - Click "Create pull request"

### PR Description Template

```markdown
## Description
Brief description of what this PR does

## Type of Change
- [ ] New feature
- [ ] Bug fix
- [ ] Breaking change
- [ ] Documentation update

## Related Issue
Closes #<issue-number>

## Testing
- [ ] I have tested these changes
- [ ] All tests pass

## Checklist
- [ ] Code follows style guidelines
- [ ] I have updated documentation
- [ ] I have added tests for new features
```

### Merging a Pull Request

1. **Ensure all checks pass**
   - Wait for CI/CD to complete
   - Resolve any failing tests

2. **Get approval from reviewer**
   - Request review from team members

3. **Merge the PR**
   - Click "Merge pull request"
   - Choose merge strategy (usually "Create a merge commit")
   - Confirm merge

4. **Delete branch**
   - Delete remote branch after merge

---

## Local Development Workflow

### Daily Workflow

```bash
# Morning: Get latest changes
git fetch origin
git pull origin main

# Throughout the day: Make commits
git add <files>
git commit -m "<type>: <message>"

# Before end of day: Push to GitHub
git push origin feature/my-feature

# Create PR if not exists
# Make commits as needed
# Push updates
git add <files>
git commit -m "<type>: <message>"
git push origin feature/my-feature
```

### Handling Merge Conflicts

```bash
# When pulling and conflicts occur
git pull origin main

# View conflicts
git status

# Edit conflicted files
# Look for:
# <<<<<<< HEAD
# your changes
# =======
# their changes
# >>>>>>> branch-name

# After resolving
git add <resolved-files>
git commit -m "fix: resolve merge conflicts"
git push origin feature/my-feature
```

---

## Repository Organization Tips

### Directory Naming
- Use lowercase with hyphens: `fastapi-app`, `django-app`, `flask-app`
- Use descriptive names: `terraform`, `grafana`, `docs`

### File Naming
- Python files: `snake_case.py`
- Configuration: `.env`, `.gitignore`, `pytest.ini`
- Documentation: `README.md`, `SETUP.md`

### Keep .gitignore Updated

Common Python patterns to ignore:
```
__pycache__/
*.pyc
*.pyo
*.egg-info/
.venv/
venv/
.env
.pytest_cache/
.coverage
htmlcov/
*.db
*.sqlite
```

---

## GitHub Repository Settings

### Recommended Settings

1. **Branch Protection**
   - Go to Settings → Branches
   - Add rule for `main` branch
   - Require pull request reviews
   - Require status checks to pass

2. **Webhooks**
   - Set up CI/CD pipeline
   - Configure automated tests

3. **Topics**
   - Add: `terraform`, `grafana`, `fastapi`, `django`, `flask`, `python`

4. **Description**
   - Add meaningful repository description
   - Include link to documentation

---

## Useful GitHub Features

### Labels for Issues

Create labels for better organization:
- `documentation` - Documentation improvements
- `enhancement` - New feature request
- `bug` - Bug report
- `help-wanted` - Requesting help
- `in-progress` - Currently being worked on
- `testing` - Needs testing

### Templates

Create issue and PR templates:
1. Go to Settings → Options
2. Click "Set up templates"
3. Create issue template
4. Create pull request template

### Milestones

Organize issues by milestones:
1. Go to Issues → Milestones
2. Create milestone (e.g., "v1.0")
3. Associate issues with milestone
4. Track progress

---

## Best Practices Summary

✅ **Commit Often** - Small, logical commits are better than large ones  
✅ **Clear Messages** - Write descriptive commit messages  
✅ **Pull Requests** - Use PRs for code review and discussion  
✅ **Keep Main Stable** - Ensure `main` branch is always working  
✅ **Document Changes** - Update README and docs with changes  
✅ **Test Before Push** - Run tests locally before pushing  
✅ **Review Code** - Review other's PRs and ask for reviews  
✅ **Use Branches** - Create branches for features and fixes  
✅ **Descriptive Names** - Use clear names for branches  
✅ **Keep Organized** - Maintain consistent structure  

---

## Quick Reference

```bash
# Setup
git clone <repo-url>
cd <repo>
git config user.email "email@example.com"
git config user.name "Your Name"

# Feature development
git checkout -b feature/name
# ... make changes ...
git add .
git commit -m "feat: description"
git push -u origin feature/name

# Pull latest
git checkout main
git pull origin main

# Update feature branch with latest main
git fetch origin
git rebase origin/main

# Clean up
git branch -D local-branch-name
git push origin --delete remote-branch-name

# View history
git log --oneline --graph --all

# Check what changed
git diff main...feature/name
```

---

This guide ensures your repository is well-organized and follows professional Git practices!
