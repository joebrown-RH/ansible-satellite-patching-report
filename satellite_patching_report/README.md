# Red Hat Satellite Patching Report Role

A comprehensive Ansible role that generates detailed patching reports from Red Hat Satellite, with multiple persistence options specifically designed for ephemeral execution environments (AAP/AWX).

## Features

- **Comprehensive Reporting**: Captures patching jobs, package updates, errata, content views, and more
- **Multiple Output Formats**: HTML (beautifully styled) and CSV (for spreadsheets)
- **Ephemeral Environment Support**: Multiple methods to persist reports from containers that don't persist
- **Flexible Persistence**: AAP artifacts, S3, NFS, Git, email, webhooks
- **Easy to Customize**: Template-based reports that can be modified to your needs

## What This Report Includes

✅ **Systems that were patched**  
✅ **Package updates** (previous version → new version)  
✅ **New packages added**  
✅ **Content view used**  
✅ **Who initiated the patching**  
✅ **Date and time of patching**  
✅ **Errata applied**  
✅ **Job status and results**

## Requirements

```yaml
collections:
  - ansible.builtin
  - community.general
  - amazon.aws (only if using S3 persistence)
  - ansible.posix (only if using NFS persistence)
```

Install required collections:
```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install amazon.aws  # if using S3
ansible-galaxy collection install ansible.posix  # if using NFS
```

## Quick Start

### Basic Usage (AAP/AWX)

```yaml
- hosts: localhost
  roles:
    - role: satellite_patching_report
      vars:
        satellite_server: "satellite.example.com"
        satellite_username: "admin"
        satellite_password: "{{ vault_satellite_password }}"
        report_days_back: 7
        use_aap_artifacts: true  # Reports automatically preserved in AAP
```

### With Email Delivery

```yaml
- hosts: localhost
  roles:
    - role: satellite_patching_report
      vars:
        satellite_server: "satellite.example.com"
        satellite_username: "admin"
        satellite_password: "{{ vault_satellite_password }}"
        report_format: "both"
        email_report: true
        email_to:
          - ops-team@example.com
        smtp_host: "smtp.example.com"
```

## 🔑 Solving the Ephemeral Execution Environment Problem

### The Problem

**Question**: *"How do you save files generated during an Ansible run when an ephemeral execution environment is doing the automation since the container does not persist?"*

**Answer**: When Ansible runs in an ephemeral execution environment (EE) like AAP/AWX or containerized environments, any files created during the run are lost when the container terminates. This role provides **five proven solutions** to this problem:

---

### Solution 1: AAP Artifact Directory (RECOMMENDED) ⭐

**How it works**: AAP/AWX automatically preserves files in the special artifact directory (`/runner/artifacts` or `$AWX_ISOLATED_DATA_DIR`) after job completion.

**Configuration**:
```yaml
use_aap_artifacts: true  # Default
```

**How to access**:
1. Navigate to AAP/AWX UI
2. Go to Jobs → [Your Job]
3. Click on the "Output" tab
4. Select the "Artifacts" tab
5. Download your reports

**Pros**: 
- ✅ No external dependencies
- ✅ Built into AAP/AWX
- ✅ Easy to access via UI
- ✅ Automatic cleanup based on retention policies

**Cons**:
- ❌ Only works in AAP/AWX environments
- ❌ Files eventually deleted based on retention settings

---

### Solution 2: Email Reports

**How it works**: Reports are emailed to specified recipients immediately after generation.

**Configuration**:
```yaml
email_report: true
email_to:
  - ops-team@example.com
  - manager@example.com
email_from: "ansible@example.com"
smtp_host: "smtp.example.com"
smtp_port: 587
```

**Pros**:
- ✅ Reports delivered to stakeholders automatically
- ✅ Email serves as permanent record
- ✅ Works in any environment

**Cons**:
- ❌ Requires SMTP access
- ❌ Large reports may hit email size limits

---

### Solution 3: S3 Storage

**How it works**: Reports are uploaded to an Amazon S3 bucket (or S3-compatible storage like MinIO, Ceph).

**Configuration**:
```yaml
persist_to_s3: true
s3_bucket: "my-satellite-reports"
s3_region: "us-east-1"
# AWS credentials via environment variables or instance role
```

**Pros**:
- ✅ Scalable, durable storage
- ✅ Can set retention policies
- ✅ Easy to integrate with other tools
- ✅ Works with S3-compatible storage

**Cons**:
- ❌ Requires AWS credentials
- ❌ Storage costs (minimal for reports)

