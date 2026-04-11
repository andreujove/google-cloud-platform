# GCP CLI Authentication

Complete guide to authenticating with Google Cloud Platform using the `gcloud` CLI.

## Authentication Methods

### 1. User Authentication (Interactive Login)

#### Login to Your Google Account
```bash
# Interactive login - opens browser
gcloud auth login

# Specify an email
gcloud auth login EMAIL@gmail.com
```

#### Check Authentication Status
```bash
# List all authenticated accounts
gcloud auth list

# Show active account (marked with *)
gcloud auth list --filter=status:ACTIVE

# Get details about current account
gcloud config get-value account
```

#### Logout
```bash
# Logout from specific account
gcloud auth revoke EMAIL@gmail.com

# Logout from all accounts
gcloud auth revoke --all
```

### 2. Service Account Authentication (Programmatic Access)

#### Create a Service Account
```bash
# Create service account
gcloud iam service-accounts create SA_NAME \
  --display-name="Service Account Display Name"

# List service accounts
gcloud iam service-accounts list

# Get details of a service account
gcloud iam service-accounts describe SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

#### Generate Service Account Keys

**Option A: JSON Key (for local development)**
```bash
# Generate JSON key
gcloud iam service-accounts keys create KEY_FILE.json \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# This creates a key file with credentials
# Location: ./KEY_FILE.json
```

**Option B: View Existing Keys**
```bash
# List all keys for a service account
gcloud iam service-accounts keys list \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Delete old keys
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

#### Authenticate with Service Account
```bash
# Activate service account
gcloud auth activate-service-account --key-file=KEY_FILE.json

# Verify activation
gcloud auth list
gcloud config get-value account
```

### 3. Application Default Credentials (ADC)

```bash
# Set up ADC for local development
gcloud auth application-default login

# This creates credentials at:
# - Linux/macOS: ~/.config/gcloud/application_default_credentials.json
# - Windows: %APPDATA%\gcloud\application_default_credentials.json

# Verify ADC is set
gcloud auth application-default print-access-token
```

**Use in applications:**
```bash
# Set environment variable
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json

# Or in code (Python example):
from google.cloud import storage
client = storage.Client()  # Uses ADC automatically
```

## Access Tokens and Credentials

### Get Access Tokens
```bash
# Print current access token
gcloud auth print-access-token

# Copy to clipboard (macOS)
gcloud auth print-access-token | pbcopy

# Copy to clipboard (Linux)
gcloud auth print-access-token | xclip -selection clipboard
```

### Get Identity Token
```bash
# Get identity token (for service-to-service auth)
gcloud auth print-identity-token

# Get identity token for specific service
gcloud auth print-identity-token --audiences=https://example.com
```

### Get Quota Project
```bash
# Get quota project ID
gcloud config get-value billing/quota_project
```

## Managing Multiple Accounts

### Switch Between Accounts
```bash
# View all accounts
gcloud auth list

# Set active account
gcloud config set account EMAIL@gmail.com

# Or use the interactive selector
gcloud auth application-default set-quota-project PROJECT_ID
```

### Configure Separate Profiles for Different Accounts
```bash
# Create configuration for work account
gcloud config configurations create work
gcloud config set account work@company.com
gcloud config set project work-project-id

# Create configuration for personal account
gcloud config configurations create personal
gcloud config set account personal@gmail.com
gcloud config set project personal-project-id

# Switch between configurations
gcloud config configurations activate work
gcloud config configurations activate personal

# List all configurations
gcloud config configurations list
```

## Security Best Practices

### 1. Never Commit Keys to Version Control
```bash
# Add to .gitignore
echo "*.json" >> .gitignore
echo "service-account-key.json" >> .gitignore
echo ".google" >> .gitignore
```

### 2. Use Workload Identity (GKE)
Instead of storing service account keys, use Workload Identity:
```bash
# Bind Kubernetes SA to GCP service account
gcloud iam service-accounts add-iam-policy-binding \
  SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/K8S_SA_NAME]"
```

### 3. Rotate Service Account Keys Regularly
```bash
# List keys older than 90 days
gcloud iam service-accounts keys list \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --format="table(name,validAfterTime)" \
  --filter="validAfterTime<-P90D"

# Delete old keys
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

### 4. Grant Minimal Permissions
```bash
# Grant specific role to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member="serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/container.developer"  # Use specific role, not roles/editor

# Verify permissions
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:SA_NAME*"
```

## Troubleshooting Authentication

### Issue: "Application Default Credentials are not available"
**Solution:**
```bash
# Set up ADC
gcloud auth application-default login

# Or set environment variable
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account-key.json
```

### Issue: "You do not currently have an active account"
**Solution:**
```bash
# Login again
gcloud auth login

# Or activate service account
gcloud auth activate-service-account --key-file=KEY_FILE.json
```

### Issue: "Permission denied" errors
**Solution:**
```bash
# Check current account
gcloud auth list

# Check IAM roles for account
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:EMAIL@gmail.com"

# Verify you have required permissions
gcloud projects get-iam-policy PROJECT_ID
```

### Issue: "Quota exceeded"
**Solution:**
```bash
# Check quota project
gcloud config get-value billing/quota_project

# Set quota project
gcloud config set billing/quota_project PROJECT_ID
```

## Related Documentation

- [Service Accounts](gcp-cli-iam.md) - Create and manage service accounts
- [Configuration Management](gcp-cli-configuration.md) - Configure gcloud settings
- [Best Practices](gcp-cli-best-practices.md) - Security best practices
