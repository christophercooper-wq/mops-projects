# GIT COMMANDS: PUSH TO REPOSITORY

**Repository**: christophercooper-wq/mops-projects (existing, public)
**Branch**: main
**Local Path**: `/Users/christopher.cooper/Documents/MOps/Assessment/`

---

## STEP 1: INITIALIZE GIT IN ASSESSMENT FOLDER

```bash
cd /Users/christopher.cooper/Documents/MOps/Assessment

# Initialize git if not already done
git init

# Add remote (your existing repo)
git remote add origin https://github.com/christophercooper-wq/mops-projects.git

# Verify remote
git remote -v
```

---

## STEP 2: CREATE .gitignore

```bash
# Create .gitignore file
cat > .gitignore << 'EOF'
# macOS
.DS_Store
.AppleDouble
.LSOverride

# Thumbnails
._*

# Files that might appear in the root of a volume
.DocumentRevisions-V100
.fseventsd
.Spotlight-V100
.TemporaryItems
.Trashes
.VolumeIcon.icns
.com.apple.timemachine.donotpresent

# Directories potentially created on remote AFP share
.AppleDB
.AppleDesktop
Network Trash Folder
Temporary Items
.apdisk

# Editor files
*.swp
*.swo
*~
.vscode/
.idea/

# Temp files
*.tmp
*.temp
EOF
```

---

## STEP 3: STAGE FILES

```bash
# Add all markdown files
git add *.md

# Add specific files (already created)
git add README.md
git add CHANGELOG.md
git add 01-root-cause-analysis.md
git add 09-project-roadmap.md
git add GIT_COMMANDS.md

# Check what's staged
git status
```

---

## STEP 4: COMMIT

```bash
# Initial commit
git commit -m "docs: Initialize MOps Infrastructure Overhaul Project

- Add README.md (master index with navigation, status dashboard)
- Add CHANGELOG.md (comprehensive project context and logging protocol)
- Add 01-root-cause-analysis.md (data architecture failure analysis)
- Add 09-project-roadmap.md (8-week timeline with Mermaid Gantt, GitHub Projects structure)
- Document $32K Skydog Ops 3-phase engagement
- Establish Git workflow and Google Drive sync preparation

Project Status: Week 1 Foundation Phase
Budget: $32K approved
Timeline: 8 weeks (Dec 2025 - Jan 2026)
Goal: Infrastructure readiness for Q1 2026 Demand Gen hire"
```

---

## STEP 5: PUSH TO GITHUB

```bash
# If repo is empty (first push)
git branch -M main
git push -u origin main

# If repo has existing content (pull first)
git pull origin main --allow-unrelated-histories
git push -u origin main

# Verify push succeeded
git log --oneline -5
```

---

## STEP 6: VERIFY ON GITHUB

Open in browser: https://github.com/christophercooper-wq/mops-projects

Should see:
- README.md rendered as homepage
- All .md files in file tree
- Mermaid Gantt chart rendering (if GitHub supports it in your repo settings)

---

## ONGOING WORKFLOW

### Adding New Files

```bash
# Create/edit file (done via Claude Code or your editor)
# Then:
cd /Users/christopher.cooper/Documents/MOps/Assessment
git add [filename].md
git commit -m "docs: [Brief description]"
git push origin main
```

### Updating Existing Files

```bash
git add [filename].md
git commit -m "docs: Update [filename] - [what changed]"
git push origin main
```

### Pulling Changes (if editing from another machine or Google Drive sync)

```bash
git pull origin main
```

---

## GOOGLE DRIVE SYNC SETUP (NEXT STEP)

### Option 1: GitHub Actions (Recommended - Automated)

Create `.github/workflows/sync-to-drive.yml`:

```yaml
name: Sync to Google Drive

on:
  push:
    branches: [ main ]
    paths:
      - '**.md'

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Convert Markdown to Google Docs
        uses: google-github-actions/auth@v1
        with:
          credentials_json: ${{ secrets.GCP_CREDENTIALS }}

      - name: Upload to Drive
        run: |
          # Script to upload .md files to Google Drive
          # (requires gdrive CLI tool or rclone)
          echo "Sync to Drive folder: /My Drive/MOps Infrastructure Project/"
```

**Setup Requirements**:
1. Create Google Cloud Project
2. Enable Google Drive API
3. Create Service Account credentials
4. Add credentials as GitHub Secret: `GCP_CREDENTIALS`
5. Share Drive folder with service account email

### Option 2: Manual Sync (Simple - For Now)

```bash
# Copy files to Google Drive local mount
cp *.md "/Users/christopher.cooper/Library/CloudStorage/GoogleDrive-christopher.cooper@rho.co/My Drive/MOps Infrastructure Project/"

# Google Drive will auto-sync to cloud
# Can convert to Google Docs manually in Drive UI
```

### Option 3: Rclone (Semi-Automated)

```bash
# Install rclone
brew install rclone

# Configure Google Drive remote
rclone config

# Sync command
rclone sync /Users/christopher.cooper/Documents/MOps/Assessment/ \
  gdrive:/MOps Infrastructure Project/ \
  --drive-import-formats md=application/vnd.google-apps.document \
  --include "*.md"
```

---

## GITHUB PROJECTS BOARD SETUP

### Via GitHub Web UI

1. Go to: https://github.com/christophercooper-wq/mops-projects
2. Click "Projects" tab
3. Click "New Project"
4. Select "Board" view
5. Name: "MOps Infrastructure Overhaul - 8 Week Timeline"
6. Create columns:
   - 📋 Backlog
   - 🔴 Blocked
   - 🏃 In Progress (WIP limit: 5)
   - 👀 Review
   - ✅ Done

### Create Issues from Roadmap

For each task in `09-project-roadmap.md`, create GitHub Issue:

```bash
# Example: Week 1 Task
gh issue create \
  --title "W1-P0-1: Request Leadership Alignment Meeting" \
  --body "See 09-project-roadmap.md for details" \
  --label "priority:p0-critical,phase:week-1-2" \
  --assignee christophercooper-wq
```

Or use GitHub web UI to manually create ~50 issues from roadmap tasks.

---

## TROUBLESHOOTING

### Problem: "Remote already exists"

```bash
git remote remove origin
git remote add origin https://github.com/christophercooper-wq/mops-projects.git
```

### Problem: "Failed to push - rejected"

```bash
# Pull first, then push
git pull origin main --allow-unrelated-histories
git push origin main
```

### Problem: "Authentication failed"

```bash
# Use GitHub Personal Access Token
# Go to: https://github.com/settings/tokens
# Generate new token with 'repo' scope
# Use token as password when prompted
```

### Problem: ".DS_Store keeps appearing"

```bash
# Remove from git tracking
find . -name .DS_Store -print0 | xargs -0 git rm -f --ignore-unmatch

# Add to .gitignore (already done above)
git add .gitignore
git commit -m "chore: Ignore .DS_Store files"
git push origin main
```

---

## NEXT STEPS AFTER PUSH

1. ✅ Verify files on GitHub: https://github.com/christophercooper-wq/mops-projects
2. ✅ Set up GitHub Projects board (create issues from roadmap)
3. ✅ Configure Google Drive sync (Option 2 manual for now, Option 1 later)
4. ✅ Continue creating remaining documentation files (02-08, 10-12)
5. ✅ Update CHANGELOG.md with each new file created

---

**Last Updated**: 2025-11-12
**Status**: Ready to execute
**Next Action**: Run Step 1 commands in terminal
