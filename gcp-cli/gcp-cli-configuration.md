# GCP CLI Configuration Management

Complete guide to configuring and managing `gcloud` CLI settings.

## Understanding gcloud Configuration

gcloud stores configuration in profiles, allowing you to manage multiple environments, projects, and credentials in one place.

### Configuration Locations
- **Linux/macOS**: `~/.config/gcloud/`
- **Windows**: `%APPDATA%\gcloud\`

### Key Directories
```
~/.config/gcloud/
├── configurations/           # Named configurations
├── properties                # Default properties
├── application_default_credentials.json
└── access_tokens             # Cached tokens
```

## Viewing Configuration

### View Current Configuration
```bash
# Show all current settings
gcloud config list

# Show specific section
gcloud config list compute

# Show specific property
gcloud config get-value core/project

# Show in different formats
gcloud config list --format=json
gcloud config list --format=yaml

# Show authentication info
gcloud auth list
gcloud auth print-access-token
```

### View Specific Properties
```bash
# Get current project
gcloud config get-value project

# Get current account
gcloud config get-value account

# Get current zone
gcloud config get-value compute/zone

# Get current region
gcloud config get-value compute/region

# Get output format
gcloud config get-value core/output_format
```

## Setting Configuration

### Set Properties
```bash
# Set project
gcloud config set project PROJECT_ID

# Set account
gcloud config set account EMAIL@gmail.com

# Set zone
gcloud config set compute/zone us-west1-a

# Set region
gcloud config set compute/region us-west1

# Set output format (json, yaml, table)
gcloud config set core/output_format json

# Disable interactive prompts
gcloud config set core/disable_prompts True

# Set default image project
gcloud config set compute/image_family debian-11
```

### Important Properties

#### Core Properties
```bash
# Project
gcloud config set core/project PROJECT_ID

# Account
gcloud config set core/account EMAIL@gmail.com

# Output format
gcloud config set core/output_format table

# Disable colored output
gcloud config set core/disable_colored_output True

# Disable prompts
gcloud config set core/disable_prompts True

# Disable usage reporting
gcloud config set core/disable_usage_reporting True

# VM serialport output
gcloud config set compute/use_new_usb_storage_interface True
```

#### Compute Properties
```bash
# Zone
gcloud config set compute/zone us-west1-a

# Region
gcloud config set compute/region us-west1

# Image family
gcloud config set compute/image_family debian-11

# Image project
gcloud config set compute/image_project debian-cloud

# Machine type
gcloud config set compute/machine_type n1-standard-1

# Network
gcloud config set compute/network default

# Preemptible VMs
gcloud config set compute/preemptible_by_default True
```

#### Container Properties
```bash
# Use application default credentials for kubectl
gcloud config set container/use_application_default_credentials True

# Use path to kubeconfig
gcloud config set container/kubeconfig_path ~/.kube/config
```

## Configuration Profiles

### Working with Configurations
```bash
# List all configurations
gcloud config configurations list

# Create new configuration
gcloud config configurations create PROFILE_NAME

# Create copy of existing configuration
gcloud config configurations create dev-config --activate

# Activate configuration
gcloud config configurations activate PROFILE_NAME

# Describe configuration
gcloud config configurations describe PROFILE_NAME

# Delete configuration
gcloud config configurations delete PROFILE_NAME
```

### Setup Multiple Project Profiles

#### Production Profile
```bash
# Create production configuration
gcloud config configurations create prod

# Set up production environment
gcloud config set project prod-project-id
gcloud config set account prod-account@company.com
gcloud config set compute/zone us-west1-a
gcloud config set compute/region us-west1

# Verify setup
gcloud config list
```

#### Development Profile
```bash
# Create development configuration
gcloud config configurations create dev

# Set up development environment
gcloud config set project dev-project-id
gcloud config set account dev-account@company.com
gcloud config set compute/zone us-central1-a
gcloud config set compute/region us-central1
```

#### Production - GKE Profile
```bash
# Create GKE-specific configuration
gcloud config configurations create prod-gke

gcloud config set project prod-project-id
gcloud config set container/use_application_default_credentials True
gcloud config set container/kubeconfig_path ~/.kube/config.prod
```

### Switch Profiles
```bash
# Switch to production
gcloud config configurations activate prod

# Verify active configuration
gcloud config configurations list  # Active marked with *

# Check current project/account
gcloud config list core
```

## Removing Configuration

### Unset Properties
```bash
# Remove specific property
gcloud config unset property-name

# Remove zone
gcloud config unset compute/zone

# Remove account
gcloud config unset account
```

### Delete Configurations
```bash
# Delete configuration profile
gcloud config configurations delete CONFIG_NAME

# Delete without confirmation
gcloud config configurations delete CONFIG_NAME --quiet
```

## Environment Variables

### Override Settings with Environment Variables

gcloud respects these environment variables:

```bash
# Project
export GCLOUD_PROJECT=PROJECT_ID
export GOOGLE_CLOUD_PROJECT=PROJECT_ID

