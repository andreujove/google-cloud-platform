# GCP CLI Best Practices

Tips, tricks, and best practices for using the `gcloud` CLI effectively and securely.

## Output Formatting and Filtering

### Optimize Command Output

#### JSON Format
```bash
# Get JSON output for scripting
gcloud compute instances list --format=json

# Parse JSON with jq
gcloud compute instances list --format=json | jq '.[] | select(.name=="instance-1")'

# Extract specific fields
gcloud compute instances list --format=json | jq '.[].name'
```

#### Table Format
```bash
# Default human-readable format
gcloud compute instances list --format=table

# Custom table columns
gcloud compute instances list --format="table(name,zone,machineType.machine_type(),status)"

# Sort by column
gcloud compute instances list --format="table(name,zone)" --sort-by=zone
```

#### Value Format (Script-friendly)
```bash
# Get just the value (no headers)
gcloud compute instances list --format="value(name)"

# Multiple columns separated by spaces
gcloud compute instances list --format="value(name,zone)"

# Use in scripts
INSTANCE_NAME=$(gcloud compute instances list --format="value(name)" --limit=1)
echo "First instance: $INSTANCE_NAME"
```

### Filter Results

```bash
# Filter by status
gcloud compute instances list --filter="status:RUNNING"

# Filter by name pattern
gcloud compute instances list --filter="name:web-*"

# Complex filters
gcloud compute instances list --filter="(zone:us-west1-a) AND (status:RUNNING)"

# Filter by labels
gcloud compute instances list --filter="labels.environment:prod"

# Combine multiple conditions
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.role:roles/editor AND bindings.members:serviceAccount:*"
```

## Credential Management

### Best Practices for Credentials

#### Never Commit Keys to Version Control
```bash
# Add to .gitignore
echo "**/service-account-*.json" >> .gitignore
echo "*.p12" >> .gitignore
echo "key.json" >> .gitignore
echo ".gcloud/" >> .gitignore

# Verify keys aren't in git
git log --all --full-history -- "*.json" | head -20
```

#### Use Application Default Credentials (ADC)
```bash
# Set up ADC (recommended for development)
gcloud auth application-default login

# This creates credentials at:
# Linux/macOS: ~/.config/gcloud/application_default_credentials.json
# Windows: %APPDATA%\gcloud\application_default_credentials.json

# Most Google libraries use this automatically
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/credentials.json
```

#### Rotate Service Account Keys
```bash
# List keys older than 90 days
gcloud iam service-accounts keys list \
  --iam-account=SA@PROJECT.iam.gserviceaccount.com \
  --format="table(name,validAfterTime)"

# Delete old keys (one at a time!)
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=SA@PROJECT.iam.gserviceaccount.com

# Create new key
gcloud iam service-accounts keys create new-key.json \
  --iam-account=SA@PROJECT.iam.gserviceaccount.com

# Use new key
export GOOGLE_APPLICATION_CREDENTIALS=new-key.json
```

## Error Debugging

### Debug Mode
```bash
# Enable debug output
gcloud --debug compute instances list

# Verbose output
gcloud --verbosity=debug compute instances list

# Very verbose
gcloud --verbosity=debug compute instances list 2>&1 | tee debug.log
```

### Verify Permissions
```bash
# Check your role in project
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:YOUR_EMAIL"

# Check service account permissions
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:SA@PROJECT.iam.gserviceaccount.com"
```

## Scripting Tips

### Extract Values for Scripts
```bash
# Get project ID
PROJECT_ID=$(gcloud config get-value project)

# Get cluster endpoint
ENDPOINT=$(gcloud container clusters describe CLUSTER_NAME \
  --zone=ZONE --format="value(endpoint)")

# Get instance internal IP
INTERNAL_IP=$(gcloud compute instances describe INSTANCE_NAME \
  --zone=ZONE --format="value(networkInterfaces[0].networkIP)")

# Get first instance name
FIRST_INSTANCE=$(gcloud compute instances list --limit=1 --format="value(name)")

# Use in commands
gcloud container clusters get-credentials $CLUSTER_NAME --zone=$ZONE
```

### Batch Operations
```bash
# Process multiple resources
for instance in $(gcloud compute instances list --format="value(name)"); do
  gcloud compute instances describe $instance --zone=us-west1-a
done

# Parallel processing with xargs
gcloud compute instances list --format="value(name)" | xargs -P 4 -I {} \
  gcloud compute instances describe {} --zone=us-west1-a

# Stop all instances matching pattern
gcloud compute instances list --filter="name:test-*" --format="value(name)" | xargs -I {} \
  gcloud compute instances stop {} --zone=us-west1-a
```

### Error Handling
```bash
# Exit on error
set -e

# Check command success
if gcloud container clusters create $CLUSTER_NAME --zone=$ZONE; then
  echo "Cluster created successfully"
else
  echo "Failed to create cluster"
  exit 1
fi

# Try command, continue on error
gcloud compute instances delete $INSTANCE || echo "Instance not found"

# Retry logic
MAX_RETRIES=3
for ((i=1; i<=MAX_RETRIES; i++)); do
  if gcloud compute instances create $INSTANCE_NAME; then
    break
  fi
  if [ $i -lt $MAX_RETRIES ]; then
    sleep 5
  fi
done
```

## Performance Optimization

