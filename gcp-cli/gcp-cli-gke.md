# GCP GKE (Google Kubernetes Engine)

Complete guide to managing Google Kubernetes Engine clusters using the `gcloud` CLI.

## Listing Clusters

### List All Clusters
```bash
# List all GKE clusters in current project
gcloud container clusters list

# List in specific zone
gcloud container clusters list --zone=us-west1-a

# List in specific region
gcloud container clusters list --region=us-west1

# List in JSON format
gcloud container clusters list --format=json

# List with custom columns
gcloud container clusters list --format="table(name,zone,status,numNodes)"

# Filter by cluster name
gcloud container clusters list --filter="name:prod-*"

# Limit results
gcloud container clusters list --limit=5
```

### Get Cluster Details
```bash
# Describe specific cluster
gcloud container clusters describe CLUSTER_NAME --zone=ZONE

# Get cluster version
gcloud container clusters describe CLUSTER_NAME --zone=ZONE \
  --format="value(currentMasterVersion)"

# Get cluster endpoint
gcloud container clusters describe CLUSTER_NAME --zone=ZONE \
  --format="value(endpoint)"

# Get in JSON format
gcloud container clusters describe CLUSTER_NAME --zone=ZONE --format=json
```

## Creating Clusters

### Basic Cluster Creation
```bash
# Create simple cluster
gcloud container clusters create CLUSTER_NAME \
  --zone=us-west1-a

# Create with specified number of nodes
gcloud container clusters create CLUSTER_NAME \
  --zone=us-west1-a \
  --num-nodes=3

# Create regional cluster (high availability)
gcloud container clusters create CLUSTER_NAME \
  --region=us-west1 \
  --num-nodes=1  # Nodes per zone (3 zones = 3 total)
```

### Advanced Cluster Creation
```bash
# Production-ready cluster
gcloud container clusters create CLUSTER_NAME \
  --region=us-west1 \
  --num-nodes=3 \
  --machine-type=n1-standard-2 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --enable-ip-alias \
  --enable-private-ip-google-access \
  --enable-network-policy \
  --workload-pool=PROJECT_ID.svc.id.goog \
  --release-channel=regular \
  --enable-stackdriver-kubernetes \
  --addons=HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver
```

### Node Configuration
```bash
# Cluster with specific machine type
gcloud container clusters create CLUSTER_NAME \
  --zone=us-west1-a \
  --machine-type=n1-standard-4

# Cluster with preemptible nodes (cost-effective)
gcloud container clusters create CLUSTER_NAME \
  --zone=us-west1-a \
  --preemptible

# Cluster with custom disk size
gcloud container clusters create CLUSTER_NAME \
  --zone=us-west1-a \
  --disk-size=50

# Cluster with custom network
gcloud container clusters create CLUSTER_NAME \
  --zone=us-west1-a \
  --network=NETWORK_NAME \
  --subnetwork=SUBNET_NAME
```

## Cluster Authentication

### Get Cluster Credentials
```bash
# Get credentials for kubectl
gcloud container clusters get-credentials CLUSTER_NAME --zone=us-west1-a

# Configure kubeconfig with specific context
gcloud container clusters get-credentials CLUSTER_NAME \
  --zone=us-west1-a \
  --project=PROJECT_ID
```

### kubectl Integration
```bash
# After get-credentials, kubectl is ready to use
kubectl get nodes

# View kubeconfig
kubectl config view

# Get cluster info
kubectl cluster-info
```

## Managing Clusters

### Resize Cluster
```bash
# Set number of nodes
gcloud container clusters resize CLUSTER_NAME \
  --num-nodes=5 \
  --zone=us-west1-a

# Disable autoscaling
gcloud container clusters resize CLUSTER_NAME \
  --enable-autoscaling=false \
  --zone=us-west1-a
```

### Autoscaling Configuration
```bash
# Enable autoscaling
gcloud container clusters update CLUSTER_NAME \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --zone=us-west1-a

# Update autoscaling bounds
gcloud container clusters update CLUSTER_NAME \
  --min-nodes=2 \
  --max-nodes=20 \
  --zone=us-west1-a

# Disable autoscaling
gcloud container clusters update CLUSTER_NAME \
  --enable-autoscaling=false \
  --zone=us-west1-a
```

### Monitoring and Logging
```bash
# Enable monitoring and logging
gcloud container clusters update CLUSTER_NAME \
  --enable-cloud-logging \
  --enable-cloud-monitoring \
  --zone=us-west1-a

# Disable monitoring
gcloud container clusters update CLUSTER_NAME \
  --logging=NONE \
  --zone=us-west1-a
```

### Network Policies
```bash
# Enable network policy
gcloud container clusters update CLUSTER_NAME \
  --enable-network-policy \
  --zone=us-west1-a

# Disable network policy
gcloud container clusters update CLUSTER_NAME \
  --enable-network-policy=false \
  --zone=us-west1-a
```

## Node Management

