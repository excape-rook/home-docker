# GitHub Setup Instructions

This document explains how to push the Homepage dashboard setup to GitHub and create a PR.

## Prerequisites

- GitHub account (robinsuter)
- GitHub CLI (`gh`) installed and authenticated
- Git configured locally

## Step 1: Authenticate with GitHub

```bash
gh auth login

# Select: GitHub.com
# Select: HTTPS
# When asked about credentials, select: Paste an authentication token
# Go to https://github.com/settings/tokens and create a fine-grained personal access token
# Or use existing token
```

## Step 2: Create Repository on GitHub

Create the public `home-docker` repository:

```bash
# Option A: Using GitHub CLI (recommended)
cd /root/.openclaw/workspace/home-docker
gh repo create robinsuter/home-docker \
  --public \
  --source=. \
  --remote=origin \
  --description "Docker Compose setup for Homepage dashboard on Robin's homelab"

# Option B: Manual - Create on GitHub.com first, then:
# Add remote manually:
git remote add origin https://github.com/robinsuter/home-docker.git
```

## Step 3: Push Branches to GitHub

```bash
cd /root/.openclaw/workspace/home-docker

# Push main branch
git push -u origin main

# Push feature branch
git push -u origin feature/homepage-dashboard
```

## Step 4: Create Pull Request

```bash
# Create PR using GitHub CLI
gh pr create \
  --base main \
  --head feature/homepage-dashboard \
  --title "feat: Add Homepage dashboard Docker Compose setup" \
  --body-file PR_TEMPLATE.md \
  --draft false

# Or create interactively:
gh pr create
# Follow prompts:
# - Title: "feat: Add Homepage dashboard Docker Compose setup"
# - Body: Copy from PR_TEMPLATE.md
# - Draft: No
```

## Step 5: Verify PR

```bash
# List PRs
gh pr list --repo robinsuter/home-docker

# View specific PR
gh pr view 1 --repo robinsuter/home-docker

# Check status
gh pr checks 1 --repo robinsuter/home-docker
```

## Complete Sequence

```bash
cd /root/.openclaw/workspace/home-docker

# 1. Create repo on GitHub
gh auth login
gh repo create robinsuter/home-docker --public --source=. --remote=origin --description "Docker Compose setup for Homepage dashboard on Robin's homelab"

# 2. Verify remotes
git remote -v

# 3. Push both branches
git push -u origin main
git push -u origin feature/homepage-dashboard

# 4. Create PR
gh pr create \
  --base main \
  --head feature/homepage-dashboard \
  --title "feat: Add Homepage dashboard Docker Compose setup" \
  --body "$(cat PR_TEMPLATE.md)"

# 5. View PR
gh pr view 1 --repo robinsuter/home-docker --web
```

## GitHub Actions

Once pushed, the validation workflow will automatically run:

```bash
# Monitor workflow status
gh run list --repo robinsuter/home-docker

# View workflow details
gh run view <RUN_ID> --repo robinsuter/home-docker
```

The workflow validates:
- ✅ docker-compose.yml syntax
- ✅ YAML file validity
- ✅ Required files presence
- ✅ Configuration syntax

## PR Review & Merge

Once PR is created:

1. **Review Changes**:
   - Check file structure
   - Review configurations
   - Verify documentation

2. **Run Automated Tests**:
   - GitHub Actions workflow validates configs
   - Review test results

3. **Manual Testing** (recommended):
   ```bash
   # Test deployment on homelab
   cd home-docker
   docker-compose config  # Validate config
   docker-compose up -d   # Start service
   docker-compose logs    # Check logs
   ```

4. **Merge to Main**:
   ```bash
   # When ready to merge:
   gh pr merge 1 \
     --merge \
     --delete-branch \
     --repo robinsuter/home-docker
   ```

## Troubleshooting

### Can't authenticate with GitHub
```bash
# Re-authenticate
gh auth login --reset

# Or use personal access token:
gh auth login --with-token < token.txt
```

### Remote URL incorrect
```bash
# Check current remote
git remote -v

# Update if needed
git remote set-url origin https://github.com/robinsuter/home-docker.git
```

### Branch already exists on GitHub
```bash
# Force push (use with caution!)
git push -u origin feature/homepage-dashboard --force
```

### Workflow failing
```bash
# Check workflow logs
gh run list --repo robinsuter/home-docker --status failure

# View specific run
gh run view <RUN_ID> --log --repo robinsuter/home-docker
```

## After PR is Merged

1. **Delete feature branch**:
   ```bash
   git branch -d feature/homepage-dashboard
   git push origin --delete feature/homepage-dashboard
   ```

2. **Update local main**:
   ```bash
   git checkout main
   git pull origin main
   ```

3. **Deploy to homelab**:
   ```bash
   cd /path/to/homelab/home-docker
   git pull origin main
   cp .env.example .env
   # Edit .env with your values
   docker-compose up -d
   ```

## Using the Repository

Once pushed to GitHub, anyone can clone it:

```bash
git clone https://github.com/robinsuter/home-docker.git
cd home-docker
cp .env.example .env
# Configure and deploy
docker-compose up -d
```

## Repository Settings (Optional)

Configure on GitHub.com for better workflows:

1. **Branch Protection**:
   - Main branch: Require PR reviews
   - Require status checks to pass
   - Require branches to be up to date

2. **Actions**:
   - Enable GitHub Actions
   - Set workflow permissions

3. **Collaborators**:
   - Add team members if sharing
   - Set appropriate permissions

## References

- [GitHub CLI Documentation](https://cli.github.com/)
- [gh pr create](https://cli.github.com/manual/gh_pr_create)
- [gh pr merge](https://cli.github.com/manual/gh_pr_merge)
- [GitHub Actions](https://docs.github.com/en/actions)

---

**Status**: Ready to push to GitHub  
**Next Step**: Run the complete sequence above to create repository and PR
