# Doc-Guardian Setup (Fork Version)

This version is designed for **personal forks** that create PRs to **rdkcentral upstream repos**.

## Single Token Setup

Since you have contributor access to rdkcentral, you only need **ONE token** that's scoped to both your fork and the upstream repo.

### Token Configuration

#### 1. Create Fine-Grained Personal Access Token

**Token Name:** `FORK_PAT`  
**Resource Owner:** `melhar098` (your account)  
**Repository Access:** Select both:
- `melhar098/entservices-usersettings` (your fork)
- `rdkcentral/entservices-usersettings` (upstream)

**Permissions:**
- Contents: Read and Write
- Pull requests: Read and Write  
- Issues: Read and Write

#### 2. Store Token in Two Places

**A. GitHub Repository Secret** (for workflow):
1. Go to fork: `github.com/melhar098/entservices-usersettings/settings/secrets/actions`
2. Click "New repository secret"
3. Name: `FORK_PAT`
4. Value: paste your token
5. Click "Add secret"

**B. Copilot Cloud Environment** (for agent):
1. Navigate to agent environment configuration
2. Add secret: `FORK_PAT`
3. Value: same token as above

## How It Works

### Workflow (`doc-guardian.yml`)
- Detects PRs with source changes but no doc updates
- Creates issue in your fork
- Assigns `@copilot` with `/agent doc-guardian`
- Uses `FORK_PAT` (repository secret) to assign agent

### Agent (`doc-guardian.agent.md`)
- Analyzes source changes
- Generates/updates documentation
- **Pushes commits to fork** using `FORK_PAT`
- **Creates PR in rdkcentral** using same `FORK_PAT`
- PR format: `melhar098:copilot/... → rdkcentral:develop`

## Testing

### Prerequisites
✅ Agent must be on **default branch** (`develop`) to be discoverable  
✅ `FORK_PAT` configured in repository secrets  
✅ `FORK_PAT` configured in Copilot environment  
✅ Token scoped to both fork and upstream repos

### Test Workflow
1. Create test PR with source changes (no doc changes)
2. Push to your fork
3. Workflow auto-triggers, creates issue
4. Agent auto-assigned
5. Agent analyzes, generates docs
6. Agent pushes to fork branch
7. Agent creates PR: `melhar098/entservices-usersettings → rdkcentral/entservices-usersettings`

### Verification
- Check issue created in fork
- Check agent assigned (`copilot-swe-agent[bot]`)
- Check PR created in **rdkcentral** (not fork)
- Check PR head: `melhar098:copilot/...`
- Check PR base: `develop` (in rdkcentral)

## Troubleshooting

### "FORK_PAT is empty"
- Add `FORK_PAT` repository secret with token value
- Verify token in Settings → Secrets → Actions

### "403 Forbidden" when assigning agent
- Token needs `Issues: Write` permission
- Verify token is scoped to your fork

### "403 Forbidden" when creating upstream PR
- Token needs write access to `rdkcentral/entservices-usersettings`
- Verify token includes upstream repo in scope
- Verify you have contributor access to rdkcentral

### Agent not appearing in dropdown
- Agent file must be on default branch (`develop`)
- Commit agent file to develop and wait ~5 minutes for GitHub to index

### PR created in fork instead of rdkcentral
- Check agent logs for `$UPSTREAM_REPO` detection
- Verify `gh pr create --repo "$UPSTREAM_REPO"` is used
- Check PR URL contains `rdkcentral`, not `melhar098`

## Token Scope Explanation

**Why one token works:**
- You're a rdkcentral contributor
- Fine-grained tokens can be scoped to multiple repos
- `FORK_PAT` has access to **both** fork and upstream
- Same token used for: issue assignment, fork push, upstream PR

**Alternative (if you weren't a contributor):**
- Would need two tokens
- `FORK_PAT`: melhar098-owned, fork access only
- `COPILOT_PAT`: rdkcentral-owned, upstream access only

## Next Steps

1. ✅ Commit this setup to `develop` branch
2. ✅ Configure `FORK_PAT` token with dual scope
3. ✅ Add token to repository secrets
4. ✅ Add token to Copilot environment
5. 🧪 Test with sample PR
6. 🚀 Roll out to other repos if successful
