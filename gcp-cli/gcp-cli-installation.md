# GCP CLI Installation & Setup

Complete guide to installing and verifying the Google Cloud SDK (`gcloud`) CLI.

## Prerequisites

- Python 3.7 or later installed on your system
- pip (Python package manager)
- For Linux: curl or wget

## Installation Methods

### Option 1: Google Cloud SDK Bundle (Recommended)

#### On macOS (using Homebrew)
```bash
brew install --cask google-cloud-sdk
```

#### On Linux - Using curl
```bash
# Download the SDK
curl https://sdk.cloud.google.com | bash

# Restart your shell
exec -l $SHELL

# Initialize gcloud
gcloud init
```

#### On Linux - Using apt (Debian/Ubuntu)
```bash
# Add Google Cloud repo
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list

# Import Google Cloud public key
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key --keyring /usr/share/keyrings/cloud.google.gpg add -

# Update and install
sudo apt-get update && sudo apt-get install google-cloud-sdk

# Initialize
gcloud init
```

#### On Windows
1. Download the installer from [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
2. Run the installer
3. Follow the setup wizard
4. Run `gcloud init` in PowerShell or Command Prompt

### Option 2: Using pip
```bash
pip install google-cloud-sdk
gcloud init
```

## Verify Installation

```bash
# Check gcloud version
gcloud --version

# Verify authentication
gcloud auth list

# Check active project
gcloud config list project
```

## Initial Setup (gcloud init)

### Interactive Setup (Recommended for first-time users)
```bash
# Start interactive setup
gcloud init
```

This will:
- Login to your Google account
- Select or create a project
- Set default compute zone/region
- Configure Application Default Credentials (if needed)

### Non-Interactive Setup
```bash
# Skip interactive mode
gcloud init --skip-diagnostics

# Then manually set values
gcloud config set project PROJECT_ID
gcloud config set compute/zone us-west1-a
gcloud config set compute/region us-west1
```

## First Configuration

### 1. Set Default Project
```bash
# Set default project
gcloud config set project PROJECT_ID

# Verify
gcloud config list project
```

### 2. Set Default Compute Zone/Region
```bash
# Set default zone
gcloud config set compute/zone us-west1-a

# Set default region
gcloud config set compute/region us-west1

# Verify
gcloud config list compute
```

### 3. Install Essential Components
```bash
# Update all components
gcloud components update

# Install GKE authentication plugin (if working with GKE)
gcloud components install gke-gcloud-auth-plugin

# List available components
gcloud components list
```

## Troubleshooting Installation

### Issue: gcloud command not found
**Solution:**
```bash
# Check if gcloud is in PATH
which gcloud

# If not found, restart your shell
exec -l $SHELL

# Or manually source the gcloud binaries
source ~/.bashrc  # or ~/.zshrc
```

### Issue: Python version error
**Solution:**
```bash
# Check your Python version
python3 --version

# If below 3.7, install a newer version
brew install python3  # macOS
sudo apt-get install python3.9  # Ubuntu
```

### Issue: Permission denied errors
**Solution:**
```bash
# Run as non-root (recommended)
# If you installed with sudo, reinstall without sudo

# Or fix permissions
sudo chown -R $USER ~/.config/gcloud
sudo chown -R $USER ~/.local/share/gcloud
```

## Updating gcloud

```bash
# Update all components
gcloud components update

# Force update
gcloud components update --force-update
```

## Uninstalling gcloud

### On macOS (Homebrew)
```bash
brew uninstall google-cloud-sdk
```

### On Linux
```bash
# Manual installation - remove directory
rm -r ~/google-cloud-sdk

# APT installation
sudo apt-get remove google-cloud-sdk
```

### On Windows
1. Open Control Panel
2. Click "Uninstall a program"
3. Find and select "Google Cloud SDK"
4. Click "Uninstall"

## Next Steps

- [Authentication](gcp-cli-authentication.md) - Login and manage credentials
- [Configuration Management](gcp-cli-configuration.md) - Configure gcloud settings
- [Projects](gcp-cli-projects.md) - Manage GCP projects