---

### Solution 4: NFS/Network Share

**How it works**: Reports are written to a mounted NFS share that persists outside the container.

**Configuration**:
```yaml
persist_to_nfs: true
nfs_server: "nfs.example.com"
nfs_export_path: "/exports/reports"
nfs_mount_path: "/mnt/reports"
```

**Pros**:
- ✅ Traditional file-based storage
- ✅ Easy to access from other systems
- ✅ No cloud dependencies

**Cons**:
- ❌ Requires NFS infrastructure
- ❌ Network dependency
- ❌ Requires mount permissions in container

---

### Solution 5: Git Repository

**How it works**: Reports are committed and pushed to a Git repository.

**Configuration**:
```yaml
persist_to_git: true
git_repo_url: "https://github.com/myorg/satellite-reports.git"
git_branch: "main"
# Git credentials via environment variables or SSH keys
```

**Pros**:
- ✅ Version control for reports
- ✅ Full audit trail
- ✅ Easy to share and collaborate

**Cons**:
- ❌ Requires Git credentials
- ❌ Binary files (HTML) increase repo size
- ❌ Not ideal for frequent, large reports

---

### Solution 6: Webhook/API Endpoint

**How it works**: Reports are sent to a custom API endpoint or webhook for processing.

**Configuration**:
```yaml
send_to_webhook: true
webhook_url: "https://api.example.com/reports/satellite"
```

**Pros**:
- ✅ Flexible integration with any system
- ✅ Can trigger downstream processing
- ✅ Real-time notifications

**Cons**:
- ❌ Requires custom endpoint
- ❌ Need to handle report storage on receiving end

---

## Which Persistence Method Should You Use?

| Use Case | Recommended Method |
|----------|-------------------|
| Running in AAP/AWX | **AAP Artifacts** |
| Need stakeholder notifications | **Email** |
| Long-term archival | **S3** |
| On-premises file-based | **NFS** |
| Audit trail required | **Git** |
| Custom integration | **Webhook** |

**Best Practice**: Use multiple methods! For example:
```yaml
use_aap_artifacts: true  # For immediate access in AAP
email_report: true       # Notify stakeholders
persist_to_s3: true      # Long-term archival
```

## Role Variables

### Satellite Connection

| Variable | Default | Description |
|----------|---------|-------------|
| `satellite_server` | `satellite.example.com` | Satellite server hostname |
| `satellite_username` | `admin` | Satellite API username |
| `satellite_password` | - | Satellite API password (use Vault!) |
| `satellite_validate_certs` | `false` | Verify SSL certificates |

### Report Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `report_days_back` | `7` | How many days of history to include |
| `report_format` | `both` | Output format: `html`, `csv`, or `both` |
| `report_output_dir` | `/tmp/satellite_reports` | Where reports are initially created |
| `report_filename_prefix` | `satellite_patching_report` | Report filename prefix |

### Persistence Options

| Variable | Default | Description |
|----------|---------|-------------|
| `use_aap_artifacts` | `true` | Copy to AAP artifact directory |
| `persist_to_s3` | `false` | Upload to S3 bucket |
| `persist_to_nfs` | `false` | Copy to NFS mount |
| `persist_to_git` | `false` | Commit to Git repository |
| `email_report` | `false` | Email the reports |
| `send_to_webhook` | `false` | POST to webhook URL |

### Filter Options

| Variable | Default | Description |
|----------|---------|-------------|
| `filter_organization` | - | Limit to specific organization |
| `filter_content_view` | - | Limit to specific content view |
| `filter_lifecycle_env` | - | Limit to specific lifecycle environment |

## Example Playbooks

### Example 1: Basic Report for Last 30 Days

```yaml
- name: Generate 30-day patching report
  hosts: localhost
  roles:
    - role: satellite_patching_report
      vars:
        satellite_server: "{{ lookup('env', 'SATELLITE_SERVER') }}"
        satellite_username: "{{ lookup('env', 'SATELLITE_USERNAME') }}"
        satellite_password: "{{ lookup('env', 'SATELLITE_PASSWORD') }}"
        report_days_back: 30
        report_format: "html"
```

### Example 2: Multi-Persistence Setup

