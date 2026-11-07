# Automated Compliance Enforcement with Ansible

This project provides a reusable Ansible automation suite that audits and remediates Linux hardening controls aligned with CIS-style practices. The playbook checks security posture across a fleet of servers and produces a JSON compliance report for each host.

## Controls covered

The suite includes the following compliance checks and remediations:

1. SSH root login disabled
2. SSH password authentication disabled
3. SSH MaxAuthTries enforced to 4
4. UFW enabled with default deny incoming
5. Accounts with empty passwords blocked
6. Unauthorized sudo access reviewed and removed
7. Sensitive file permissions enforced (`/etc/passwd`, `/etc/shadow`, SSH private keys)
8. Legacy services disabled (`telnet`, `rsh`, `rlogin`)
9. Security updates checked and installed
10. Logging service enabled and running
11. IP forwarding disabled
12. ICMP redirects disabled

## Repository layout

```text
.
├── ansible.cfg
├── .vault_pass
├── README.md
├── group_vars/
│   ├── all.yml
│   └── all/
│       └── vault.yml
├── inventory/
│   └── hosts.ini
├── playbooks/
│   └── compliance.yml
├── reports/
│   └── compliance_report.json
├── roles/
│   ├── compliance_checks/
│   │   ├── defaults/
│   │   └── tasks/
│   └── compliance_remediation/
│       ├── defaults/
│       └── tasks/

```

## Prerequisites

- Ansible 2.14+ installed on the control node
- SSH or WinRM access to managed Linux hosts
- Root or sudo privileges on managed servers
- A vault password file for encrypted secrets

## Configure inventory

Update the sample inventory in `inventory/hosts.ini` with your managed hosts:

```ini
[linux_servers]
web01 ansible_host=10.0.0.11 ansible_user=admin ansible_become=yes
app01 ansible_host=10.0.0.12 ansible_user=admin ansible_become=yes
```

## Vault usage

Sensitive values are stored in `group_vars/all/vault.yml` and protected with Ansible Vault. To encrypt the file after editing:

```bash
ansible-vault encrypt group_vars/all/vault.yml
```

Use the included `.vault_pass` file for local automation or create a custom password file in a secure location:

```bash
ansible-vault decrypt group_vars/all/vault.yml --vault-password-file .vault_pass
```

## Run an audit only

This mode reports on compliance without modifying the system:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/compliance.yml --check
```

## Run remediation

This mode applies the remediations required by the compliance checks:

```bash
ansible-playbook -i inventory/hosts.ini playbooks/compliance.yml -e compliance_remediate=true
```

## Report generation

A JSON report is generated in `reports/compliance_report.json` after the playbook runs. The file contains the compliance status for each host and every control.

Example structure:

```json
{
  "generated_at": "2026-11-07T15:30:00Z",
  "hosts": [
    {
      "host": "server01",
      "controls": [
        { "control": "ssh_root_login", "status": "pass" },
        { "control": "ufw_default_deny", "status": "fail" }
      ]
    }
  ]
}
```

## Notes

- All tasks are structured to be idempotent.
- The suite supports both compliance-only and remediation modes.
- The playbook can be extended with additional CIS Level 1 controls or alert integrations.
