# GCP Components Management

Complete guide to managing gcloud CLI components and tools.

## What are Components?

gcloud components are optional software packages that extend the functionality of the `gcloud` CLI. They include:
- Language-specific SDKs and runtimes
- Service-specific tools and utilities
- Authentication plugins
- Cloud development tools

## Listing Components

### View All Components
```bash
# List all available components
gcloud components list

# Show only installed components
gcloud components list --only-available=false

# List in JSON format
gcloud components list --format=json

# Show with sizes
gcloud components list --format="table(name,size,status)"
```

### Component Information
```bash
# Describe specific component
gcloud components describe COMPONENT_NAME

# Get component details (YAML)
gcloud components describe kubectl --format=yaml

# Show installed version
gcloud components describe COMPONENT_NAME --format="value(version)"
```

## Installing Components

### Install Single Component
```bash
# Install component
gcloud components install COMPONENT_NAME

# Install without prompts
gcloud components install COMPONENT_NAME --quiet
```

### Install Common Components

#### GKE Required
```bash
# GKE gcloud authentication plugin (REQUIRED for GKE)
gcloud components install gke-gcloud-auth-plugin

# kubectl - Kubernetes command-line tool
gcloud components install kubectl

# Verify gke-gcloud-auth-plugin
gke-gcloud-auth-plugin --version
```

#### Cloud Development
```bash
# Skaffold (local development for Kubernetes)
gcloud components install skaffold

# Cloud Code (for IDE)
gcloud components install cloud-code

# Minikube support
gcloud components install minikube
```

#### App Engine
```bash
# App Engine Python runtime
gcloud components install app-engine-python

# App Engine Python extras
gcloud components install app-engine-python-extras

# App Engine Go runtime
gcloud components install app-engine-go

# App Engine Java runtime
gcloud components install app-engine-java
```

#### Other Tools
```bash
# Cloud SQL Proxy (for Cloud SQL connections)
gcloud components install cloud-sql-proxy

# Cloud Datastore Emulator
gcloud components install cloud-datastore-emulator

# Cloud Bigtable Emulator
gcloud components install bigtable

# Firestore Emulator
gcloud components install firestore-emulator

# Pub/Sub Emulator
gcloud components install pubsub-emulator

# Docker (if needed)
gcloud components install docker
```

### Install Multiple Components
```bash
# Install multiple components at once
gcloud components install \
  gke-gcloud-auth-plugin \
  kubectl \
  skaffold \
  cloud-sql-proxy

# Install without prompts
gcloud components install \
  gke-gcloud-auth-plugin \
  kubectl \
  --quiet
```

## Updating Components

### Update All Components
```bash
# Update all installed components
gcloud components update

# Update without prompts
gcloud components update --quiet

# Force update even if already latest
gcloud components update --force-update
```

### Update Specific Component
```bash
# First uninstall, then reinstall to get latest
gcloud components remove kubectl
gcloud components install kubectl
```

## Removing Components

### Remove Components
```bash
# Remove single component
gcloud components remove COMPONENT_NAME

# Remove without prompts
gcloud components remove COMPONENT_NAME --quiet

# Remove multiple components
gcloud components remove component1 component2 --quiet
```

## Component Directory

### Common Component Names

#### Required for GKE
| Component | Purpose |
|-----------|---------|
| `gke-gcloud-auth-plugin` | Authenticate with GKE clusters |
| `kubectl` | Kubernetes command-line interface |

#### Cloud Development
| Component | Purpose |
|-----------|---------|
| `skaffold` | Local Kubernetes development |
| `cloud-code` | VS Code/JetBrains IDE support |
| `minikube` | Local Kubernetes cluster |

#### App Engine
| Component | Purpose |
|-----------|---------|
| `app-engine-python` | Python App Engine runtime |
| `app-engine-go` | Go App Engine runtime |
| `app-engine-java` | Java App Engine runtime |
| `app-engine-node-js` | Node.js App Engine runtime |