# Compute zone
export GCLOUD_COMPUTE_ZONE=us-west1-a

# Compute region
export GCLOUD_COMPUTE_REGION=us-west1

# Account
export CLOUDSDK_CORE_ACCOUNT=EMAIL@gmail.com

# Output format
export CLOUDSDK_CORE_OUTPUT_FORMAT=json

# Disable colored output
export CLOUDSDK_CORE_DISABLE_COLORED_OUTPUT=1

# Credentials file
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account-key.json

# Custom kubeconfig
export KUBECONFIG=~/.kube/config

# Proxy settings
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=https://proxy.example.com:8080
export NO_PROXY=localhost,127.0.0.1
```

### Using Environment Variables
```bash
# Temporary override for single command
GCLOUD_PROJECT=other-project gcloud compute instances list

# Check which settings are set via environment
env | grep GCLOUD
env | grep GOOGLE
env | grep CLOUDSDK
```

## Configuration Files

### Edit Configuration Files Directly

#### Linux/macOS
```bash
# Edit default properties
nano ~/.config/gcloud/properties

# Edit specific configuration
nano ~/.config/gcloud/configurations/configname

# View kubeconfig (if using kubectl)
cat ~/.kube/config
```

#### Windows
```bash
# Properties file
%APPDATA%\gcloud\properties

# Configuration directory
%APPDATA%\gcloud\configurations\
```

### Properties File Format
```ini
[core]
project = my-project-id
account = user@example.com
disable_prompts = True
output_format = json

[compute]
zone = us-west1-a
region = us-west1

[container]
use_application_default_credentials = True
```

## Advanced Configuration

### Service Account Configuration
```bash
# Set service account credentials
gcloud auth activate-service-account --key-file=KEY_FILE.json

# View service account details
gcloud config get-value account

# Use specific service account
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account-key.json
```

### Proxy Configuration
```bash
# Set HTTP proxy
gcloud config set core/proxy http://proxy.example.com:8080

# Set HTTPS proxy  
gcloud config set core/proxy_type https

# No proxy for certain hosts
gcloud config set core/no_proxy localhost,127.0.0.1
```

### Log and Debug Configuration
```bash
# Enable debug logging
gcloud --verbosity=debug compute instances list

# Set debug level (debug, info, warning, error, critical)
gcloud --verbosity=debug projects list

# View gcloud logs
tail -f ~/.config/gcloud/logs/2024-*.log
```

## Configuration Best Practices

### Organize by Team/Project
```bash
# Create configurations for each team
gcloud config configurations create team-backend
gcloud config configurations create team-frontend
gcloud config configurations create team-infrastructure

# Switch easily between teams
gcloud config configurations activate team-backend
```

### Development vs Production
```bash
# Never use production credentials for development
gcloud config configurations create dev    # Uses dev credentials
gcloud config configurations create prod   # Uses prod credentials

# Always activate correct configuration before commands
gcloud config configurations list
gcloud config configurations activate correct-config
```

### Use Minimal Scope
```bash
# Always set default zone/region to avoid mistakes
gcloud config set compute/zone us-west1-a
gcloud config set compute/region us-west1

# This prevents accidental operations in wrong region
# gcloud compute instances list  # Shows us-west1 instances only
```

### Documentation
```bash
# Add comments in properties file (if editing directly)
# vim ~/.config/gcloud/properties

# Keep separate kubeconfig files
export KUBECONFIG=~/.kube/config.prod:~/.kube/config.dev

# Document configuration purposes
# prod - Production workloads (us-west1)
# dev  - Development workloads (us-central1)
# test - Test workloads (us-east1)
```

## Troubleshooting Configuration

### Issue: Wrong project/zone used
**Solution:**
```bash
# Check current configuration
gcloud config list

# Reset configuration
gcloud init

# Or set correct defaults
gcloud config set project CORRECT_PROJECT_ID
gcloud config set compute/zone CORRECT_ZONE
```

### Issue: Credentials not working
**Solution:**
```bash
# Check current account
gcloud auth list

# Re-authenticate
gcloud auth login

# Or use service account
gcloud auth activate-service-account --key-file=KEY_FILE.json

# Verify credentials work
gcloud auth application-default print-access-token
```

### Issue: Configuration conflicts
**Solution:**
```bash
# List all configurations
gcloud config configurations list

# Remove conflicting configuration
gcloud config configurations delete CONFIG_NAME

# Or create fresh configuration
gcloud config configurations create clean-config
gcloud config configurations activate clean-config
```

## Related Documentation

- [Installation](gcp-cli-installation.md) - Initial setup
- [Authentication](gcp-cli-authentication.md) - Credential management
- [Best Practices](gcp-cli-best-practices.md) - Configuration tips
