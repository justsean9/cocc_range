# Attack Range Container Deployment Guide for Coworkers

This guide provides step-by-step instructions for deploying the Attack Range with SentinelOne support using Docker. You can optionally provide bad files for testing that will be referenced from outside the container.

## Overview

This containerized Attack Range allows you to:
- Deploy AWS infrastructure with SentinelOne pre-installed
- Optionally inject bad files for testing (stored outside the container)
- Run everything in a containerized environment for easy deployment

## Prerequisites

### For All Users:
- AWS Account with appropriate permissions
- AWS Access Key ID and Secret Access Key
- SentinelOne Customer ID (obtain from your SentinelOne admin)
- SentinelOne Windows Agent (WindowsSensor.exe)

### For Windows Users:
- Windows 10/11 Pro or Enterprise (for Docker Desktop)
- WSL2 installed (required for Docker Desktop)
- Docker Desktop for Windows

### For Mac Users:
- macOS 10.15 or later
- Docker Desktop for Mac

## Step-by-Step Instructions

### Part 1: Windows Setup

1. **Install Docker Desktop**
   - Download Docker Desktop from https://www.docker.com/products/docker-desktop
   - Run the installer and follow the prompts
   - Ensure WSL2 is enabled when prompted
   - Restart your computer after installation

2. **Set up project directory**
   ```powershell
   # Create a workspace
   mkdir C:\attack-range-deployment
   cd C:\attack-range-deployment
   
   # Clone the repository
   git clone https://github.com/splunk/attack_range.git
   cd attack_range
   ```

3. **Prepare your files**
   ```powershell
   # Copy SentinelOne installer to the existing apps folder
   copy "C:\Path\To\Your\WindowsSensor.exe" apps\
   
   # Copy any bad files to the existing badfiles folder (optional)
   copy "C:\Path\To\Your\BadFiles\*" badfiles\
   
   # Copy any sample files to the existing sample_files folder (optional)
   copy "C:\Path\To\Your\SampleFiles\*" sample_files\
   ```

4. **Create configuration file**
   ```powershell
   # Copy the default config
   copy configs\attack_range_default.yml configs\my_attack_range.yml
   
   # Edit the configuration (using notepad or your preferred editor)
   notepad configs\my_attack_range.yml
   ```

5. **Update the configuration** (in my_attack_range.yml):
   ```yaml
   general:
     attack_range_password: "YourSecurePassword123!"
     cloud_provider: "aws"
     key_name: "your-aws-key-pair"
     attack_range_name: "myrange"
     ip_whitelist: "YOUR.PUBLIC.IP.ADDRESS/32"
     sentinelone_customer_ID: "YOUR-S1-CUSTOMER-ID"
   
   windows_servers_default:
     install_sentinelone: "1"
     sentinelone_windows_agent: "WindowsSensor.exe"
   ```

6. **Build the Docker image**
   ```powershell
   # Build the Docker image locally
   docker build -t my-attack-range .
   ```

7. **Run the container**
   ```powershell
   # Run the container with volume mounts for the existing directories
   docker run -it `
     -v ${PWD}/configs:/attack_range/configs `
     -v ${PWD}/apps:/attack_range/apps `
     -v ${PWD}/badfiles:/attack_range/badfiles `
     -v ${PWD}/sample_files:/attack_range/sample_files `
     -e AWS_ACCESS_KEY_ID="your-access-key" `
     -e AWS_SECRET_ACCESS_KEY="your-secret-key" `
     -e AWS_DEFAULT_REGION="us-west-2" `
     my-attack-range
   ```

### Part 2: Mac Setup

1. **Install Docker Desktop**
   ```bash
   # Download and install Docker Desktop from https://www.docker.com/products/docker-desktop
   # Or use Homebrew:
   brew install --cask docker
   ```

2. **Set up project directory**
   ```bash
   # Create a workspace
   mkdir ~/attack-range-deployment
   cd ~/attack-range-deployment
   
   # Clone the repository
   git clone https://github.com/splunk/attack_range.git
   cd attack_range
   ```

3. **Prepare your files**
   ```bash
   # Copy SentinelOne installer to the existing apps folder
   cp ~/Downloads/WindowsSensor.exe apps/
   
   # Copy any bad files to the existing badfiles folder (optional)
   cp ~/YourBadFiles/* badfiles/
   
   # Copy any sample files to the existing sample_files folder (optional)
   cp ~/YourSampleFiles/* sample_files/
   ```

4. **Create configuration file**
   ```bash
   # Copy the default config
   cp configs/attack_range_default.yml configs/my_attack_range.yml
   
   # Edit the configuration
   vim configs/my_attack_range.yml
   # or use nano/your preferred editor
   ```

5. **Update the configuration** (same as Windows step 5)

6. **Build the Docker image**
   ```bash
   # Build the Docker image locally
   docker build -t my-attack-range .
   ```

