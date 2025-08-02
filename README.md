<div align="center">
  <img src="https://github.com/ansible/logos/raw/main/community-marks/Ansible-Community-Mark-Mango.png" alt="Ansible Logo" width="200">
  
  # Ansible Certificate Deployment
  
  **Automated deployment of PFX certificates from Azure Key Vault to Windows servers using Ansible.**
</div>

## 🎯 What This Does

- Downloads certificates from Azure Key Vault
- Securely transfers and installs them on multiple Windows servers
- Handles credentials without logging sensitive data
- Provides comprehensive error handling and cleanup

## ✨ Key Features

- **Secure**: No credentials stored in files or logs (`no_log: true` on all sensitive tasks)
- **Scalable**: Deploy to multiple servers simultaneously
- **Reliable**: Comprehensive error handling and rollback
- **Clean**: Automatic cleanup of temporary files
- **Interactive**: Secure credential prompts with hidden input
- **Idempotent**: Safe to run multiple times

## 🏗️ Architecture

- **Control Node**: CentOS server running Ansible
- **Target Servers**: Windows servers with WinRM enabled
- **Certificate Source**: Azure Key Vault
- **Communication**: Secure WinRM over encrypted channels

## 🔄 Process Flow

```mermaid
flowchart TD
    A[User runs: ansible-playbook -i inventory.ini deploy-certificate.yml] --> B[Ansible reads inventory.ini]
    B --> C[Ansible loads group_vars/windows_servers.yml]
    C --> D[🔐 Prompt for Azure Client ID]
    D --> E[🔐 Prompt for Azure Secret - Hidden Input]
    E --> F[🔐 Prompt for Azure Tenant ID]
    F --> G[🔐 Prompt for Certificate Password - Hidden Input]
    
    G --> H[🔄 Begin loop: For each server in windows_servers group]
    H --> I[Connect to current Windows server via WinRM]
    
    I --> J{WinRM Connection Successful?}
    J -->|No| K[❌ Connection Failed - Check WinRM/Credentials]
    K --> L{More servers to process?}
    J -->|Yes| M[📁 Task 1: Create C:\temp directory on Windows]
    
    M --> N{Directory Created?}
    N -->|No| O[❌ Failed to create directory]
    O --> L
    N -->|Yes| P[☁️ Task 2: Connect to Azure Key Vault from CentOS]
    
    P --> Q{Azure Authentication Successful?}
    Q -->|No| R[❌ Azure auth failed - Check credentials]
    R --> L
    Q -->|Yes| S[📥 Download certificate from Key Vault]
    
    S --> T{Certificate Downloaded?}
    T -->|No| U[❌ Certificate not found or access denied]
    U --> L
    T -->|Yes| V[📤 Task 3: Transfer certificate to Windows server]
    
    V --> W{File Transfer Successful?}
    W -->|No| X[❌ File transfer failed]
    X --> L
    W -->|Yes| Y[🔧 Task 4: Install certificate in Windows cert store]
    
    Y --> Z{Certificate Installation Successful?}
    Z -->|No| AA[❌ Installation failed - Check password/format]
    AA --> L
    Z -->|Yes| BB[🗑️ Task 5: Delete temporary certificate file]
    
    BB --> CC{Cleanup Successful?}
    CC -->|No| DD[⚠️ Warning: Temp file not deleted]
    DD --> L
    CC -->|Yes| EE[✅ Task completed successfully for this server]
    
    EE --> L
    L -->|Yes| H
    L -->|No| FF[🎉 All servers processed - Playbook completed]
    
    classDef successClass fill:#d4edda,stroke:#28a745,stroke-width:2px
    classDef errorClass fill:#f8d7da,stroke:#dc3545,stroke-width:2px
    classDef warningClass fill:#fff3cd,stroke:#ffc107,stroke-width:2px
    classDef processClass fill:#e2e3e5,stroke:#6c757d,stroke-width:2px
    classDef inputClass fill:#cce5ff,stroke:#007bff,stroke-width:2px
    classDef loopClass fill:#e7f3ff,stroke:#0066cc,stroke-width:3px
    
    class FF successClass
    class K,O,R,U,X,AA errorClass
    class DD warningClass
    class A,B,C,I,M,P,S,V,Y,BB,EE processClass
    class D,E,F,G inputClass
    class H,L loopClass
```

## 🚀 Quick Start

### 1. Prerequisites

**On CentOS Control Node:**
```bash
# Install required packages
pip3 install azure-keyvault-secrets azure-identity pywinrm

# Install Ansible collections
ansible-galaxy collection install azure.azcollection
ansible-galaxy collection install ansible.windows
```