#### Database Tools
| Component | Purpose |
|-----------|---------|
| `cloud-sql-proxy` | Connect to Cloud SQL |
| `cloud-datastore-emulator` | Local Datastore testing |
| `bigtable` | Bigtable emulator |
| `firestore-emulator` | Firestore emulator |

#### Emulators & Testing
| Component | Purpose |
|-----------|---------|
| `pubsub-emulator` | Pub/Sub emulator |
| `cloud-firestore-emulator` | Firestore emulator |
| `dataflow` | Apache Beam local runner |

### Check Component Status
```bash
# Is gke-gcloud-auth-plugin installed and working?
which gke-gcloud-auth-plugin
gke-gcloud-auth-plugin --version

# Is kubectl installed?
which kubectl
kubectl version --client

# Is skaffold installed?
which skaffold
skaffold version
```

## Troubleshooting Components

### Issue: "Component not found"
**Solution:**
```bash
# List available components
gcloud components list

# Check spelling
gcloud components install GKE_GCLOUD_AUTH_PLUGIN  # WRONG
gcloud components install gke-gcloud-auth-plugin  # CORRECT
```

### Issue: Installation fails or hangs
**Solution:**
```bash
# Try with verbose output
gcloud components install COMPONENT_NAME --verbosity=debug

# Or try update instead
gcloud components update

# Check internet connection
ping www.google.com
```

### Issue: Component command not in PATH
**Solution:**
```bash
# After installation, restart terminal
exec -l $SHELL

# Or source gcloud
source ~/.bashrc  # or ~/.zshrc

# Check gcloud bin directory
ls ~/.local/share/google-cloud-sdk/bin/

# Add to PATH manually if needed
export PATH=$PATH:~/.local/share/google-cloud-sdk/bin
```

### Issue: Version conflicts
**Solution:**
```bash
# Update all components
gcloud components update --force-update

# Check component versions
gcloud components describe gke-gcloud-auth-plugin --format="value(version)"
gcloud components describe kubectl --format="value(version)"
```

## Working with Components

### After Installing gke-gcloud-auth-plugin
```bash
# Update kubeconfig to use plugin
gcloud container clusters get-credentials CLUSTER_NAME --zone=ZONE

# Verify plugin is being used
kubectl auth can-i get nodes

# Debug authentication
kubectl auth auth-provider
```

### Working with Cloud SQL Proxy
```bash
# After installation, use to connect
cloud_sql_proxy -instances=PROJECT_ID:REGION:INSTANCE_NAME=tcp:3306

# Or as background process
cloud_sql_proxy -instances=PROJECT_ID:REGION:INSTANCE_NAME=tcp:3306 &
```

### Working with Emulators
```bash
# Start Firestore emulator
gcloud beta emulators firestore start

# Start Pub/Sub emulator
gcloud beta emulators pubsub start

# Set environment for tests
export FIRESTORE_EMULATOR_HOST=localhost:8081
```

## Best Practices

### Keep Components Updated
```bash
# Regular update schedule
gcloud components update

# Check for new versions
gcloud components list --format="table(name,version,state)"
```

### Only Install What You Need
```bash
# Don't install unneeded components
# They consume disk space and can slow down operations

# For GKE only, install just:
gcloud components install gke-gcloud-auth-plugin
gcloud components install kubectl

# For App Engine Python only, install just:
gcloud components install app-engine-python
```

### Use Container Images for Tools
```bash
# Instead of installing components, use Docker images
# For tools you don't use frequently

docker run -it gcr.io/cloud-builders/kubectl:latest \
  --context=gke_PROJECT_ID_ZONE_CLUSTER_NAME

docker run -it gcr.io/cloud-builders/skaffold \
  version
```

## Related Documentation

- [Installation](gcp-cli-installation.md) - gcloud SDK setup
- [GKE](gcp-cli-gke.md) - Kubernetes cluster management
- [Best Practices](gcp-cli-best-practices.md) - CLI best practices
