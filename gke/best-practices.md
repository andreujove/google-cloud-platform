# Google Kubernetes Engine (GKE) Best Practices

A comprehensive guide to creating, configuring, and managing Google Kubernetes Engine clusters in production environments.

## Table of Contents

- [Networking Best Practices](#networking-best-practices)
- [Cluster Configuration](#cluster-configuration)
- [Security Best Practices](#security-best-practices)
- [Scalability & Performance](#scalability--performance)
- [Monitoring & Logging](#monitoring--logging)
- [Cost Optimization](#cost-optimization)
- [Disaster Recovery](#disaster-recovery)

## Networking Best Practices

### VPC Configuration
- **Disable auto_create_subnetworks**: Never use "Default" VPCs for production; they are messy and insecure.
  - Explicitly define VPCs and subnets for better control and isolation
  - Use separate subnets for different workload types

### Private IP Access
- **Enable private_ip_google_access**: This allows your pods to talk to BigQuery, Cloud Storage, and other Google services without leaving the Google network.
  - Improves security by avoiding internet-facing traffic
  - Reduces latency for GCP service access

### Outbound Traffic Management
- **Cloud NAT**: Only allow outbound traffic from your cluster.
  - Never assign public IPs to GKE nodes unless absolutely necessary
  - Use Cloud NAT for egress traffic to external services
  - Implement network policies to control pod-to-pod communication

## Cluster Configuration

### Regional Deployments
- **Use Regional Locations**: By setting `location = "us-central1"`, GKE automatically:
  - Replicates your control plane across multiple zones
  - Distributes your nodes across three zones (e.g., a, b, and c)
  - Provides high availability and disaster recovery built-in
  - Delivers automatic control plane upgrades with zero downtime

### Node Pool Configuration
- Use **multiple node pools** for different workload types:
  - Standard nodes for general workloads
  - Memory-optimized nodes for data-intensive applications
  - CPU-optimized nodes for compute-intensive tasks
  - Preemptible/Spot VMs for cost-effective, non-critical workloads

- **Enable Node Auto-Scaling**:
  - Set `min_node_count` and `max_node_count` for each node pool
  - GKE automatically scales nodes based on resource demand
  - Prevents overprovisioning and reduces costs

### Release Channels
- Use **GKE Release Channels** for automatic updates:
  - **Rapid**: Latest features with minimal testing
  - **Regular**: Balanced approach (recommended)
  - **Stable**: Conservative, battle-tested versions
  - Channels provide automatic node upgrades (window: 5-10 days)

## Security Best Practices

### RBAC (Role-Based Access Control)
- Always enable **Workload Identity** for pod authentication:
  - Map Kubernetes Service Accounts to IAM service accounts
  - Pods get temporary credentials without needing long-lived keys
  - Automatic credential rotation

### Network Policies
- Implement **Network Policies** to control traffic flow:
  - Restrict ingress/egress between pods
  - Deny all by default, allow only necessary traffic
  - Better security posture than relying solely on firewalls

### Pod Security Standards
- Enforce **Pod Security Policies** or **Pod Security Standards**:
  - Restrict privileged containers
  - Prevent root user execution
  - Disable host network access
  - Read-only root filesystem where possible

### Image Security
- **Enable Binary Authorization**:
  - Only allow verified, signed container images to run
  - Integrate with your CI/CD pipeline
  - Ensure image provenance and integrity

- Use **Google Artifact Registry** instead of Docker Hub:
  - Automatically scan images for vulnerabilities
  - Private repositories for sensitive workloads
  - Better integration with GCP services

## Scalability & Performance

### Horizontal Pod Autoscaling (HPA)
- Configure HPA based on:
  - CPU utilization (threshold: 70-80%)
  - Memory usage
  - Custom metrics (requests per second, latency)
  - Combine multiple metrics for better decision-making

### Vertical Pod Autoscaling (VPA)
- Use **VPA** for right-sizing containers:
  - Automatically recommends CPU and memory requests
  - Adjusts resource limits based on actual usage
  - Works well with HPA for comprehensive scaling

### Cluster Autoscaling
- Enable with appropriate limits:
  - Set `min_node_count` and `max_node_count`
  - Configure scaledown settings (stabilization window: 10 minutes)
  - Reserve capacity for critical workloads

## Monitoring & Logging

### Google Cloud Operations Suite
- **Enable GKE Monitoring**:
  - Use Google Cloud Operations (formerly Stackdriver)
  - Monitor control plane metrics
  - Track node health and availability
  - Set up alerts for critical metrics

- **Enable GKE Logging**:
  - Collect container logs automatically
  - Stream to Cloud Logging for central analysis
  - Enable audit logging for compliance and security

### Observability Best Practices
- Implement **structured logging** (JSON format):
  - Use consistent log levels (DEBUG, INFO, WARN, ERROR)
  - Include request IDs for distributed tracing
  - Log relevant context (user, tenant, region)

- Use **Cloud Trace** for distributed tracing:
  - Track requests across multiple microservices
  - Identify performance bottlenecks
  - Analyze call chains and latencies

## Cost Optimization

### Preemptible and Spot VMs
- Use **Preemptible VMs** (batch jobs, non-critical workloads):
  - 60-90% cheaper than standard compute
  - Can be interrupted with 30-second notice
  - Combine with regular nodes for fault tolerance

- Use **Spot VMs** (newer, longer availability):
  - Similar discounts to preemptible
  - Better availability than preemptible VMs
  - Ideal for flexible, non-time-critical workloads

### Resource Requests and Limits
- Always define resource requests:
  - Enables efficient bin-packing and scheduling
  - Improves cluster utilization
  - Makes cost calculations more accurate

- Set appropriate limits:
  - Prevent runaway containers from consuming resources
  - Protect cluster stability
  - Balance between safety and flexibility

### Committed Use Discounts (CUDs)
- For production workloads with predictable usage:
  - 1-year or 3-year commitments (25-50% savings)
  - Negotiate after establishing baseline usage
  - Apply to compute resources and GKE clusters

## Disaster Recovery

### Multi-Region Strategy
- **Multi-region deployments** for critical applications:
  - Deploy across multiple GCP regions
  - Use global load balancing
  - Implement cross-region failover
  - Reduces blast radius of regional outages

### Backup Strategy
- **Enable GKE Backup and Disaster Recovery**:
  - Automated backups of cluster state (etcd)
  - Point-in-time recovery capabilities
  - Test restore procedures regularly
  - Document RTO/RPO requirements

### Pod Disruption Budgets (PDBs)
- Define PDBs for critical workloads:
  - Specify minimum available replicas during disruptions
  - Protect against accidental pod evictions
  - Cluster updates respect PDB constraints

## Quick Reference: Regional Cluster Template

```bash
# Create a production-ready regional GKE cluster
gcloud container clusters create CLUSTER_NAME \
  --region us-central1 \
  --num-nodes 3 \
  --machine-type n1-standard-4 \
  --enable-autoscaling \
  --min-nodes 3 \
  --max-nodes 10 \
  --network CUSTOM_VPC \
  --subnetwork CUSTOM_SUBNET \
  --enable-ip-alias \
  --enable-private-ip-google-access \
  --enable-network-policy \
  --workload-pool=PROJECT_ID.svc.id.goog \
  --enable-stackdriver-kubernetes \
  --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver \
  --release-channel regular
```