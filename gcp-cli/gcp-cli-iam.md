# GCP IAM (Identity and Access Management)

Complete guide to managing Identity and Access Management using the `gcloud` CLI.

## Understanding IAM Basics

### IAM Concepts
- **Principal**: User, service account, or group that can take actions
- **Role**: Collection of permissions
- **Permission**: Ability to perform a specific action
- **Policy**: Binding of principals to roles on resources

## Viewing IAM Policies

### Project-Level Policies
```bash
# Get project IAM policy
gcloud projects get-iam-policy PROJECT_ID

# Get in JSON format
gcloud projects get-iam-policy PROJECT_ID --format=json

# Show specific member
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:EMAIL@gmail.com"

# Show all service accounts
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:serviceAccount:*"

# Show members with specific role
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.role:roles/viewer"
```

### Resource-Level Policies
```bash
# Get IAM policy for GKE cluster
gcloud container clusters get-iam-policy CLUSTER_NAME --zone=ZONE

# Get IAM policy for storage bucket
gsutil iam get gs://BUCKET_NAME

# Get IAM policy for compute instance (via resource manager)
gcloud resource-manager get-iam-policy RESOURCE_ID
```

## Managing Roles

### List Available Roles
```bash
# List all predefined roles
gcloud iam roles list

# List custom roles
gcloud iam roles list --show-custom=true

# Search for specific role
gcloud iam roles list --filter="name:*viewer*"

# Get role details
gcloud iam roles describe roles/viewer

# Show permissions in role
gcloud iam roles describe roles/viewer --format="value(includedPermissions[])"
```

### Role Categories
```bash
# Basic roles (legacy - not recommended)
roles/viewer          # Read-only
roles/editor          # Read and write
roles/owner           # Full control

# Predefined roles
roles/container.developer              # GKE developer
roles/container.admin                  # GKE admin
roles/compute.instanceAdmin.v1         # Compute admin
roles/storage.admin                    # Storage admin
roles/iam.serviceAccountAdmin          # Service account admin
roles/iam.securityAdmin                # Security admin

# Custom roles
# Create custom roles for specific needs
```

## Granting and Revoking Access

### Grant User Access
```bash
# Grant role to user
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:user@example.com \
  --role=roles/viewer

# Grant role to multiple users
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:user1@example.com \
  --member=user:user2@example.com \
  --role=roles/editor

# Grant role to group
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=group:developers@company.com \
  --role=roles/container.developer

# Grant role to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/container.admin
```

### Grant Resource-Level Access
```bash
# Grant role on specific resource (GKE cluster)
gcloud container clusters add-iam-policy-binding CLUSTER_NAME \
  --member=user:user@example.com \
  --role=roles/container.developer \
  --zone=us-west1-a

# Grant role on storage bucket
gsutil iam ch user:user@example.com:objectViewer gs://BUCKET_NAME

# Grant to service account on bucket
gsutil iam ch serviceAccount:SA@PROJECT.iam.gserviceaccount.com:objectAdmin gs://BUCKET_NAME
```

### Revoke Access
```bash
# Remove role from user
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member=user:user@example.com \
  --role=roles/viewer

# Remove multiple bindings
gcloud projects remove-iam-policy-binding PROJECT_ID \
  --member=user:user@example.com \
  --role=roles/viewer \
  --quiet

# Remove access from bucket
gsutil iam ch -d user:user@example.com gs://BUCKET_NAME
```

## Service Accounts

### Create Service Accounts
```bash
# Create service account
gcloud iam service-accounts create SA_NAME \
  --display-name="Service Account Display Name" \
  --description="Description of service account"

# List service accounts
gcloud iam service-accounts list

# Describe service account
gcloud iam service-accounts describe SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Get service account details
gcloud iam service-accounts describe SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --format=json
```

### Service Account Keys

#### Create Keys
```bash
# Create JSON key
gcloud iam service-accounts keys create KEY_FILE.json \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Create P12 key
gcloud iam service-accounts keys create KEY_FILE.p12 \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --key-file-type=p12
```

