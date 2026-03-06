# Detection Rules for Azure-Splunk Integration

This folder contains 13 SPL detection rules for monitoring Azure Entra ID sign-in activity.

## Rules Overview

| File | Detection | MITRE ATT&CK |
|------|-----------|--------------|
| 01-failed-logins.spl | Failed Logins | T1110 |
| 02-brute-force.spl | Brute Force | T1110 |
| 03-brute-force-success.spl | Brute Force → Success | T1110 |
| 04-login-summary.spl | Login Summary by User | T1078 |
| 05-logins-by-location.spl | Logins by Location | T1078 |
| 06-mfa-blocked.spl | MFA Blocked | T1111 |
| 07-conditional-access-failures.spl | CA Failures | T1078 |
| 08-impossible-travel.spl | Impossible Travel | T1078 |
| 09-risky-signins.spl | Risky Sign-ins | T1078 |
| 10-multiple-ips.spl | Multiple IPs per User | T1078 |
| 11-client-app-analysis.spl | Client App Analysis | T1078 |
| 12-logins-by-app.spl | Logins by Application | T1078 |
| 13-failed-logins-location.spl | Failed Logins by Location | T1110 |

## Usage

1. Copy the SPL query from any `.spl` file
2. Paste into Splunk Search
3. Adjust time range as needed
4. Save as Report or Alert for continuous monitoring
