# Ansible-Automation-Certificate-Deployment-To-Windows-Servers
Ansible automation for deploying PFX certificates from Azure Key Vault to Windows servers with secure credential handling and comprehensive error management.

# Ansible Certificate Deployment

**Automated deployment of PFX certificates from Azure Key Vault to Windows servers using Ansible.**

## 🎯 What This Does
- Downloads certificates from Azure Key Vault
- Securely transfers and installs them on multiple Windows servers
- Handles credentials without logging sensitive data
- Provides comprehensive error handling and cleanup

## ✨ Key Features
- **Secure**: No credentials stored in files or logs
- **Scalable**: Deploy to multiple servers simultaneously  
- **Reliable**: Comprehensive error handling and rollback
- **Clean**: Automatic cleanup of temporary files

## 🏗️ Architecture
- **Control Node**: CentOS server running Ansible
- **Target Servers**: Windows servers with WinRM enabled
- **Certificate Source**: Azure Key Vault
- **Communication**: Secure WinRM over encrypted channels
