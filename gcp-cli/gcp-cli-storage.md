# GCP Cloud Storage (gsutil)

Complete guide to managing Google Cloud Storage buckets and objects using `gsutil`.

## Listing Buckets and Objects

### List Buckets
```bash
# List all buckets in project
gsutil ls

# List with size information
gsutil ls -lh

# List with storage class
gsutil ls -L gs://BUCKET_NAME

# List buckets matching pattern
gsutil ls gs://bucket-prefix*

# List in long format
gsutil ls -l
```

### List Objects in Bucket
```bash
# List objects in bucket
gsutil ls gs://BUCKET_NAME

# List objects recursively
gsutil ls -r gs://BUCKET_NAME

# List by size
gsutil ls -lh gs://BUCKET_NAME

# List with timestamps
gsutil ls -l gs://BUCKET_NAME

# Filter by prefix (folder-like)
gsutil ls gs://BUCKET_NAME/prefix/
```

### Object Information
```bash
# Get object metadata
gsutil stat gs://BUCKET_NAME/OBJECT_NAME

# Get bucket metadata
gsutil stat gs://BUCKET_NAME

# Show object details
gsutil du -sh gs://BUCKET_NAME/OBJECT_NAME
```

## Bucket Management

### Create Buckets
```bash
# Create bucket (must be globally unique)
gsutil mb gs://unique-bucket-name

# Create with storage class
gsutil mb -c STANDARD gs://bucket-name

# Create in specific location
gsutil mb -l us-west1 gs://bucket-name

# Create with storage class and location
gsutil mb -c STANDARD -l us gs://bucket-name
```

### Storage Classes
```bash
# Standard (for frequent access)
gsutil mb -c STANDARD gs://bucket-name

# Nearline (infrequent access)
gsutil mb -c NEARLINE gs://bucket-name

# Coldline (rare access)
gsutil mb -c COLDLINE gs://bucket-name

# Archive (long-term storage)
gsutil mb -c ARCHIVE gs://bucket-name
```

### Delete Buckets
```bash
# Delete empty bucket
gsutil rb gs://BUCKET_NAME

# Delete bucket with objects (recursive)
gsutil -m rm -r gs://BUCKET_NAME

# Must be completely empty to delete
gsutil -m rm -r gs://BUCKET_NAME/*
gsutil rb gs://BUCKET_NAME
```

## Uploading Objects

### Upload Single Files
```bash
# Upload file
gsutil cp LOCAL_FILE gs://BUCKET_NAME

# Upload with specific name
gsutil cp LOCAL_FILE gs://BUCKET_NAME/OBJECT_NAME

# Upload with progress display
gsutil -m cp LOCAL_FILE gs://BUCKET_NAME

# Copy with no clobber (skip if exists)
gsutil -m cp -n LOCAL_FILE gs://BUCKET_NAME
```

### Upload Directories
```bash
# Upload entire directory recursively
gsutil -m cp -r LOCAL_DIR gs://BUCKET_NAME

# Upload directory with no clobber
gsutil -m cp -r -n LOCAL_DIR gs://BUCKET_NAME

# Upload and preserve directory structure
gsutil -m cp -r LOCAL_DIR/* gs://BUCKET_NAME/prefix/
```

### Upload with Options
```bash
# Upload with compression
gsutil -m -h "Content-Encoding:gzip" cp LOCAL_FILE.gz gs://BUCKET_NAME

# Upload with specific content type
gsutil -h "Content-Type:application/json" cp file.json gs://BUCKET_NAME

# Upload with metadata
gsutil -h "Cache-Control:public, max-age=3600" cp LOCAL_FILE gs://BUCKET_NAME

# Upload with multiple headers
gsutil -h "Content-Type:text/html" \
  -h "Cache-Control:public, max-age=86400" \
  cp index.html gs://BUCKET_NAME
```

### Parallel Upload
```bash
# Upload with parallelism (faster for large files)
gsutil -m -o GSUtil:parallel_thread_count=10 cp -r LOCAL_DIR gs://BUCKET_NAME

# Upload multiple files in parallel
gsutil -m cp file1 file2 file3 gs://BUCKET_NAME
```

## Downloading Objects

### Download Files
```bash
# Download single file
gsutil cp gs://BUCKET_NAME/OBJECT_NAME LOCAL_FILE

# Download to current directory
gsutil cp gs://BUCKET_NAME/OBJECT_NAME .

# Download multiple files
gsutil -m cp gs://BUCKET_NAME/file1 gs://BUCKET_NAME/file2 .

# Download with specific destination
gsutil cp gs://BUCKET_NAME/OBJECT_NAME /path/to/local/file
```

### Download Directories
```bash
# Download entire bucket
gsutil -m cp -r gs://BUCKET_NAME .

# Download objects with prefix
gsutil -m cp -r gs://BUCKET_NAME/prefix/ LOCAL_DIR

# Download and overwrite
gsutil -m cp -r gs://BUCKET_NAME LOCAL_DIR
```

## Moving and Copying

### Copy Between Buckets
```bash
# Copy object from one bucket to another
gsutil cp gs://SOURCE_BUCKET/OBJECT gs://DEST_BUCKET

# Copy with prefix
gsutil cp gs://SOURCE_BUCKET/prefix/* gs://DEST_BUCKET/prefix/

# Recursive copy between buckets
gsutil -m cp -r gs://SOURCE_BUCKET gs://DEST_BUCKET
```