7. **Run the container**
   ```bash
   # Run the container with volume mounts for the existing directories
   docker run -it \
     -v $(pwd)/configs:/attack_range/configs \
     -v $(pwd)/apps:/attack_range/apps \
     -v $(pwd)/badfiles:/attack_range/badfiles \
     -v $(pwd)/sample_files:/attack_range/sample_files \
     -e AWS_ACCESS_KEY_ID="your-access-key" \
     -e AWS_SECRET_ACCESS_KEY="your-secret-key" \
     -e AWS_DEFAULT_REGION="us-west-2" \
     my-attack-range
   ```

### Part 3: Using the Attack Range

Once inside the container, follow these steps:

1. **Configure AWS credentials**
   ```bash
   # The AWS credentials are already set via environment variables
   # Verify with:
   aws configure list
   ```

2. **Deploy the Attack Range**
   ```bash
   # Use your custom config
   python attack_range.py build -c configs/my_attack_range.yml
   ```

3. **Check the deployment status**
   ```bash
   python attack_range.py show -c configs/my_attack_range.yml
   ```

4. **Perform attack simulations**
   ```bash
   # Example: Run a specific technique
   python attack_range.py simulate -e ART -te T1003.001 -t ar-win-0 -c configs/my_attack_range.yml
   ```

5. **Destroy the range when done**
   ```bash
   python attack_range.py destroy -c configs/my_attack_range.yml
   ```

## Important Notes

### File Management
- **apps/**: Place your SentinelOne installer (WindowsSensor.exe) here
- **badfiles/**: Place malware samples for testing here (optional)
- **sample_files/**: Place benign sample files here (optional)
- All these directories already exist in the repository structure

### Configuration Best Practices
1. Always use strong passwords
2. Restrict IP whitelist to your specific IP or range
3. Keep your SentinelOne Customer ID secure
4. Use unique key names for different deployments

### Volume Mounts Explained
- `/attack_range/configs`: Your custom configuration files
- `/attack_range/apps`: Application installers (SentinelOne, etc.)
- `/attack_range/badfiles`: Bad files for testing (optional)
- `/attack_range/sample_files`: Sample files for testing (optional)

### Troubleshooting

**Docker Issues on Windows:**
- Ensure WSL2 is properly installed
- Check that virtualization is enabled in BIOS
- Run Docker Desktop as administrator if needed

**Docker Issues on Mac:**
- Ensure Docker Desktop has sufficient resources (CPU/Memory)
- Check System Preferences > Security & Privacy for permissions

**AWS Issues:**
- Verify AWS credentials are correct
- Check AWS service quotas (especially security groups)
- Ensure your IP is whitelisted in the configuration

**Container Issues:**
- Use `docker logs [container-id]` to check for errors
- Ensure all required files are in the correct directories
- Verify volume mounts are correctly specified

## Security Considerations

1. **Bad Files Handling**:
   - Store bad files in the `badfiles/` directory
   - Use proper access controls on the directory
   - Never commit bad files to the git repository
   - Delete bad files after testing

2. **Credentials Management**:
   - Never hardcode credentials in configuration files
   - Use environment variables for sensitive data
   - Rotate AWS keys regularly
   - Keep SentinelOne Customer ID confidential

3. **Network Security**:
   - Always use specific IP whitelisting
   - Don't use 0.0.0.0/0 for production deployments
   - Monitor AWS costs and usage
   - Destroy ranges when not in use

## Directory Structure Overview

```
attack_range/
├── apps/                    # Place SentinelOne installer here
│   └── WindowsSensor.exe
├── badfiles/               # Place malware samples here (optional)
├── sample_files/           # Place benign samples here (optional)
├── configs/                # Configuration files
│   ├── attack_range_default.yml
│   └── my_attack_range.yml # Your custom config
├── terraform/              # Infrastructure as code
├── modules/                # Python modules
└── Dockerfile              # Container definition
```

## Quick Reference

### Windows Commands
```powershell
# Build image
docker build -t my-attack-range .

# Run container
docker run -it `
  -v ${PWD}/configs:/attack_range/configs `
  -v ${PWD}/apps:/attack_range/apps `
  -v ${PWD}/badfiles:/attack_range/badfiles `
  -v ${PWD}/sample_files:/attack_range/sample_files `
  -e AWS_ACCESS_KEY_ID="your-key" `
  -e AWS_SECRET_ACCESS_KEY="your-secret" `
  my-attack-range
```

### Mac/Linux Commands
```bash
# Build image
docker build -t my-attack-range .

# Run container
docker run -it \
  -v $(pwd)/configs:/attack_range/configs \
  -v $(pwd)/apps:/attack_range/apps \
  -v $(pwd)/badfiles:/attack_range/badfiles \
  -v $(pwd)/sample_files:/attack_range/sample_files \
  -e AWS_ACCESS_KEY_ID="your-key" \
  -e AWS_SECRET_ACCESS_KEY="your-secret" \
  my-attack-range
```

## Support

For issues or questions:
1. Check the main Attack Range documentation
2. Review AWS CloudTrail logs for deployment issues
3. Contact your security team for SentinelOne configuration
4. Submit issues to the GitHub repository

Remember to always destroy your attack range when finished to avoid unnecessary AWS charges!