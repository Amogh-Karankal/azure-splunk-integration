# Setup Guide: Azure to Splunk SIEM Integration

Complete step-by-step guide to connect Microsoft Azure Entra ID to Splunk Enterprise.

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| Splunk Enterprise | Free license (500MB/day) or higher |
| Azure Subscription | With Entra ID (Azure AD) |
| Azure Role | Global Administrator or Security Reader |
| Network | Outbound HTTPS to Microsoft APIs |

---

## Phase 1: Install Splunk Enterprise

### Option A: Windows/Mac/Linux Direct Install

1. Download from: https://www.splunk.com/en_us/download.html
2. Run installer
3. Set admin password
4. Access: `http://localhost:8000`

### Option B: Docker

```bash
docker run -d --name splunk \
  -p 8000:8000 \
  -e SPLUNK_START_ARGS='--accept-license' \
  -e SPLUNK_PASSWORD='YourPassword123!' \
  splunk/splunk:latest
```

Access: `http://localhost:8000`  
Login: `admin` / `YourPassword123!`

---

## Phase 2: Create Azure App Registration

### Step 2.1: Register Application

1. Go to **Azure Portal** → **Microsoft Entra ID**
2. Click **App registrations** → **New registration**
3. Configure:
   - Name: `Splunk-Log-Collector`
   - Supported account types: **Single tenant**
   - Redirect URI: Leave blank
4. Click **Register**

### Step 2.2: Note Application IDs

From the Overview page, copy:
- **Application (client) ID**: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`
- **Directory (tenant) ID**: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

### Step 2.3: Create Client Secret

1. Go to **Certificates & secrets**
2. Click **New client secret**
3. Configure:
   - Description: `Splunk-Secret`
   - Expires: 24 months
4. Click **Add**
5. **IMMEDIATELY copy the Value** (shown only once!)

### Step 2.4: Grant API Permissions

1. Go to **API permissions**
2. Click **Add a permission**

**Add Microsoft Graph permissions:**
- Select **Microsoft Graph** → **Application permissions**
- Add: `AuditLog.Read.All`
- Add: `Directory.Read.All`

**Add Office 365 Management APIs permissions:**
- Click **Add a permission**
- Select **APIs my organization uses**
- Search: `Office 365 Management APIs`
- Select **Application permissions**
- Add: `ActivityFeed.Read`

3. Click **Grant admin consent** → **Yes**
4. Verify all permissions show green checkmarks ✅

---

## Phase 3: Install Splunk Add-on

### Step 3.1: Install from Splunkbase

1. In Splunk Web, click **Apps** → **Find More Apps**
2. Search: `Splunk Add-on for Microsoft Office 365`
3. Click **Install**
4. Enter Splunk.com credentials (not local Splunk login)
5. Restart Splunk when prompted

### Step 3.2: Manual Install (Alternative)

If app store login fails:

1. Go to: https://splunkbase.splunk.com
2. Search: `Splunk Add-on for Microsoft Office 365`
3. Download the `.tgz` file
4. In Splunk: **Apps** → **Install app from file**
5. Upload the `.tgz` file
6. Restart Splunk

---

## Phase 4: Configure Add-on

### Step 4.1: Add Tenant

1. Go to **Splunk Add-on for Microsoft Office 365**
2. Click **Configuration** tab
3. Click **Add** under Tenant

4. Configure:
   | Field | Value |
   |-------|-------|
   | Name | `azure-entra-logs` |
   | Endpoint | `Worldwide` |
   | Tenant ID | (from Azure) |
   | Client ID | (from Azure) |
   | Authentication Type | Client Secret Based |
   | Client Secret | (from Azure) |

5. Click **Add**

### Step 4.2: Create Index

1. Go to **Settings** → **Indexes**
2. Click **New Index**
3. Index Name: `azure_logs`
4. Click **Save**

### Step 4.3: Create Input

1. Go to Add-on → **Inputs** tab
2. Click **Create New Input** → **Audit Logs**
3. Configure:
   | Field | Value |
   |-------|-------|
   | Name | `azure_audit_logs` |
   | Tenant | `azure-entra-logs` |
   | Content Type | `Audit Logs.Sign Ins` |
   | Index | `azure_logs` |
   | Interval | 300 |

4. Click **Save**

---

## Phase 5: Verify Data Flow

### Wait Time

- First data: 15-30 minutes
- Full data flow: Up to 24 hours

### Verify Data

```spl
index=azure_logs
| stats count by sourcetype
```

Expected result: `o365:graph:api` with count > 0

### Check for Errors

```spl
index=_internal sourcetype=splunkd *o365* error
| table _time, message
| head 20
```

---

## Troubleshooting

### No Data Appearing

| Check | Solution |
|-------|----------|
| Input enabled? | Inputs tab → verify Active |
| Index exists? | Settings → Indexes → verify `azure_logs` |
| Permissions granted? | Azure → API permissions → green checkmarks |
| Secret expired? | Azure → create new secret |
| Correct tenant? | Verify Tenant ID matches |

### Authentication Errors

```spl
index=_internal "unauthorized" OR "401" OR "403" o365
| table _time, message
```

- Regenerate client secret
- Re-grant admin consent
- Verify Client ID matches

### Index Errors

If you see "index does not exist":
1. Settings → Indexes → New Index
2. Match exact name from Input configuration
3. Case sensitive! `azure_logs` ≠ `Azure_logs`

---

## Key Field Names

| Field | Description |
|-------|-------------|
| `userPrincipalName` | User email/UPN |
| `userDisplayName` | User friendly name |
| `ipAddress` | Source IP |
| `status.errorCode` | 0 = success, other = failure |
| `status.failureReason` | Why login failed |
| `location.city` | Login city |
| `location.countryOrRegion` | Login country |
| `appDisplayName` | Application used |
| `conditionalAccessStatus` | CA policy result |
| `riskState` | Risk level |

---

## Next Steps

1. Create detection rules (see `/detection-rules/`)
2. Build dashboard (see `/dashboard/`)
3. Set up alerts for critical detections