### Move Objects (Rename)
```bash
# Move/rename object within bucket
gsutil mv gs://BUCKET_NAME/old-name gs://BUCKET_NAME/new-name

# Move between buckets
gsutil mv gs://SOURCE_BUCKET/OBJECT gs://DEST_BUCKET

# Move with prefix
gsutil -m mv gs://BUCKET_NAME/old-prefix/* gs://BUCKET_NAME/new-prefix/
```

## Deleting Objects

### Delete Files
```bash
# Delete single file
gsutil rm gs://BUCKET_NAME/OBJECT_NAME

# Delete multiple files
gsutil -m rm gs://BUCKET_NAME/file1 gs://BUCKET_NAME/file2

# Delete with pattern
gsutil -m rm gs://BUCKET_NAME/prefix/*.txt

# Delete all objects with prefix
gsutil -m rm -r gs://BUCKET_NAME/prefix/
```

### Batch Deletion
```bash
# Delete using -m for parallelism
gsutil -m rm gs://BUCKET_NAME/path/

# Confirm before deletion
gsutil -m rm gs://BUCKET_NAME/prefix/*  # Interactive prompt

# Force delete without confirmation
gsutil -m rm -r gs://BUCKET_NAME/prefix/
```

## Access Control and Permissions

### Public Access
```bash
# Make object public
gsutil acl ch -u AllUsers:R gs://BUCKET_NAME/OBJECT_NAME

# Make bucket public
gsutil iam ch allUsers:objectViewer gs://BUCKET_NAME

# Make object private
gsutil acl ch -d AllUsers gs://BUCKET_NAME/OBJECT_NAME

# Make bucket private
gsutil iam ch -d allUsers:objectViewer gs://BUCKET_NAME
```

### Grant User Permissions
```bash
# Grant user read access
gsutil iam ch user:EMAIL:objectViewer gs://BUCKET_NAME

# Grant user write access
gsutil iam ch user:EMAIL:objectEditor gs://BUCKET_NAME

# Grant service account access
gsutil iam ch serviceAccount:SA@PROJECT.iam.gserviceaccount.com:objectViewer gs://BUCKET_NAME

# Remove user access
gsutil iam ch -d user:EMAIL gs://BUCKET_NAME
```

## Versioning

### Enable Versioning
```bash
# Enable object versioning on bucket
gsutil versioning set on gs://BUCKET_NAME

# Disable versioning
gsutil versioning set off gs://BUCKET_NAME

# Check versioning status
gsutil versioning get gs://BUCKET_NAME
```

### Working with Versions
```bash
# List all versions
gsutil ls -a gs://BUCKET_NAME

# Download specific version
gsutil cp gs://BUCKET_NAME/OBJECT_NAME#VERSION_ID LOCAL_FILE

# Delete specific version
gsutil rm gs://BUCKET_NAME/OBJECT_NAME#VERSION_ID
```

## Lifecycle Management

### Set Lifecycle Policy
```bash
# Create lifecycle configuration
cat > lifecycle.json <<EOF
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "Delete"},
        "condition": {"age": 30}
      }
    ]
  }
}
EOF

# Apply lifecycle policy
gsutil lifecycle set lifecycle.json gs://BUCKET_NAME

# View lifecycle policy
gsutil lifecycle get gs://BUCKET_NAME
```

## Synchronization

### Sync Local to Cloud
```bash
# Sync local directory to bucket
gsutil -m rsync -r LOCAL_DIR gs://BUCKET_NAME

# Sync with deletion (-d flag)
gsutil -m rsync -r -d LOCAL_DIR gs://BUCKET_NAME

# Sync without deleting destination
gsutil -m rsync -r -n LOCAL_DIR gs://BUCKET_NAME  # Dry run
```

### Sync Cloud to Local
```bash
# Sync bucket to local directory
gsutil -m rsync -r gs://BUCKET_NAME LOCAL_DIR

# Dry run (shows what would happen)
gsutil -m rsync -r -n gs://BUCKET_NAME LOCAL_DIR
```

## Performance Optimization

### Parallel Operations
```bash
# Set parallel thread count
gsutil -m -o GSUtil:parallel_thread_count=24 cp -r LOCAL_DIR gs://BUCKET_NAME

# Parallel composite uploads (for large files)
gsutil -o GSUtil:parallel_composite_upload_threshold=100M cp large-file.zip gs://BUCKET_NAME

# Set max retries
gsutil -o GSUtil:max_retry_delay=60 cp LOCAL_FILE gs://BUCKET_NAME
```

## Troubleshooting

### Issue: "AccessDenied" or "Permission denied"
**Solution:**
```bash
# Check current credentials
gsutil -D ls gs://BUCKET_NAME 2>&1 | head -20

# Authenticate again
gcloud auth login

# Check bucket permissions
gsutil iam get gs://BUCKET_NAME
```

### Issue: "Bucket not found"
**Solution:**
```bash
# List buckets you have access to
gsutil ls

# Check bucket name is correct
# Bucket names are globally unique and lowercase
```

### Issue: "Quota exceeded"
**Solution:**
```bash
# Reduce parallel threads
gsutil -o GSUtil:parallel_thread_count=4 cp file gs://BUCKET_NAME

# Or spread uploads over time
gsutil cp file1 gs://BUCKET_NAME && sleep 5 && gsutil cp file2 gs://BUCKET_NAME
```

## Related Documentation

- [Configuration Management](gcp-cli-configuration.md) - gsutil configuration
- [Authentication](gcp-cli-authentication.md) - Set up credentials
- [IAM](gcp-cli-iam.md) - Manage bucket permissions
