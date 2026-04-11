# GCP Compute Engine Instances

Complete guide to managing Compute Engine VM instances using the `gcloud` CLI.

## Listing Instances

### List All Instances
```bash
# List all instances in current project/zone
gcloud compute instances list

# List in specific zone
gcloud compute instances list --zones=us-west1-a

# List in specific region
gcloud compute instances list --regions=us-west1

# List in JSON format
gcloud compute instances list --format=json

# List with custom columns
gcloud compute instances list --format="table(name,zone,machineType.machine_type(),status)"

# Filter by status
gcloud compute instances list --filter="status:RUNNING"

# Filter by name
gcloud compute instances list --filter="name:web-*"

# Limit results
gcloud compute instances list --limit=5
```

### Get Instance Details
```bash
# Describe specific instance
gcloud compute instances describe INSTANCE_NAME --zone=ZONE

# Get specific fields
gcloud compute instances describe INSTANCE_NAME --zone=ZONE --format="value(internalIp)"
gcloud compute instances describe INSTANCE_NAME --zone=ZONE --format="value(externalIp)"

# Get in JSON format
gcloud compute instances describe INSTANCE_NAME --zone=ZONE --format=json
```

## Creating Instances

### Basic Instance Creation
```bash
# Create simple instance
gcloud compute instances create INSTANCE_NAME \
  --zone=us-west1-a

# Create with specific machine type
gcloud compute instances create INSTANCE_NAME \
  --zone=us-west1-a \
  --machine-type=n1-standard-1

# Create with custom image
gcloud compute instances create INSTANCE_NAME \
  --zone=us-west1-a \
  --image-family=debian-11 \
  --image-project=debian-cloud

# Create with boot disk size
gcloud compute instances create INSTANCE_NAME \
  --zone=us-west1-a \
  --boot-disk-size=100GB
```

### Advanced Instance Creation
```bash
# Create with multiple options
gcloud compute instances create INSTANCE_NAME \
  --zone=us-west1-a \
  --machine-type=n1-standard-2 \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-standard \
  --metadata=startup-script='#!/bin/bash\necho "Hello"' \
  --network=vpc-name \
  --subnet=subnet-name \
  --tags=http-server,https-server \
  --labels=environment=dev,team=backend \
  --scopes=https://www.googleapis.com/auth/cloud-platform
```

### Using Instance Templates
```bash
# Create from template
gcloud compute instances create INSTANCE_NAME \
  --source-instance-template=TEMPLATE_NAME \
  --zone=us-west1-a

# List templates
gcloud compute instance-templates list

# Describe template
gcloud compute instance-templates describe TEMPLATE_NAME
```

### Create Multiple Instances
```bash
# Create with name pattern
gcloud compute instances create web-{1,2,3} \
  --zone=us-west1-a \
  --machine-type=n1-standard-1
```

## Connecting to Instances

### SSH Access
```bash
# SSH into instance (uses gcloud to manage SSH keys)
gcloud compute ssh INSTANCE_NAME --zone=us-west1-a

# SSH with specific user
gcloud compute ssh USER@INSTANCE_NAME --zone=us-west1-a

# Run command via SSH
gcloud compute ssh INSTANCE_NAME --zone=us-west1-a -- 'command to run'

# Run script via SSH
gcloud compute ssh INSTANCE_NAME --zone=us-west1-a -- 'bash -s' < script.sh

# SCP - Copy file to instance
gcloud compute scp LOCAL_FILE INSTANCE_NAME:/remote/path --zone=us-west1-a

# SCP - Copy file from instance
gcloud compute scp INSTANCE_NAME:/remote/file LOCAL_PATH --zone=us-west1-a

# SCP - Copy directory
gcloud compute scp --recurse LOCAL_DIR INSTANCE_NAME:/remote/path --zone=us-west1-a
```

### RDP Access (Windows)
```bash
# Get RDP credentials
gcloud compute instances get-serial-port-output INSTANCE_NAME --zone=us-west1-a

# Set Windows password
gcloud compute reset-windows-password INSTANCE_NAME --zone=us-west1-a
```

## Managing Instance State

### Start, Stop, and Restart
```bash
# Stop instance (preserves disks and static IPs)
gcloud compute instances stop INSTANCE_NAME --zone=us-west1-a

# Start stopped instance
gcloud compute instances start INSTANCE_NAME --zone=us-west1-a

# Restart instance
gcloud compute instances restart INSTANCE_NAME --zone=us-west1-a

# Stop multiple instances
gcloud compute instances stop INSTANCE_1 INSTANCE_2 --zone=us-west1-a

# Stop all instances in zone
gcloud compute instances list --zones=us-west1-a --format="value(name)" | xargs gcloud compute instances stop --zone=us-west1-a
```

### Delete Instances
```bash
# Delete instance
gcloud compute instances delete INSTANCE_NAME --zone=us-west1-a

# Delete without confirmation
gcloud compute instances delete INSTANCE_NAME --zone=us-west1-a --quiet

# Delete multiple instances
gcloud compute instances delete INSTANCE_1 INSTANCE_2 --zone=us-west1-a
```

