# Ansible Windows AD Automation

A comprehensive Ansible automation framework for managing and configuring Windows Active Directory environments.

## Project Overview

This project provides a modular and scalable approach to automating Active Directory deployment, configuration, and management across Windows Server environments. It includes roles for domain setup, organizational units, users, groups, policies, and more.

## Directory Structure

```
ansible-windows-ad/
├── inventory/                    # Ansible inventory configuration
│   ├── production/              # Production environment
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   └── host_vars/
│   ├── lab/                     # Lab/test environment
│   │   ├── hosts.yml
│   │   ├── group_vars/
│   │   └── host_vars/
│   └── group_vars/              # Global group variables
│
├── playbooks/                    # Ansible playbooks
│   ├── domain.yml               # Domain configuration
│   ├── ous.yml                  # Organizational Units
│   ├── users.yml                # User management
│   ├── groups.yml               # Group management
│   ├── computers.yml            # Computer management
│   ├── service_accounts.yml     # Service accounts
│   ├── gmsa.yml                 # Group Managed Service Accounts
│   ├── password_policies.yml    # Password policies
│   ├── gpo.yml                  # Group Policy Objects
│   ├── delegation.yml           # AD delegation
│   ├── dns.yml                  # DNS configuration
│   ├── sites.yml                # Sites and subnets
│   ├── spns.yml                 # Service Principal Names
│   ├── certificates.yml         # Certificate management
│   └── health_checks.yml        # Health verification
│
├── roles/                        # Ansible roles
│   ├── domain/                  # Domain setup and configuration
│   ├── ous/                     # OU creation and management
│   ├── users/                   # User account management
│   ├── groups/                  # Group management
│   ├── computers/               # Computer management
│   ├── service_accounts/        # Service accounts
│   ├── gmsa/                    # GMSA configuration
│   ├── password_policies/       # Password policy settings
│   ├── gpo/                     # GPO management
│   ├── delegation/              # AD delegation
│   ├── dns/                     # DNS setup
│   ├── sites/                   # Sites configuration
│   ├── spns/                    # SPN management
│   ├── certificates/            # Certificate handling
│   └── health_checks/           # Health checks and verification
│
├── .gitignore                    # Git ignore rules
├── layout.md                     # Project layout documentation
└── README.md                     # This file

```

## Execution Order

The roles should be executed in the following order:

1. **Windows Baseline** - OS-level prerequisites
2. **Domain** - Forest and domain creation
3. **Domain Controllers** - Additional DC setup
4. **DNS** - DNS configuration
5. **OUs** - Organizational Unit hierarchy
6. **Groups** - Group creation
7. **Users** - User account creation
8. **Computers** - Computer management
9. **Service Accounts** - Service account setup
10. **GPO** - Group Policy configuration
11. **Delegation** - AD delegation setup
12. **Sites/Subnets** - Site configuration
13. **SPNs** - Service Principal Names
14. **Health Checks** - Verification and health checks

## Prerequisites

- Ansible 2.9+
- Python 3.7+
- Windows Server 2016 or later
- PowerShell 5.0+ on target hosts
- Network connectivity to target hosts
- Administrative credentials for target domain

## Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd ansible-windows-ad
   ```

2. **Install Ansible**
   ```bash
   pip install ansible
   ```

3. **Install required Python packages**
   ```bash
   pip install pywinrm
   ```

4. **Configure inventory** - Update inventory files with your environment details
   ```bash
   # Edit production inventory
   vim inventory/production/hosts.yml
   vim inventory/production/group_vars/domain.yml
   ```

## Configuration

### Inventory Setup

1. **Production Environment** (`inventory/production/`)
   - Configure production domain controllers and servers
   - Update `hosts.yml` with actual hostnames/IPs
   - Set domain variables in `group_vars/domain.yml`

2. **Lab Environment** (`inventory/lab/`)
   - Configure lab/test domain controllers
   - Update `hosts.yml` with lab hostnames/IPs
   - Set lab domain variables in `group_vars/domain.yml`

### Variables

Each role has default variables in `defaults/main.yml` and role-specific variables in `vars/main.yml`.

Example domain role inputs:
```yaml
domain_name: "example.com"
netbios_name: "EXAMPLE"
domain_mode: "10"
forest_mode: "10"
```

## Usage

### Run entire domain setup
```bash
ansible-playbook playbooks/domain.yml -i inventory/production
```

### Run specific playbook
```bash
ansible-playbook playbooks/ous.yml -i inventory/production
```

### Run with tags
```bash
ansible-playbook playbooks/users.yml -i inventory/production --tags "create"
```

### Create and use Ansible Vault variables

Use Ansible Vault to protect passwords and other sensitive values. The repository ignores files named `vault.yml`, so do not commit an unencrypted vault file.

1. **Create a vault file** for the environment you will use:

   ```bash
   ansible-vault create inventory/lab/group_vars/vault.yml
   # or
   ansible-vault create inventory/production/group_vars/vault.yml
   ```

   Add variables in YAML format, for example:

   ```yaml
   ansible_user: Administrator
   ansible_password: "replace-with-a-secret"
   ansible_winrm_transport: ntlm
   ```

2. **Encrypt an existing file** if it was created as plain text:

   ```bash
   ansible-vault encrypt inventory/lab/group_vars/vault.yml
   ```

3. **Edit an encrypted file** without decrypting it permanently:

   ```bash
   ansible-vault edit inventory/lab/group_vars/vault.yml
   ```

4. **Run a playbook and provide the vault password interactively**:

   ```bash
   ansible-playbook playbooks/domain.yml -i inventory/lab --ask-vault-pass
   ```

5. **Use a password file in automation**. Store the file outside the repository and restrict its permissions:

   ```bash
   ansible-playbook playbooks/domain.yml \
     -i inventory/production \
     --vault-password-file C:\\secure\\ansible-vault-password.txt
   ```

6. **Inspect or rotate the vault password**:

   ```bash
   ansible-vault view inventory/lab/group_vars/vault.yml
   ansible-vault rekey inventory/lab/group_vars/vault.yml
   ```

Never place vault passwords, private keys, or unencrypted credentials in the repository. If a secret is exposed, rotate it immediately and remove it from Git history.

## Security Best Practices

1. **Vault for Sensitive Data**
   - Store passwords and sensitive credentials in vault files
   - Use `ansible-vault` to encrypt sensitive files
   - Never commit unencrypted credentials

2. **Credential Management**
   - Use Ansible credential files or environment variables
   - Implement principle of least privilege
   - Regular audit of access permissions

3. **Network Security**
   - Use WinRM over HTTPS (port 5986)
   - Restrict network access to domain controllers
   - Implement VPN or bastion hosts for remote access

## Troubleshooting

### Common Issues

**WinRM Connection Issues**
```bash
# Test WinRM connectivity
ansible all -i inventory/production -m win_ping
```

**Credential Problems**
- Verify credentials in inventory files
- Check AD user permissions
- Ensure WinRM is enabled on target hosts

**DNS Resolution**
- Verify DNS entries in inventory
- Test connectivity: `ansible-inventory -i inventory/production --list`

## Contributing

1. Create a feature branch
2. Make changes in a modular way
3. Test in lab environment first
4. Commit with descriptive messages
5. Submit pull request

## Support & Documentation

- Ansible Documentation: https://docs.ansible.com/
- Windows Modules: https://docs.ansible.com/ansible/latest/collections/ansible/windows/
- Active Directory PowerShell: https://docs.microsoft.com/en-us/powershell/module/addsadministration/

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Changelog

### v1.0.0 (2025-08-25)
- Initial project structure
- 15 core roles for AD automation
- Production and lab inventory templates
- Comprehensive playbook suite
