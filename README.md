# Zabbix 7.0 LTS native HA on Ubuntu 24.04

This project deliberately separates prerequisite validation, installation, and post-install checks.

## Architecture assumption
- Two Zabbix server/frontend nodes use Zabbix native HA.
- HAProxy provides one web entry point.
- PostgreSQL is an existing highly available service exposed by a VIP or DNS name. The playbook validates reachability but does not build Patroni/etcd.
- TLS termination and firewall policy should be added according to the environment security standard.

## Before execution
1. Replace IP addresses and DNS names in `inventory.yml` and `group_vars/all.yml`.
2. Store `zabbix_db_password` with Ansible Vault, not plain text.
3. Create the PostgreSQL database and login before running this repository.
4. Confirm TCP 5432 from both Zabbix nodes to the database, TCP 10051 between monitored components and Zabbix nodes, and TCP 80/443 to the load balancer as required.

## Commands
```bash
ansible-inventory --graph
ansible-playbook 00_precheck.yml
ansible-playbook 10_install.yml
ansible-playbook 20_postcheck.yml
```

## Database preparation example
Run as a PostgreSQL administrator on the database service:
```sql
CREATE USER zabbix WITH PASSWORD 'use-a-secret';
CREATE DATABASE zabbix OWNER zabbix;
```

## Important production improvements
- Put the database password in Ansible Vault or an enterprise secret manager.
- Add TLS on HAProxy and validate certificates.
- Replace the simple HAProxy node health check with the organization-approved health policy.
- Back up the PostgreSQL database before upgrades.
- Test Zabbix node failover and database failover separately.

cd zabbix_ha_ansible

# Review inventory
ansible-inventory --graph

# Run prerequisite validation
ansible-playbook 00_precheck.yml

# Installation runs only when validation markers exist
ansible-playbook 10_install.yml

# Validate services, web frontend and native HA status
ansible-playbook 20_postcheck.yml

Prerequisite checks included
Ubuntu 24.04 or later
Supported CPU architecture
Minimum 2 vCPU
Minimum 4 GB RAM
Minimum 20 GB free root-disk space
Privilege escalation access
Time synchronization
PostgreSQL port connectivity
Zabbix repository availability
Frontend DNS resolution
Per-host validation marker
Important assumption

The package expects an existing HA PostgreSQL service. It does not build Patroni and etcd. This avoids presenting two Zabbix application nodes with a single database that would remain a hidden single point of failure. Before production use, replace all sample IP addresses, protect the database password with Ansible Vault, and add TLS on HAProxy.