```yaml
- name: Generate report with multiple delivery methods
  hosts: localhost
  roles:
    - role: satellite_patching_report
      vars:
        satellite_server: "satellite.prod.example.com"
        satellite_username: "report_user"
        satellite_password: "{{ vault_sat_password }}"
        report_days_back: 7
        report_format: "both"
        
        # AAP artifacts for immediate access
        use_aap_artifacts: true
        
        # Email to stakeholders
        email_report: true
        email_to:
          - ops-team@example.com
          - compliance@example.com
        smtp_host: "smtp.example.com"
        
        # S3 for long-term storage
        persist_to_s3: true
        s3_bucket: "compliance-reports"
        s3_region: "us-east-1"
```

### Example 3: Scheduled Weekly Report

Create a scheduled job in AAP that runs weekly:

```yaml
- name: Weekly Satellite Patching Report
  hosts: localhost
  roles:
    - role: satellite_patching_report
      vars:
        satellite_server: "{{ satellite_url }}"
        satellite_username: "{{ satellite_user }}"
        satellite_password: "{{ satellite_pass }}"
        report_days_back: 7
        report_format: "both"
        
        use_aap_artifacts: true
        
        email_report: true
        email_to:
          - weekly-reports@example.com
        email_from: "aap-automation@example.com"
        smtp_host: "smtp.corp.example.com"
```

## Environment Variables

For better security, use environment variables:

```bash
export SATELLITE_SERVER="satellite.example.com"
export SATELLITE_USERNAME="admin"
export SATELLITE_PASSWORD="SecurePassword123"
export AWS_ACCESS_KEY_ID="your-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
```

Then in your playbook:
```yaml
satellite_server: "{{ lookup('env', 'SATELLITE_SERVER') }}"
satellite_username: "{{ lookup('env', 'SATELLITE_USERNAME') }}"
satellite_password: "{{ lookup('env', 'SATELLITE_PASSWORD') }}"
```

## Security Best Practices

1. **Never commit credentials**: Use Ansible Vault or AAP credentials
2. **Use environment variables**: For sensitive data
3. **Enable certificate validation**: Set `satellite_validate_certs: true` in production
4. **Limit API user permissions**: Create a read-only Satellite user for reporting
5. **Encrypt reports**: If emailing or storing sensitive patch information

## Ansible Vault Example

```bash
# Create encrypted variables file
ansible-vault create vars/satellite_creds.yml
```

Content:
```yaml
vault_satellite_password: "YourSecurePassword"
vault_smtp_password: "SMTPPassword"
```

Use in playbook:
```yaml
- name: Generate report with vaulted credentials
  hosts: localhost
  vars_files:
    - vars/satellite_creds.yml
  roles:
    - role: satellite_patching_report
      vars:
        satellite_password: "{{ vault_satellite_password }}"
```

## Troubleshooting

### Report shows no data

**Cause**: No patching jobs in the specified time range  
**Solution**: Increase `report_days_back` or verify jobs exist in Satellite

### Authentication failed

**Cause**: Incorrect Satellite credentials  
**Solution**: Verify username/password and user permissions

### Files not in AAP artifacts

**Cause**: `AWX_ISOLATED_DATA_DIR` not set  
**Solution**: Ensure running in AAP/AWX or set `aap_artifact_dir` manually

### S3 upload failed

**Cause**: Missing AWS credentials or permissions  
**Solution**: Verify AWS credentials and S3 bucket permissions

### Email not sending

**Cause**: SMTP configuration or network issues  
**Solution**: Test SMTP connectivity from execution environment

## Customizing Report Templates

The report templates are located in `templates/`:

- `patching_report.html.j2` - HTML report template
- `patching_report.csv.j2` - CSV report template

Modify these templates to customize:
- Report styling and branding
- Data fields included
- Table layouts
- Filtering logic

## Integration with AAP Workflows

Create a workflow in AAP:

1. **Job 1**: Run patching playbook
2. **Job 2**: Run this reporting role (always runs)
3. **Job 3**: Optional - trigger downstream actions based on report

This ensures a report is generated after every patching run.

## License

Apache 2.0

## Author

Red Hat Ansible Automation Team

## Support

For issues and contributions, please open an issue in the repository.

---

## Summary: Ephemeral Environment File Persistence

The key takeaway: **Files created in ephemeral containers are lost unless you explicitly persist them elsewhere**. This role provides six battle-tested methods to solve this problem:

1. ⭐ **AAP Artifacts** - Best for AAP/AWX users
2. 📧 **Email** - Best for notifications
3. ☁️ **S3** - Best for scalable archival
4. 📁 **NFS** - Best for on-premises file storage
5. 🔄 **Git** - Best for audit trails
6. 🔗 **Webhook** - Best for custom integrations

Choose the method (or combination of methods) that fits your infrastructure and compliance requirements!
