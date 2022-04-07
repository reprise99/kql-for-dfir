# Combined Queries

Within Azure Data Explorer, you can query across databases, much like a regular Log Analytics workspace. If you have ingested a few data sets into your ADX instance and want to query across them, you can.

Some of examples of these follow. They are based on the following data structure, yours may vary so just use this as a guide.

When using a join or union, you just need to specify the databases you are combining.

```kql
union database('ActiveDirectoryIR'), database('WindowsIR')
```

Or on a join include the database and table name.

```kql
AADRoles
| join kind=inner database('Office365IR').O365UAP on
```

| Database | Table Name | Log Source |
| --- | --- | --- |
| ActiveDirectoryIR | DFIActivities | Defender for Identity Portal
|  | DFIDirectoryEvents | Defender for Identity Portal
|  | DFIAlerts | Defender for Identity Portal
| | SecurityEvent | Security Events
| | CyberTriage | Forensic Tooling
| Office365IR | O365UAP | Office 365 Unified Audit
|  | O365MessageTrace | Exchange Online Message Trace
|  | CloudApp | Defender for Cloud App Events
| AzureADIR | AADSPApplicationPermissions | Azure AD Service Principal Application Permissions |
|  | AADSPDelegatedPermissions | Azure AD Service Principal Delegated Permissions |
|  | AADAuditLogs | Azure AD Audit Logs  |
|  | AADRoles | Azure AD Privileged Role Assignments |
|  | AADPIMAssignments | Azure AD PIM Assignments |
|  | AADPIMRequests | Azure AD PIM Requests |
|  | AADConditionalAccess | Azure AD Conditional Access |
|  | AADDismissedUserRisk | Azure AD Dismissed User Risk |
|  | AADOnPrem | Azure AD Privileged User On Prem Correlation |
|  | AADSignInDetail | Azure AD Sign In Detail |
|  | AADRiskySignIns | Azure AD Risky Sign Ins
|  | AADLastSignIn | Azure AD Last Sign In |
|  | AADSSPR | Azure AD Self Service Password Reset |
|  | AADDomainInfo | Azure AD Domain Information |
|  | AADMFA | Azure AD MFA Analysis |
|  | AADMFAPhoneLocation | Azure AD MFA Phone to Location |
|  | CloudApp | Defender for Cloud App Logs |
| WindowsIR | InstalledPrograms | DfE Investigation Package
|  | Processes | Processes
|  | ScheduledTasks | Scheduled Tasks
| | SecurityEvent |  Windows Security Event Log 
|  | Services | Services
|  | CyberTriage | Forensic Tooling

## Hunting

#### Retrieve all O365 audit actitivites from privileged Azure AD Users

```kql
AADRoles
| where DirectoryRole in ("Global Administrator","Application Administrator","Exchange Administrator","Teams Administrator")
| distinct RoleMemberUPN, DirectoryRole
| join kind=inner database('Office365IR').O365UAP on $left.RoleMemberUPN == $right.UserIds
| project CreationDate, UserIds, DirectoryRole, Operations
```

#### Malicious email received by a user with a privileged AAD role

```kql
AADRoles
| where DirectoryRole in ("Global Administrator","Application Administrator","Exchange Administrator","Teams Administrator")
| distinct RoleMemberUPN, DirectoryRole
| join kind=inner 
(
database('Office365IR').O365MessageTrace
| where Subject == "Malicious Subject"
)
 on $left.RoleMemberUPN == $right.RecipientAddress
 ```

 #### Application consent or service principal addition by a user who received a malicious email

```kql
let attackedusers=
O365MessageTrace
| where Subject == "Malicious Subject"
| distinct RecipientAddress;
database("AzureADIR").AADAuditLogs
| where activityDisplayName == "Consent to application"
| parse targetResources with * '{id=' AppId ';' *
| parse targetResources with * 'displayName=' AppDisplayName ';' *
| where initiatedBy_user_userPrincipalName in (attackedusers)
| project activityDateTime, activityDisplayName, initiatedBy_user_userPrincipalName, initiatedBy_user_ipAddress, AppDisplayName, AppId
```
