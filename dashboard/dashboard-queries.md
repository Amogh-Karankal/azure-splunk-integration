# Dashboard Panel Queries

## Azure Entra ID Security Dashboard

---

### Panel 1: Total Sign-in Events (Single Value)

```spl
index=azure_logs
| stats count as "Total Sign-ins"
```

**Visualization:** Single Value

---

### Panel 2: Failed vs Successful Logins (Pie Chart)

```spl
index=azure_logs
| spath status.errorCode
| eval Result=if('status.errorCode'==0, "Success", "Failed")
| stats count by Result
```

**Visualization:** Pie Chart

---

### Panel 3: Failed Logins Over Time (Line Chart)

```spl
index=azure_logs
| spath status.errorCode
| where 'status.errorCode' != 0
| timechart count as "Failed Logins"
```

**Visualization:** Line Chart

---

### Panel 4: Top Targeted Users (Bar Chart)

```spl
index=azure_logs
| spath status.errorCode
| where 'status.errorCode' != 0
| stats count as Failures by userDisplayName
| sort - Failures
| head 10
```

**Visualization:** Bar Chart (Horizontal)

---

### Panel 5: Sign-ins by Country (Pie Chart)

```spl
index=azure_logs
| spath location.countryOrRegion
| stats count by location.countryOrRegion
```

**Visualization:** Pie Chart

---

### Panel 6: Sign-ins by City (Bar Chart)

```spl
index=azure_logs
| spath location.city
| stats count by location.city
| sort - count
| head 10
```

**Visualization:** Bar Chart

---

### Panel 7: Brute Force Attempts (Table)

```spl
index=azure_logs
| spath status.errorCode
| where 'status.errorCode' != 0
| stats count as FailedAttempts by userPrincipalName, ipAddress
| where FailedAttempts > 2
| sort - FailedAttempts
```

**Visualization:** Table

---

### Panel 8: MFA Blocks (Table)

```spl
index=azure_logs
| spath status.failureReason
| where 'status.failureReason'="Strong Authentication is required."
| table _time, userDisplayName, ipAddress, appDisplayName
```

**Visualization:** Table

---

### Panel 9: Impossible Travel Detected (Table)

```spl
index=azure_logs
| spath status.errorCode
| spath location.countryOrRegion
| where 'status.errorCode'=0
| stats dc(location.countryOrRegion) as CountryCount, values(location.countryOrRegion) as Countries by userPrincipalName
| where CountryCount > 1
```

**Visualization:** Table

---

### Panel 10: Conditional Access Failures (Table)

```spl
index=azure_logs
| spath conditionalAccessStatus
| where conditionalAccessStatus="failure"
| table _time, userDisplayName, ipAddress, appDisplayName
```

**Visualization:** Table

---

## Dashboard Layout

```
┌────────────────┬────────────────┬────────────────┐
│ Total Sign-ins │ Failed vs Pass │  By Country    │
│ (Single Value) │  (Pie Chart)   │  (Pie Chart)   │
├────────────────┴────────────────┴────────────────┤
│         Failed Logins Over Time (Line Chart)     │
├────────────────┬─────────────────────────────────┤
│ Top Targeted   │     Sign-ins by City            │
│ Users (Bar)    │     (Bar Chart)                 │
├────────────────┴─────────────────────────────────┤
│         Brute Force Attempts (Table)             │
├──────────────────────────────────────────────────┤
│  MFA Blocks  │ Impossible Travel │  CA Failures  │
└──────────────────────────────────────────────────┘
```
