# Satellite Patching Report Types

This role now supports three different report types, each designed for specific use cases in your patching workflow.

## Report Types Overview

### 1. Enhanced Report (Default)
**Purpose:** General-purpose job-centric patching report  
**Use Case:** Day-to-day patching operations and troubleshooting  
**Variable:** `report_type: enhanced`

**What it shows:**
- Patching jobs from the last N days
- Package updates per host within each job
- Job success/failure status
- Errata associated with each host
- Searchable and filterable interface

**Best for:**
- IT operations teams tracking patching job status
- Troubleshooting failed patching jobs
- Reviewing recent patching activity

---

### 2. Pre-Patching Report (NEW)
**Purpose:** Shows AVAILABLE errata before patching  
**Use Case:** Security team review before monthly/weekly patching window  
**Variable:** `report_type: pre-patch`

**What it shows:**
- All available (not yet applied) errata per host
- Organized by severity (Critical, Important, Moderate, Low)
- CVE details for each erratum
- Hosts grouped by environment
- Complete list of affected packages

**Best for:**
- **Wednesday 9:30am CST** - Pre-patching security review
- Identifying which hosts have critical vulnerabilities
- Planning patching priorities
- Creating exemption lists

**Workflow:**
```yaml
# Run before your patching window
- name: Generate pre-patch report for security review
  hosts: localhost
  vars:
    satellite_server: satellite.example.com
    satellite_username: admin
    satellite_password: !vault |...
    report_type: pre-patch
    report_format: both
    email_report: true
    email_to:
      - security-team@example.com
  roles:
    - satellite_patching_report
```

**Output Files:**
- `satellite_patching_report_pre_patch_TIMESTAMP.html` - Interactive HTML report
- `satellite_patching_report_pre_patch_TIMESTAMP.csv` - Flat file for Excel/BI tools

**CSV Columns:**
```
Hostname, Environment, ContentView, ContentViewVersion, OperatingSystem,
ErrataID, CVE, Type, Severity, Title, IssuedDate, PackagesAffected,
PackageCount, Status, LastCheckin
```

---

### 3. Post-Patching Report (NEW)
**Purpose:** Shows APPLIED errata after patching  
**Use Case:** Compliance verification after patching window  
**Variable:** `report_type: post-patch`

**What it shows:**
- What was actually applied during patching jobs
- Success/failure status per host
- Package version changes (old → new)
- Errata that were applied
- Error messages for failed hosts
- Discrepancies (hosts that were skipped)

**Best for:**
- Post-patching compliance verification
- Proving to security team that patches were applied
- Identifying failed hosts requiring manual intervention
- Finding hosts that were skipped without exemption

**Workflow:**
```yaml
# Run after your patching window
- name: Generate post-patch compliance report
  hosts: localhost
  vars:
    satellite_server: satellite.example.com
    satellite_username: admin
    satellite_password: !vault |...
    report_type: post-patch
    report_days_back: 7  # Look back at last week's patching
    report_format: both
    email_report: true
    email_to:
      - security-team@example.com
      - compliance-team@example.com
  roles:
    - satellite_patching_report
```

**Output Files:**
- `satellite_patching_report_post_patch_TIMESTAMP.html` - Interactive HTML report
- `satellite_patching_report_post_patch_TIMESTAMP.csv` - Flat file for Excel/BI tools

**CSV Columns:**
```
Hostname, Environment, ContentView, OperatingSystem, JobID, JobDescription,
JobStartTime, JobStatus, PackageName, OldVersion, NewVersion, Arch,
ErrataID, CVE, Severity, Type, ErrataTitle, AppliedDate, RebootRequired,
ErrorMessage
```

---

## Comparison: Pre-Patch vs Post-Patch

| Feature | Pre-Patch Report | Post-Patch Report |
|---------|-----------------|-------------------|
| **Shows** | Available errata | Applied errata |
| **Status** | What WILL be patched | What WAS patched |
| **When to Run** | Before patching window | After patching window |
| **Primary Audience** | Security team | Security & Compliance teams |
| **Use Case** | Planning & approval | Verification & proof |
| **Package Info** | List of affected packages | Version changes (old→new) |
| **Error Details** | N/A | Shows why hosts failed |
| **Success Metrics** | Errata count by severity | Success rate, packages updated |