**On Windows Target Servers:**
```powershell
# Run as Administrator
winrm quickconfig -y
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
```

### 2. Setup Azure

- Create an App Registration in Azure AD
- Grant Key Vault access permissions
- Note: Client ID, Client Secret, and Tenant ID

### 3. Configure Repository

**Clone and setup:**
```bash
git clone <your-repo-url>
cd ansible-certificate-deployment
```

**Create inventory file:**
```ini
# inventory.ini
[windows_servers]
web-server-01 ansible_host=192.168.1.100 ansible_user=Administrator ansible_password=YourPassword
web-server-02 ansible_host=192.168.1.101 ansible_user=Administrator ansible_password=YourPassword

[windows_servers:vars]
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore
```

**Configure group variables:**
```yaml
# group_vars/windows_servers.yml
key_vault_name: "your-keyvault-name"
certificate_name: "your-certificate-name"
cert_store_name: "My"
cert_store_location: "LocalMachine"
temp_cert_path: "C:\\temp\\certificate.pfx"
```

### 4. Run the Playbook

```bash
# Test connectivity first
ansible windows_servers -i inventory.ini -m win_ping

# Deploy certificates
ansible-playbook -i inventory.ini deploy-certificate.yml
```

**You'll be prompted for:**
- Azure Client ID
- Azure Client Secret (hidden)
- Azure Tenant ID
- Certificate Password (hidden)

## 📁 Repository Structure

```
ansible-certificate-deployment/
├── README.md                      # This file
├── LICENSE                        # MIT License
├── deploy-certificate.yml         # Main playbook
├── inventory.ini                  # Server inventory (create this)
├── group_vars/
│   └── windows_servers.yml       # Group variables (create this)
├── host_vars/                     # Host-specific variables (optional)
├── docs/
│   ├── troubleshooting.md        # Common issues and solutions
│   └── security-considerations.md # Security best practices
└── examples/
    ├── inventory.ini.example     # Example inventory file
    └── group_vars.yml.example    # Example group variables
```

## ⚙️ Configuration Options

### Certificate Store Locations

| Store Name | Description | Location |
|------------|-------------|----------|
| `My` | Personal certificates | Current User or Local Machine |
| `Root` | Trusted Root CAs | Local Machine |
| `CA` | Intermediate CAs | Local Machine |
| `Trust` | Enterprise Trust | Local Machine |

### Common Variables

```yaml
# group_vars/windows_servers.yml
key_vault_name: "prod-certificates"           # Your Key Vault name
certificate_name: "wildcard-ssl-cert"         # Certificate name in Key Vault
cert_store_name: "My"                         # Certificate store
cert_store_location: "LocalMachine"           # Store location
temp_cert_path: "C:\\temp\\certificate.pfx"   # Temporary file path
```

## 🔒 Security Features

- **No credential storage**: All sensitive data prompted at runtime
- **Hidden input**: Passwords and secrets not displayed
- **No logging**: `no_log: true` prevents sensitive data in logs
- **Temporary files**: Automatic cleanup after installation
- **Encrypted transport**: WinRM over HTTPS (recommended)

## 🐛 Troubleshooting

### Common Issues

**WinRM Connection Failed:**
```bash
# Check WinRM configuration
winrm get winrm/config/service
```

**Azure Authentication Failed:**
- Verify App Registration permissions
- Check Client ID, Secret, and Tenant ID
- Ensure Key Vault access policies are set

**Certificate Installation Failed:**
- Verify PFX password
- Check certificate format
- Ensure sufficient permissions

### Debug Mode

```bash
# Run with verbose output
ansible-playbook -i inventory.ini deploy-certificate.yml -vvv
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📋 Requirements

### Control Node (CentOS)
- Ansible 2.9+
- Python 3.6+
- Azure SDK for Python
- pywinrm

### Target Servers (Windows)
- Windows Server 2012+
- WinRM enabled
- PowerShell 3.0+
- Network connectivity to control node

### Azure
- Azure Key Vault with certificates
- App Registration with Key Vault permissions
- Valid Azure subscription

## 📖 Documentation

- [Security Considerations](docs/security-considerations.md)
- [Troubleshooting Guide](docs/troubleshooting.md)
- [Advanced Configuration](docs/advanced-configuration.md)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Ansible community for excellent Windows modules
- Azure team for Key Vault integration
- Contributors and users of this project

## 📧 Support

- Create an [Issue](../../issues) for bug reports
- Start a [Discussion](../../discussions) for questions
- Check [existing issues](../../issues?q=is%3Aissue) before creating new ones

---

**⭐ If this project helped you, please give it a star!**
