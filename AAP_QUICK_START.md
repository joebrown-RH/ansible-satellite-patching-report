# AAP Quick Start

## Using with AAP's Built-In Red Hat Satellite Credential

The playbooks are configured to work with AAP's built-in **Red Hat Satellite** credential type.

### Setup in AAP:

1. **Create a Satellite Credential**:
   - Go to: **Resources → Credentials → Add**
   - **Name**: `My Satellite Server`
   - **Credential Type**: `Red Hat Satellite 6`
   - Fill in:
     - **Host**: Your Satellite server (e.g., `satellite.example.com`)
     - **Username**: Your Satellite username
     - **Password**: Your Satellite password
   - Click **Save**

2. **Create Project**:
   - Go to: **Resources → Projects → Add**
   - **Name**: `Satellite Patching Reports`
   - **SCM Type**: `Git`
   - **SCM URL**: `https://github.com/joebrown-RH/ansible-satellite-patching-report.git`
   - **SCM Branch**: `main`
   - Click **Save** and sync

3. **Create Job Template**:
   - Go to: **Resources → Templates → Add → Job Template**
   - **Name**: `Satellite Patching Report`
   - **Inventory**: Select any inventory
   - **Project**: `Satellite Patching Reports`
   - **Playbook**: `satellite_patching_report_test.yml`
   - **Credentials**: Add your `My Satellite Server` credential
   - Click **Save**

4. **Launch and Get Reports**:
   - Click **Launch** on your job template
   - After completion, view the job
   - Go to **Output → Artifacts** tab
   - Download your HTML and CSV reports

## Customizing the Report

Add these to **Extra Variables** in your Job Template:

```yaml
# Look back 30 days instead of 7
report_days_back: 30

# Generate only HTML (or 'csv' or 'both')
report_format: html

# Filter by organization
filter_organization: "MyOrg"
```

## Troubleshooting

**If you get "undefined variable" errors**, the playbooks will try these variable names in order:
1. `host`, `username`, `password` (AAP built-in credential)
2. `satellite_url`, `satellite_username`, `satellite_password` (custom credentials)
3. Environment variables `SATELLITE_SERVER`, `SATELLITE_USERNAME`, `SATELLITE_PASSWORD`

If none work, add them manually to **Extra Variables**:
```yaml
host: "satellite.example.com"
username: "admin"
password: "changeme"
```
