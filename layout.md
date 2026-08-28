# Ansible Windows AD Project Layout

## Repository Structure

```text
ansible-windows-ad/
├── inventory/
│   ├── group_vars/
│   ├── production/
│   │   ├── group_vars/
│   │   ├── host_vars/
│   │   └── hosts.yml
│   └── lab/
│       ├── group_vars/
│       ├── host_vars/
│       └── hosts.yml
├── playbooks/
│   ├── certificates.yml
│   ├── computers.yml
│   ├── delegation.yml
│   ├── dns.yml
│   ├── domain.yml
│   ├── gmsa.yml
│   ├── gpo.yml
│   ├── groups.yml
│   ├── health_checks.yml
│   ├── ous.yml
│   ├── password_policies.yml
│   ├── service_accounts.yml
│   ├── sites.yml
│   ├── spns.yml
│   └── users.yml
├── roles/
│   ├── certificates/
│   ├── computers/
│   ├── delegation/
│   ├── dns/
│   ├── domain/
│   ├── gmsa/
│   ├── gpo/
│   ├── groups/
│   ├── health_checks/
│   ├── ous/
│   ├── password_policies/
│   ├── service_accounts/
│   ├── sites/
│   ├── spns/
│   └── users/
├── .gitignore
└── README.md
```

## Role Execution Order

The intended execution order is shown below. Windows baseline and additional domain controller automation are planned stages; the remaining role directories are present in the repository.

1. Windows baseline
2. Domain and forest
3. Domain controllers
4. DNS
5. Organizational units (OUs)
6. Groups
7. Users
8. Computers
9. Service accounts
10. Group Managed Service Accounts (GMSAs)
11. Password policies
12. Group Policy Objects (GPOs)
13. Delegation
14. Sites and subnets
15. Service Principal Names (SPNs)
16. Certificates
17. Health checks

## Role Responsibilities

| Role | Responsibility |
| --- | --- |
| `domain` | Install AD DS and create or configure the forest and domain. |
| `ous` | Create and verify the OU hierarchy. |
| `users` | Create and manage user accounts. |
| `groups` | Create groups and manage group memberships. |
| `computers` | Manage computer accounts. |
| `service_accounts` | Create and manage service accounts. |
| `gmsa` | Configure Group Managed Service Accounts. |
| `password_policies` | Configure domain and fine-grained password policies. |
| `gpo` | Create, link, and configure Group Policy Objects. |
| `delegation` | Configure delegated permissions in Active Directory. |
| `dns` | Configure and verify Active Directory-integrated DNS. |
| `sites` | Configure sites, subnets, and site links. |
| `spns` | Manage Service Principal Names. |
| `certificates` | Manage certificate-related configuration. |
| `health_checks` | Verify domain services, DNS, and replication health. |

## Domain Role Inputs

The `domain` role accepts these primary variables:

```yaml
domain_name: "example.com"
netbios_name: "EXAMPLE"
domain_mode: "10"
forest_mode: "10"
```

Use environment-specific values in the appropriate inventory variable files. Keep passwords and other secrets in Ansible Vault-protected files.

## Domain Task Flow

```text
playbooks/domain.yml
└── roles/domain/tasks/main.yml
    ├── prerequisites.yml
    ├── install_ad_ds.yml
    ├── create_forest.yml
    ├── configure_domain.yml
    └── verify.yml
```

The domain role is responsible for AD DS prerequisites, initial forest creation, domain-level configuration, functional levels, and domain verification.