#### List Keys
```bash
# List all keys for service account
gcloud iam service-accounts keys list \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# List with format
gcloud iam service-accounts keys list \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --format="table(name,validAfterTime,validBeforeTime)"
```

#### Delete Keys
```bash
# Delete key
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Delete old keys (older than 90 days)
gcloud iam service-accounts keys list \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --format="value(name)" | xargs -I {} \
  gcloud iam service-accounts keys delete {} \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

### Grant Roles to Service Accounts
```bash
# Grant role to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/container.admin

# Grant multiple roles
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/container.developer

# Assign permissions for specific resource (GKE)
gcloud container clusters add-iam-policy-binding CLUSTER_NAME \
  --member=serviceAccount:SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/container.developer \
  --zone=us-west1-a
```

### Service Account Impersonation
```bash
# Allow user to impersonate service account
gcloud iam service-accounts add-iam-policy-binding \
  SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --member=user:user@example.com \
  --role=roles/iam.serviceAccountUser

# Impersonate service account
gcloud auth application-default print-access-token \
  --impersonate-service-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

## Custom Roles

### Create Custom Role
```bash
# Create custom role from file
gcloud iam roles create CUSTOM_ROLE_NAME \
  --project=PROJECT_ID \
  --title="Custom Role Title" \
  --description="Description of custom role" \
  --permissions=compute.instances.get,compute.instances.list

# Create from role file (YAML)
gcloud iam roles create CUSTOM_ROLE_NAME \
  --project=PROJECT_ID \
  --file=role.yaml
```

### Manage Custom Roles
```bash
# List custom roles
gcloud iam roles list --project=PROJECT_ID --show-custom=true

# Describe custom role
gcloud iam roles describe projects/PROJECT_ID/roles/CUSTOM_ROLE_NAME

# Update custom role
gcloud iam roles update CUSTOM_ROLE_NAME \
  --project=PROJECT_ID \
  --permissions=permission1,permission2

# Delete custom role
gcloud iam roles delete CUSTOM_ROLE_NAME --project=PROJECT_ID
```

## Testable Permissions

### Check Permissions
```bash
# Test if current user has permission
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:YOUR_EMAIL"

# Get list of testable permissions
gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com

# Filter testable permissions
gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com \
  --filter="*container*"
```

## Conditions and Attributes

### Time-Based Access (Preview)
```bash
# Grant access with time condition
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:user@example.com \
  --role=roles/compute.admin \
  --condition='resource.matchTag("env", "prod") && resource.matchTag("team", "backend")'
```

## Audit Logging

### Enable Audit Logs
```bash
# Get audit log configurations
gcloud projects get-iam-policy PROJECT_ID --format=json

# Update audit log retention
gcloud logging create-bucket BUCKET_NAME \
  --location=global \
  --retention-days=30
```

## Best Practices

### Principle of Least Privilege
```bash
# AVOID: Don't grant Editor or Owner roles
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:user@example.com \
  --role=roles/editor  # AVOID!

# DO: Grant specific, minimal roles
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:user@example.com \
  --role=roles/container.developer
```

### Service Account Best Practices
```bash
# Use unique service accounts per application
gcloud iam service-accounts create app-frontend-sa
gcloud iam service-accounts create app-backend-sa
gcloud iam service-accounts create app-database-sa

# Grant minimal permissions to each
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:app-backend-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/cloudsql.client

# Use short-lived credentials
# Avoid storing keys - use Workload Identity for GKE
```

## Troubleshooting

### Issue: "Permission denied" when granting access
**Solution:**
```bash
# Verify you have iam.serviceAccountAdmin role
gcloud projects get-iam-policy PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.members:user:YOUR_EMAIL"

# Request higher permissions or ask admin
```

### Issue: Service account key leak
**Solution:**
```bash
# List and delete leaked key
gcloud iam service-accounts keys list \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Delete compromised key
gcloud iam service-accounts keys delete LEAKED_KEY_ID \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com

# Create new key
gcloud iam service-accounts keys create new-key.json \
  --iam-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

## Related Documentation

- [Authentication](gcp-cli-authentication.md) - Service account authentication
- [Best Practices](gcp-cli-best-practices.md) - Security best practices
