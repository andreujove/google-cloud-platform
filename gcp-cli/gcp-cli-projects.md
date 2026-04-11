# GCP Projects Management

Complete guide to managing Google Cloud Platform projects using the `gcloud` CLI.

## Listing and Viewing Projects

### List All Projects
```bash
# List all projects you have access to
gcloud projects list

# List in JSON format
gcloud projects list --format=json

# List with specific columns
gcloud projects list --format="table(projectId,name,projectNumber)"

# Filter by name
gcloud projects list --filter="name:my-project"

# Filter by status
gcloud projects list --filter="lifecycleState:ACTIVE"

# Limit results
gcloud projects list --limit=10
```

### Get Project Details
```bash
# Get current project
gcloud config get-value project

# Get project info
gcloud projects describe PROJECT_ID

# Get specific fields
gcloud projects describe PROJECT_ID --format="value(projectNumber)"
gcloud projects describe PROJECT_ID --format="value(name)"
```

### Display Project Information
```bash
# View all project details in JSON
gcloud projects describe PROJECT_ID --format=json

# View in YAML
gcloud projects describe PROJECT_ID --format=yaml

# View as table
gcloud projects describe PROJECT_ID --format=table
```

## Setting Active Project

### Set Default Project
```bash
# Set active project
gcloud config set project PROJECT_ID

# Verify current project
gcloud config list project

# Or get current project
gcloud config get-value project
```

### Temporary Project Override
```bash
# Use project flag to override default (no permanent change)
gcloud compute instances list --project=OTHER_PROJECT_ID

# Can be used with any command
gcloud container clusters list --project=OTHER_PROJECT_ID
```

## Creating Projects

### Create New Project
```bash
# Create project
gcloud projects create PROJECT_ID \
  --name="Project Display Name"

# Create with organization ID
gcloud projects create PROJECT_ID \
  --name="Project Name" \
  --organization-id=ORG_ID

# Create with folder ID
gcloud projects create PROJECT_ID \
  --name="Project Name" \
  --folder-id=FOLDER_ID
```

### Verify Project Creation
```bash
# List recent projects
gcloud projects list --limit=5 --sort-by=~createTime

# Describe new project
gcloud projects describe PROJECT_ID
```

## Project Configuration

### Link to Billing Account
```bash
# List billing accounts available to you
gcloud billing accounts list

# Link project to billing account
gcloud billing projects link PROJECT_ID \
  --billing-account=BILLING_ACCOUNT_ID

# Verify billing is enabled
gcloud billing projects describe PROJECT_ID
```

### Update Project Metadata
```bash
# Update project name
gcloud projects update PROJECT_ID \
  --name="New Project Name"

# Add labels
gcloud projects update PROJECT_ID \
  --update-labels=environment=prod,team=backend

# Clear labels
gcloud projects update PROJECT_ID \
  --remove-labels=old-label

# View labels
gcloud projects describe PROJECT_ID --format="value(labels)"
```

## Project Environment

### Quick Project Setup
```bash
# After creating a project:

# 1. Set as active project
gcloud config set project PROJECT_ID

# 2. Set default zone/region
gcloud config set compute/zone us-west1-a
gcloud config set compute/region us-west1

# 3. Link to billing (if not already done)
gcloud billing projects link PROJECT_ID --billing-account=BILLING_ACCOUNT_ID

# 4. Verify setup
gcloud config list
```

## Working with Multiple Projects

### Using Configuration Profiles
```bash
# Create configuration for each project
gcloud config configurations create prod
gcloud config set project prod-project-id
gcloud config set compute/zone us-west1-a

gcloud config configurations create dev
gcloud config set project dev-project-id
gcloud config set compute/zone us-central1-a

# Switch between projects
gcloud config configurations activate prod
gcloud config configurations activate dev

# List configurations
gcloud config configurations list
```

### Per-Command Project Override
```bash
# Run command in specific project without changing default
gcloud compute instances list --project=OTHER_PROJECT_ID

# Works with all gcloud commands
gcloud container clusters list --project=OTHER_PROJECT_ID
gcloud storage buckets list --project=OTHER_PROJECT_ID
```

## Project Hierarchies

### Preview: Organization and Folder Structure
```bash
# List organizations (if you have access)
gcloud organizations list

# List folders in organization
gcloud resource-manager folders list --organization-id=ORG_ID

# View resource hierarchy
gcloud projects get-ancestors PROJECT_ID
```

### Move Project to Folder
```bash
# Move existing project to folder
gcloud projects move PROJECT_ID \
  --organization-id=ORG_ID

# Move to specific folder
gcloud projects move PROJECT_ID \
  --folder-id=FOLDER_ID
```

## Project Operations

### List Enabled APIs
```bash
# List all enabled services in project
gcloud services list

# List available services (not just enabled)
gcloud services list --available

# Check if specific service is enabled
gcloud services list --filter="name:*.googleapis.com" \
  --format="table(name,state)"
```

### Enable/Disable Services
```bash
# Enable API
gcloud services enable compute.googleapis.com

# Disable API
gcloud services disable storage.googleapis.com

# Enable multiple APIs
gcloud services enable \
  compute.googleapis.com \
  container.googleapis.com \
  storage.googleapis.com
```

## Quotas and Limits

### Check Project Quotas
```bash
# Describe quota info for a project
gcloud compute project-info describe --project=PROJECT_ID

# View specific quota details
gcloud compute project-info describe PROJECT_ID \
  --format="table(quotas[].metric,quotas[].limit,quotas[].usage)"
```

## Troubleshooting

### Issue: "Project not found"
**Solution:**
```bash
# Verify project exists and you have access
gcloud projects list

# Check project ID format (should be lowercase with hyphens)
# Format: [a-z0-9-]{6,30}
```

### Issue: "Permission denied"
**Solution:**
```bash
# Verify you're authenticated
gcloud auth list

# Check your IAM role in the project
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:YOUR_EMAIL"
```

### Issue: "Billing account not found"
**Solution:**
```bash
# List available billing accounts
gcloud billing accounts list

# Ensure billing account ID is correct
# Format: 0X0X0X-0X0X0X-0X0X0X
```

## Related Documentation

- [Authentication](gcp-cli-authentication.md) - Login and manage credentials
- [IAM](gcp-cli-iam.md) - Manage project permissions and roles
- [Configuration Management](gcp-cli-configuration.md) - Configure gcloud settings