### List Nodes
```bash
# List node pools
gcloud container node-pools list --cluster=CLUSTER_NAME --zone=us-west1-a

# Describe node pool
gcloud container node-pools describe NODE_POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=us-west1-a
```

### Create Node Pools
```bash
# Create additional node pool
gcloud container node-pools create NODE_POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=us-west1-a \
  --machine-type=n1-standard-2 \
  --num-nodes=3

# Create preemptible node pool
gcloud container node-pools create preemptible-pool \
  --cluster=CLUSTER_NAME \
  --zone=us-west1-a \
  --preemptible \
  --num-nodes=1 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5
```

### Update Node Pools
```bash
# Enable autoscaling on node pool
gcloud container node-pools update NODE_POOL_NAME \
  --cluster=CLUSTER_NAME \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=10 \
  --zone=us-west1-a

# Add labels to node pool
gcloud container node-pools update NODE_POOL_NAME \
  --cluster=CLUSTER_NAME \
  --node-labels=workload=batch \
  --zone=us-west1-a

# Update machine type (requires new pool)
gcloud container node-pools delete NODE_POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=us-west1-a
```

### Delete Node Pools
```bash
# Delete node pool
gcloud container node-pools delete NODE_POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=us-west1-a
```

## Cluster Updates

### Kubernetes Version Management
```bash
# Check available master upgrades
gcloud container get-server-config --zone=us-west1-a

# Get current cluster version
gcloud container clusters describe CLUSTER_NAME --zone=us-west1-a \
  --format="value(currentMasterVersion)"

# Upgrade control plane
gcloud container clusters upgrade CLUSTER_NAME \
  --master \
  --cluster-version=VERSION \
  --zone=us-west1-a

# Upgrade nodes
gcloud container clusters upgrade CLUSTER_NAME \
  --node-pool=default-pool \
  --zone=us-west1-a
```

### Release Channels
```bash
# Enable release channel
gcloud container clusters update CLUSTER_NAME \
  --release-channel=regular \
  --zone=us-west1-a

# Change release channel (regular, rapid, stable)
gcloud container clusters update CLUSTER_NAME \
  --release-channel=stable \
  --zone=us-west1-a
```

## Workload Identity

### Setup Workload Identity
```bash
# Enable Workload Identity on cluster
gcloud container clusters update CLUSTER_NAME \
  --workload-pool=PROJECT_ID.svc.id.goog \
  --zone=us-west1-a

# Create Kubernetes service account
kubectl create serviceaccount KSA_NAME

# Create Google service account
gcloud iam service-accounts create GSA_NAME

# Bind Kubernetes SA to Google SA
gcloud iam service-accounts add-iam-policy-binding \
  GSA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="serviceAccount:PROJECT_ID.svc.id.goog[NAMESPACE/KSA_NAME]"

# Annotate Kubernetes service account
kubectl annotate serviceaccount KSA_NAME \
  iam.gke.io/gcp-service-account=GSA_NAME@PROJECT_ID.iam.gserviceaccount.com
```

## Clustering Operations

### Delete Cluster
```bash
# Delete cluster
gcloud container clusters delete CLUSTER_NAME --zone=us-west1-a

# Delete without confirmation
gcloud container clusters delete CLUSTER_NAME --zone=us-west1-a --quiet
```

### Cluster Backup and Recovery
```bash
# Enable daily backups
gcloud container fleet dataplane-management backup create \
  --cluster=CLUSTER_NAME

# List backups
gcloud container backup-restore backup-plans list

# Restore from backup
gcloud container backup-restore restores create RESTORE_NAME \
  --backup-plan=BACKUP_PLAN \
  --cluster=CLUSTER_NAME
```

## Troubleshooting

### Issue: "Cannot connect to cluster"
**Solution:**
```bash
# Get cluster credentials again
gcloud container clusters get-credentials CLUSTER_NAME --zone=us-west1-a

# Verify kubectl can connect
kubectl cluster-info

# Check kubeconfig
kubectl config current-context
```

### Issue: "Insufficient resources"
**Solution:**
```bash
# Resize cluster
gcloud container clusters resize CLUSTER_NAME --num-nodes=5 --zone=us-west1-a

# Or enable autoscaling if not already enabled
gcloud container clusters update CLUSTER_NAME \
  --enable-autoscaling \
  --max-nodes=20 \
  --zone=us-west1-a
```

### Issue: "Node pool quota exceeded"
**Solution:**
```bash
# Check quota
gcloud compute project-info describe --project=PROJECT_ID

# Request quota increase through GCP Console
# Or delete unused clusters/node pools
gcloud container node-pools delete NODE_POOL_NAME \
  --cluster=CLUSTER_NAME \
  --zone=us-west1-a
```

## Related Documentation

- [GKE Best Practices](../gke/README.md) - Production recommendations
- [Authentication](gcp-cli-authentication.md) - Service account setup
- [Components](gcp-cli-components.md) - GKE authentication plugin