## Instance Configuration

### Resize Instance
```bash
# Stop instance first
gcloud compute instances stop INSTANCE_NAME --zone=us-west1-a

# Change machine type
gcloud compute instances set-machine-type INSTANCE_NAME \
  --machine-type=n1-standard-4 \
  --zone=us-west1-a

# Start instance
gcloud compute instances start INSTANCE_NAME --zone=us-west1-a
```

### Metadata Management
```bash
# Add metadata
gcloud compute instances add-metadata INSTANCE_NAME \
  --metadata=key1=value1,key2=value2 \
  --zone=us-west1-a

# Remove metadata
gcloud compute instances remove-metadata INSTANCE_NAME \
  --keys=key1,key2 \
  --zone=us-west1-a

# Set startup script
gcloud compute instances add-metadata INSTANCE_NAME \
  --metadata-from-file=startup-script=script.sh \
  --zone=us-west1-a
```

### Labels
```bash
# Add labels
gcloud compute instances add-labels INSTANCE_NAME \
  --labels=environment=prod,team=backend \
  --zone=us-west1-a

# Remove labels
gcloud compute instances remove-labels INSTANCE_NAME \
  --labels=old-label \
  --zone=us-west1-a

# Update labels
gcloud compute instances update INSTANCE_NAME \
  --update-labels=environment=prod,billing=project-123 \
  --zone=us-west1-a
```

## Networking

### Configure IP Addresses
```bash
# Reserve static external IP
gcloud compute addresses create STATIC_IP_NAME --region=us-west1

# Attach static IP to instance
gcloud compute instances add-access-config INSTANCE_NAME \
  --address=STATIC_IP_NAME \
  --zone=us-west1-a

# View instance IPs
gcloud compute instances describe INSTANCE_NAME --zone=us-west1-a \
  --format="table(name,internalIp,externalIp)"

# Release static IP from instance
gcloud compute instances delete-access-config INSTANCE_NAME \
  --access-config-name="External NAT" \
  --zone=us-west1-a
```

### Firewall and Security
```bash
# Add tags to instance (for firewall rules)
gcloud compute instances add-tags INSTANCE_NAME \
  --tags=http-server,https-server \
  --zone=us-west1-a

# Remove tags
gcloud compute instances remove-tags INSTANCE_NAME \
  --tags=old-tag \
  --zone=us-west1-a

# Apply service account to instance
gcloud compute instances create INSTANCE_NAME \
  --service-account=SA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --zone=us-west1-a
```

## Snapshots and Disks

### Create Snapshots
```bash
# Create snapshot from instance's boot disk
gcloud compute disks snapshot BOOT_DISK_NAME \
  --snapshot-names=SNAPSHOT_NAME \
  --zone=us-west1-a

# List snapshots
gcloud compute snapshots list

# Delete snapshot
gcloud compute snapshots delete SNAPSHOT_NAME
```

### Manage Disks
```bash
# List disks
gcloud compute disks list --zones=us-west1-a

# Create new disk
gcloud compute disks create DISK_NAME \
  --size=100GB \
  --zone=us-west1-a

# Attach disk to instance
gcloud compute instances attach-disk INSTANCE_NAME \
  --disk=DISK_NAME \
  --zone=us-west1-a

# Detach disk
gcloud compute instances detach-disk INSTANCE_NAME \
  --disk=DISK_NAME \
  --zone=us-west1-a

# Delete disk
gcloud compute disks delete DISK_NAME --zone=us-west1-a
```

## Instance Groups and Autoscaling

### Instance Groups
```bash
# List instance groups
gcloud compute instance-groups list

# List instances in group
gcloud compute instance-groups list-instances INSTANCE_GROUP \
  --zone=us-west1-a

# Create instance group
gcloud compute instance-groups managed create INSTANCE_GROUP \
  --base-instance-name=base-name \
  --size=3 \
  --template=TEMPLATE_NAME \
  --zone=us-west1-a
```

## Troubleshooting

### Issue: "Cannot SSH into instance"
**Solution:**
```bash
# Check if instance is running
gcloud compute instances describe INSTANCE_NAME --zone=us-west1-a

# Check firewall rules allow SSH
gcloud compute firewall-rules list --filter="allowed.ports:22"

# Add SSH firewall rule
gcloud compute firewall-rules create allow-ssh \
  --allow=tcp:22 \
  --target-tags=ssh-enabled
```

### Issue: "External IP not available"
**Solution:**
```bash
# Add access config to get external IP
gcloud compute instances add-access-config INSTANCE_NAME \
  --zone=us-west1-a
```

## Related Documentation

- [Configuration Management](gcp-cli-configuration.md) - Default zones and regions
- [Best Practices](gcp-cli-best-practices.md) - SSH tips and best practices