### Parallel Operations
```bash
# Enable parallel uploads/downloads
gsutil -m cp -r LOCAL_DIR gs://BUCKET_NAME

# Set parallel thread count
gsutil -m -o GSUtil:parallel_thread_count=24 cp -r LOCAL_DIR gs://BUCKET_NAME

# For gcloud operations, use background processes
for i in {1..5}; do
  gcloud compute instances create instance-$i &
done
wait  # Wait for all background jobs to finish
```

### Caching and Avoiding Redundant Operations
```bash
# Cache results to avoid repeated calls
INSTANCES=$(gcloud compute instances list --format=json)

# Reuse cached results
echo $INSTANCES | jq '.[] | select(.status=="RUNNING")'
echo $INSTANCES | jq '.[] | select(.zone=="us-west1-a")'

# Compare resources before making changes
BEFORE=$(gcloud compute instances list --format=json)
# ... make changes ...
AFTER=$(gcloud compute instances list --format=json)
diff <(echo $BEFORE | jq sort) <(echo $AFTER | jq sort)
```

## Security Best Practices

### Command-Line Argument Safety
```bash
# Avoid passing credentials on command line
# AVOID: gcloud auth login --email=user@example.com PASSWORD

# Use interactive login instead
gcloud auth login

# Never pass secrets on command line
# AVOID: gcloud compute instances create instance --metadata=api_key=secret123

# Use secret management (Cloud Secret Manager)
SECRET=$(gcloud secrets versions access latest --secret=api-key)
gcloud compute instances create instance --metadata=api_key_secret_ref=$SECRET
```

### Audit and Compliance
```bash
# Generate audit logs
gcloud logging read "resource.type=gke_cluster AND severity=ERROR" \
  --limit=50 \
  --format=json

# Monitor for security events
gcloud logging read "resource.type=gce_instance AND\
  protoPayload.methodName=compute.instances.delete" \
  --format=json

# Export logs for analysis
gcloud logging read "resource.type=k8s_cluster" \
  --format=json > audit-logs.json
```

## Shell Integration

### Shell Completion
```bash
# Bash completion (add to ~/.bashrc)
source <(gcloud completion bash)

# Zsh completion (add to ~/.zshrc)
gcloud completion zsh | sudo tee /usr/share/zsh/site-functions/_gcloud
source /usr/share/zsh/site-functions/_gcloud

# Fish completion (add to ~/.config/fish/completions)
gcloud completion fish | source
```

### Aliases for Common Operations
```bash
# Add to ~/.bashrc or ~/.zshrc
alias gcplist="gcloud compute instances list"
alias gkelist="gcloud container clusters list"
alias gcpregions="gcloud compute regions list"
alias gcpzones="gcloud compute zones list"
alias gcslist="gsutil ls"
alias gcpinit="gcloud init"
alias gcpauth="gcloud auth login"
alias gcpproject="gcloud config set project"
alias gcpzone="gcloud config set compute/zone"
alias gcpinfo="gcloud config list"

# Useful functions
function gcp-switch() {
  gcloud config configurations activate $1
  echo "Switched to $(gcloud config get-value project)"
}

function gcp-ssh() {
  gcloud compute ssh $1 --zone=$(gcloud config get-value compute/zone)
}
```

## Useful Command Patterns

### Health Check
```bash
# Check gcloud setup
gcloud --version
gcloud auth list
gcloud config list
gcloud projects list

# Quick health check function
gcp-health() {
  echo "=== gcloud Version ==="
  gcloud --version
  echo -e "\n=== Authentication ==="
  gcloud auth list
  echo -e "\n=== Project Configuration ==="
  gcloud config list core/project
  echo -e "\n=== Available Resources ==="
  gcloud compute instances list --limit=1 && echo "Compute instances: OK" || echo "Compute instances: FAILED"
  gcloud container clusters list --limit=1 && echo "GKE clusters: OK" || echo "GKE clusters: FAILED"
  gsutil ls --max-items=1 && echo "Cloud Storage: OK" || echo "Cloud Storage: FAILED"
}
```

### Resource Cleanup
```bash
# Delete all instances with pattern
gcloud compute instances list --filter="name:temp-*" --format="value(name)" | xargs -I {} \
  gcloud compute instances delete {} --quiet

# Clean up old disks
gcloud compute disks list --filter="name:old-*" --format="value(name)" | xargs -I {} \
  gcloud compute disks delete {} --quiet

# Remove unused networks
gcloud compute networks list --format="value(name)" | while read net; do
  [ "$net" != "default" ] && gcloud compute networks delete $net --quiet || true
done
```

### Troubleshooting Commands
```bash
# Check service status
gcloud service-usage services list --filter="state:ENABLED"

# Check quotas
gcloud compute project-info describe --project=$(gcloud config get-value project) \
  --format="table(quotas[].metric,quotas[].limit,quotas[].usage)"

# Check recent operations
gcloud compute operations list

# Get detailed error information
gcloud compute instances create test --zone=invalid-zone --verbosity=debug 2>&1 | tail -50
```

## Remember

- **Always verify the correct project/zone before operations** ✓
- **Use `--quiet` flag carefully** - suppresses confirmation prompts
- **Test commands with `--format=json` first** before scripting
- **Keep credentials secure** - never commit keys to version control
- **Document your configurations** - use comments and aliases
- **Use configurations/profiles** to manage multiple environments
- **Enable debug mode** when troubleshooting issues

## Related Documentation

- [Installation](gcp-cli-installation.md) - Setup guide
- [Configuration](gcp-cli-configuration.md) - Configuration management
- [Authentication](gcp-cli-authentication.md) - Credential security