---

## Scheduling Both Reports

For a complete security workflow, schedule both reports:

### AAP Job Template: Pre-Patch Report
**Schedule:** (before weekly patching)

```yaml
---
- name: Weekly Pre-Patch Security Review
  hosts: localhost
  vars:
    report_type: pre-patch
    report_format: both
    email_report: true
    email_to:
      - security@example.com
    email_subject: "Pre-Patch Report - Available Errata for {{ ansible_date_time.date }}"
  roles:
    - satellite_patching_report
```

### AAP Job Template: Post-Patch Report
**Schedule:** (after weekend patching)

```yaml
---
- name: Weekly Post-Patch Compliance Report
  hosts: localhost
  vars:
    report_type: post-patch
    report_days_back: 7  # Capture last week's patching
    report_format: both
    email_report: true
    email_to:
      - security@example.com
      - compliance@example.com
    email_subject: "Post-Patch Report - Compliance Verification for {{ ansible_date_time.date }}"
  roles:
    - satellite_patching_report
```

---

## Scalability Features (All Report Types)

All new report types are designed to handle **hundreds of hosts** efficiently:

✅ **Collapsed by Default** - Only summary cards visible initially  
✅ **Progressive Disclosure** - Click to expand details  
✅ **Instant Search** - Client-side JavaScript filtering  
✅ **Smart Grouping** - By environment, severity, or job  
✅ **Table View Toggle** - Switch between cards and compact table  
✅ **CSV Export** - Complete flat file for Excel analysis  
✅ **Sticky Header** - Summary stats always visible while scrolling  

### Search Examples:
- `CVE-2024-12345` - Find all hosts affected by a specific CVE
- `kernel` - Find all hosts with kernel updates
- `critical` - Show only hosts with critical errata
- `server01` - Jump to a specific host

---

## Configuration Variables

```yaml
# Choose your report type
report_type: enhanced      # Options: enhanced, pre-patch, post-patch

# How far back to look for jobs
report_days_back: 7        # Default: 7 days

# Output format
report_format: both        # Options: html, csv, both

# Output location
report_output_dir: /tmp/satellite_reports

# Email delivery (optional)
email_report: true
email_to:
  - security@example.com
email_from: satellite-reports@example.com
email_subject: "{{ 'Pre-Patch' if report_type == 'pre-patch' else 'Post-Patch' }} Report"

# AAP artifact directory (recommended for AAP environments)
use_aap_artifacts: true
aap_artifact_dir: /runner/artifacts
```

---

## Migration from Existing Reports

The original `patching_report.html.j2` and `patching_report_enhanced.html.j2` templates are **unchanged**.

To use the new report types:
1. Set `report_type: pre-patch` or `report_type: post-patch`
2. No other changes required

To keep using existing reports:
- Leave `report_type: enhanced` (or omit it)
- Reports will work exactly as before

---

## Troubleshooting

### "No errata available for this host" (Pre-Patch Report)
- This is normal if the host is fully patched
- Check that the content view has been updated recently
- Verify the host is checking in to Satellite

### "No packages updated" (Post-Patch Report)
- Job may have run but found nothing to update
- Check that errata were available before the job ran
- Review job logs for dependency conflicts

### Report is slow with 500+ hosts
- Use CSV export for large datasets
- Enable `report_format: csv` only
- Use filtering in AAP inventory to limit hosts

### Email not sending
- Verify SMTP settings in defaults/main.yml
- Check that `email_report: true` is set
- Ensure email_to list is populated

---

## Support

For issues or questions about these report types:
1. Check the task logs in AAP
2. Review the generated CSV for data accuracy
3. Verify Satellite API permissions for the service account

## Examples

See `examples/` directory for:
- AAP job template YAML
- Sample cron schedules
- Integration with ServiceNow/ITSM tools
