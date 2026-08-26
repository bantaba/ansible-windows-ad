ansible/
├── inventory/
├── playbooks/
│   └── domain.yml
├── roles/
│   ├── ad_ous/
│   ├── ad_users/
│   ├── ad_groups/
│   ├── ad_computers/
│   ├── ad_service_accounts/
│   ├── ad_gpo/
│   ├── ad_dns/
│   ├── ad_sites/
│   └── ad_delegation/
└── vars/
    └── domain.yml


ad-automation/
│
├── inventory/
│   ├── production/
│   └── lab/
│
├── group_vars/
│   └── domain.yml
│
├── roles/
│   ├── domain/
│   ├── ous/
│   ├── users/
│   ├── groups/
│   ├── computers/
│   ├── service_accounts/
│   ├── gmsa/
│   ├── password_policies/
│   ├── gpo/
│   ├── delegation/
│   ├── dns/
│   ├── sites/
│   ├── spns/
│   ├── certificates/
│   └── health_checks/
│
└── playbooks/
    ├── domain.yml
    ├── identities.yml
    ├── computers.yml
    ├── policies.yml
    ├── dns.yml
    └── health.yml    

roles/
└── domain/
    ├── defaults/
    │   └── main.yml
    ├── vars/
    │   └── main.yml
    ├── tasks/
    │   ├── main.yml
    │   ├── prerequisites.yml
    │   ├── install_ad_ds.yml
    │   ├── create_forest.yml
    │   ├── configure_domain.yml
    │   └── verify.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    ├── files/
    └── meta/
        └── main.yml
roles/
├── domain/
│   ├── defaults/main.yml
│   └── tasks/...
│
├── ous/
│   ├── defaults/main.yml
│   └── tasks/...
│
├── users/
│   ├── defaults/main.yml
│   └── tasks/...
│
├── groups/
│   ├── defaults/main.yml
│   └── tasks/...
│
└── gpo/
    ├── defaults/main.yml
    └── tasks/...
    
---
01 windows_baseline
        ↓
02 domain
        ↓
03 domain_controllers
        ↓
04 dns
        ↓
05 ous
        ↓
06 groups
        ↓
07 users
        ↓
08 computers
        ↓
09 service_accounts
        ↓
10 gpo
        ↓
11 delegation
        ↓
12 sites/subnets
        ↓
13 spns
        ↓
14 health_checks
---

Reponsible:
    Forest
    Domain
    Domain-level configuration
    Functional levels
    AD DS prerequisites
    Initial forest creation
    Domain verification


---

domain role
    inputs:
        domain_name
        netbios_name
        domain_mode
        forest_mode

    does:
        install/configure AD DS
---

Installing AD DS on additional servers
Promoting additional DCs
Adding DCs to sites
DNS installation on DCs
Global Catalog configuration
FSMO roles
Replication verification
DC demotion


---
groups role
    inputs:
        ad_groups
        ad_group_memberships

    does:
        create groups
        manage membership

---
ous role
    inputs:
        ad_ous

    does:
        create OU hierarchy        
01 windows_baseline
        ↓
02 domain
        ↓
03 domain_controllers
        ↓
04 dns
        ↓
05 ous
        ↓
06 groups
        ↓
07 users
        ↓
08 computers
        ↓
09 service_accounts
        ↓
10 gpo
        ↓
11 delegation
        ↓
12 sites/subnets
        ↓
13 spns
        ↓
14 health_checks    



playbooks/domain.yml
        │
        │ roles:
        ▼
     domain
        │
        ▼
roles/domain/tasks/main.yml
        │
        ├── prerequisites.yml
        ├── install_ad_ds.yml
        ├── create_forest.yml
        ├── configure_domain.yml
        └── verify.yml