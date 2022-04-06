# Table Of Contents

- [Collecting Data](#Collecting-Data)
    - [Azure AD Incident Response PowerShell](#Azure-AD-Incident-Response-PowerShell)
    - [Microsoft Graph](#Microsoft-Graph)
    - [Azure AD Portal](#Azure-AD-Portal)
    - [Defender for Cloud Apps](#Defender-for-Cloud-Apps)
- [Ingesting Data](#Ingesting-Data)
- [Hunting](#Hunting)

## Collecting Data

Some of the data sources below are event driven data, and some export current configuration. It is important to note the difference. Log data such as activity logs or sign in logs will show particular events. For example, when a user signs in, or when a setting is changed. You may not retain enough data to see when particular events occured. So it is important to also include configuration data, such as a list of permissions a service principal has. That data will allow you to query on exactly what a tenant looks like at the point of export, even if you don't have the event data to show when the changes happened.

### Azure AD Incident Response PowerShell

The Azure AD Incident Response PowerShell module is available here(https://github.com/AzureAD/Azure-AD-Incident-Response-PowerShell-Module)

### Installing the module

You may need some prerequisite modules prior to installing the Azure AD Incident Response PowerShell. You can install them in order.

```powershell
Install-Module AzureAD -Force 

Install-Module MSOnline -Force 

Install-PackageProvider NuGet -Force 

Install-Module PowerShellGet -Force 

Install-Module AzureADIncidentResponse 
```

### Connecting to Azure AD Incident Response

The Azure AD Incident Response PowerShell module is a wrapper for the Microsoft Graph, so when you connect you are issued a token. That token is then re-used to retrieve the information you request. An account with either the Global Reader or Security Reader role is recommended. You can use Global Administrator too, though that is not recommended during incident response.

For many of the commands you will require the Azure AD Tenant Id of the tenant you wish to audit.

For these examples we will use the fake tenant ID of 3a37ec34-401f-49d6-b4a0-cd939838128b

```powershell
Connect-AzureADIR -tenantid 3a37ec34-401f-49d6-b4a0-cd939838128b
```

### Get Service Principal Permissions to CSV

This will extract the permissions assigned, both delegated and application, to any service principals in the Azure AD tenant.

```powershell
Get-AzureADIRPermission -tenantid 3a37ec34-401f-49d6-b4a0-cd939838128b -CsvOutput 
```

This will output the permissions as a CSV ready for ingestion to Azure Data Explorer.

You will get two CSV files, one for delegated and one for application permissions.

### Get Privileged Role Assignments to CSV

This will extract the list of all Azure AD privileged roles and any assignments.

```powershell
Get-AzureADIRPrivilegedRoleAssignment -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -CsvOutput
```

This will output the assignments as a CSV ready for ingestion to Azure Data Explorer.

### Get Azure AD PIM Assignments to CSV

This will extract the list of all Azure AD Privileged Identity Management assignments.

```powershell
Get-AzureADIRPimPrivilegedRoleAssignment -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -All -CsvOutput
```

This will output the assignments as a CSV ready for ingestion to Azure Data Explorer.

### Get Azure AD PIM Assignment Requests

This will extract the list of all Azure AD Privileged Identity Management requests

```powershell
Get-AzureADIRPimPrivilegedRoleAssignmentRequest -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -CsvOutput
```

This will output the requests as a CSV ready for ingestion to Azure Data Explorer.

### Get Azure AD Audit Activity

This will extract Azure AD Audit Activity events.

```powershell
Get-AzureADIRAuditActivity -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b
```

This cmdlet has some further inputs to filter the output. You can add which Object Id's initiated the events. If you want to check all your Global Admins for instance, or you believed a particular one was compromised. You can also filter based on the category of event, such as User Management or Application Management. You can find a list of categories [here](https://docs.microsoft.com/en-us/azure/active-directory/reports-monitoring/concept-audit-logs#filtering-audit-logs).

This cmdlet won't output the output to a CSV by default. The response by default is JSON, so you can save to JSON ready for ingestion

```powershell
Get-AzureADIRAuditActivity -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b | ConvertTo-Json | Out-file c:\ir\audit.json
```

### Get Azure AD Dismissed User Risk

This will extract the list of all Azure AD users who have had their risk state dismissed in Azure AD Identity Protection.

```powershell
Get-AzureADIRDismissedUserRisk -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -CsvOutput
```

This will output the requests as a CSV ready for ingestion to Azure Data Explorer.

### Get Azure AD Privileged User to On Premises Correlation

For this cmdlet you will need to run it from a location which can access an on premises Domain Controller and have the Windows Server Active Directory module installed.

```powershell
Get-AzureADIRPrivilegedUserOnPremCorrelation -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -OnPremDomain company.local -CsvOutput
```

### Get Azure AD Sign In Detail

This will extract sign in detail for particular users or service principals, it can also be filtered to time range if required. This should again be output as a JSON file ready for ingestion.

```powershell
 Get-AzureADIRSignInDetail -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b | ConvertTo-Json | out-file c:\ir\signins.json
```

This will export the sign in data as JSON ready for ingestion to Azure Data Explorer.

### Get Azure AD Last Sign In Detail

This will extract the last sign in detail for all users in Azure AD to CSV ready for ingestion to Azure Data Explorer.

```powershell
Get-AzureADIRUserLastSignInActivity -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -All -CsvOutput
```

### Get Azure AD Self Service Password Reset History

This will extract the details of any Self Service Password Reset activity ready for ingestion

```powershell
Get-AzureADIRSsprUsageHistory -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b-CsvOutput
```

### Get Azure AD Domain Information

This will lookup the list of public domains associated with the Azure AD tenant and use the sysinternals tool WhoIs.exe to retrieve the relevavnt information about each.

```powershell
Get-AzureADIRDomainRegistrationDetail -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -CsvOutput
```

This will output the information as a CSV ready for ingestion. You will also get a zip file with a text file of each domain with the raw output.

### Get Azure AD MFA Auth Methods Analysis

This will lookup MFA analysis for all users. If required you can scope the query to a particular user, group or location.

```powershell
Get-AzureADIRMfaAuthMethodAnalysis -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -CsvOutput
```

This will output the MFA information as a CSV ready for ingestion.

### Get Azure AD MFA Phone to Location Analysis

```powershell
Get-AzureADIRMfaPhoneToLocationCheck -TenantId 3a37ec34-401f-49d6-b4a0-cd939838128b -CsvOutput
```

This will output the MFA phone and location information as a CSV ready for ingestion.

### Get an Object Id from a friendly name

This is a cmdlet that lets you retrieve an Object Id from a friendly name, for instance if you have someones username and need their Object Id.

```powershell
Get-AzureADIRDisplayNameToObjectId -DisplayNameStartsWith "Bob" -ObjectType User
```

This will retrieve the Object Id of all user objects that start with "Bob"

### Get a friendly name from an Object Id

This cmdlet is the reverse of the previous one, it lets you retrieve the friendly name from an Object Id.

```powershell
 Get-AzureADIRObjectIdToDisplayName -ObjectIds ecebf7a6-b316-48ec-80de-69b2b0554a45
 ```

This retrieves the display name of Object Id ecebf7a6-b316-48ec-80de-69b2b0554a45 and what type of object it is.

### Get a Tenant Id from a domain name

```powershell
Get-AzureADIRTenantId -DomainName test123.com
```

This retrieves the Tenant Id for the domain test123.com

### Microsoft Graph

Although the Azure AD IR PowerShell module covers a lot of data, you may want to extract other data from Microsoft Graph. You can query the Microsoft Graph directly using PowerShell, the Graph Explorer or a tool like Postman.

There is a guide to using Postman for Microsoft Graph [here](https://docs.microsoft.com/en-us/graph/use-postman). Any responses from Microsoft Graph will be in JSON and you should be able to upload them to ADX.

Some endpoints you may be interested in.

[Azure AD Conditional Access Policies](https://docs.microsoft.com/en-us/graph/api/conditionalaccessroot-list-policies?view=graph-rest-1.0&tabs=http)

[Security Alerts](https://docs.microsoft.com/en-us/graph/api/resources/alert?view=graph-rest-1.0)

There may be some overlap with the data taken from the Azure AD IR PowerShell exports, but you may also get additional details from Microsoft Graph useful to your investigation.

### Azure AD Portal

There are a number of locations in the Azure AD Portal you can extract or download data directly to CSV or JSON. You don't always need to use PowerShell or other tooling, downloading the data directly is just as easy.

Some examples you may be able to use

[MFA Authentication Registration Details](https://portal.azure.com/#blade/Microsoft_AAD_IAM/AuthenticationMethodsMenuBlade/UserRegistrationDetails) - Available as a CSV.

[Azure AD Sign In Data](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/SignIns) - Available as CSV or JSON. I recommend using JSON as the data has nested values.

[Azure AD Audit Data](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Audit) - Available as CSV or JSON. I recommend using JSON as the data has nested values. You can filter and extract particular actions from here. For instance if you select Service: Azure MFA, Category: UserManagement, you will get all MFA events such as users registing new MFA methods.

[Azure AD Risky Sign-ins](https://portal.azure.com/#blade/Microsoft_AAD_IAM/IdentityProtectionMenuBlade/RiskySignIns) - 

Often exporting from the portal you are limited in the amount of data that can be downloaded. So you may want to filter your query prior to exporting.

There may be some overlap with the data taken from the Azure AD IR PowerShell exports, but you may also get additional details useful to your investigation.

### Defender for Cloud Apps

You can export the Activity Log directly to CSV from the Defender for Cloud Apps (previous Cloud App Security) portal into CSV ready to ingest.

There is a limit to the number of records you can download so you should filter your query first. If you are investigating particular users you could export all events for those users over the time frame you were interested in.

There is a limit of 5000 records that can be exported from the UI.

There may be some overlap with the data taken from the Azure AD IR PowerShell exports, but you may also get additional details from Defender for Cloud App useful to your investigation.

## Ingesting Data

You can create a free instance of Azure Data Explorer here(https://aka.ms/kustofree). Any Microsoft account, even a personal one, will suffice. If you already have an instance you can of course use that too.

When you first sign in you will need to create a cluster and a database. You can follow the instruction here(https://docs.microsoft.com/en-us/azure/data-explorer/start-for-free-web-ui)

You can call your cluster whatever you like. When you name your database you can also choose whatever you like, for these examples I have named my database 'AzureADIR'. If you already have a database for other incident response you can use that too of course. Especially if you want to query easily between sources.

Once your cluster and database are ready, you can ingest your data.

You can see your database name listed here, then click 'Ingest Data'. We will just use the GUI to ingest our data.

![AAD 1](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/azureadir1.png?raw=true)

Once you click Ingest Data, you need to create a Table. If you are used to Log Analytics or Microsoft Sentinel, this will be how you query your data.

For our example, we are going to ingest our CSV of our Service Principal application permissions we exported. We will send them to a table called AppPermissions.

![AAD 2](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/azureadir2.png?raw=true)

Upload your file, then when you select next you can choose what type of file it is. You will be given a preview of your data prior to ingestion. For a CSV you can ignore the first record if it has column headers already.

![AAD 3](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/azureadir3.png?raw=true)

It should then ingest for you and let you know when complete.

![AAD 4](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/azureadir4.png?raw=true)

You can then click one of the quick queries and it will send you over to the data view.

![AAD 5](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/azureadir5.png?raw=true)

If you are uploading JSON and it has nested properties you can have ADX extract those out to new columns. Depending on the source data you may want to extract some out, and leave some nested. ADX has a column limit so eventually you will reach that. For anything you don't extract at ingestion time, you can still use KQL operators such as mv-expand to access that data during queries.

![AAD 6](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/azureadir6.png?raw=true)

Once you have ingested all your data you should have a number of tables depending on what sources you are using. 

![AAD 7](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/azureadir7.png?raw=true)

## Hunting

Once your data has been loaded you can query it via KQL, the same as Log Analytics or Microsoft Sentinel. Some example queries are below. The following queries assume you have loaded the data into the following tables. Adjust them if you have named your tables differently.

Depending on your source for your data, the schema may not exactly match these examples, they are just to be used as a guide to what actions may be interesting in terms of forensics and incident response.

| Audit Source | Table Name | Source |
| --- | --- | --- |
| Azure AD Service Principal Application Permissions | AADSPApplicationPermissions | Azure AD IR |
| Azure AD Service Principal Delegated Permissions | AADSPDelegatedPermissions | Azure AD IR |
| Azure AD Audit Logs | AADAuditLogs | Azure AD IR |
| Azure AD Privileged Role Assignments | AADRoles | Azure AD IR |
| Azure AD PIM Assignments | AADPIMAssignments | Azure AD IR |
| Azure AD PIM Requests | AADPIMRequests | Azure AD IR |
| Azure AD Conditional Access | AADConditionalAccess | Azure AD IR |
| Azure AD Dismissed User Risk | AADDismissedUserRisk | Azure AD IR |
| Azure AD Privileged User On Prem Correlation | AADOnPrem | Azure AD IR |
| Azure AD Sign In Detail | AADSignInDetail | Azure AD IR |
| Azure AD Risky Sign Ins | AADRiskySignIns | Azure AD Portal
| Azure AD Last Sign In | AADLastSignIn | Azure AD IR |
| Azure AD Self Service Password Reset | AADSSPR | Azure AD IR |
| Azure AD Domain Information | AADDomainInfo | Azure AD IR |
| Azure AD MFA Analysis | AADMFA | Azure AD IR |
| Azure AD MFA Phone to Location | AADMFAPhoneLocation | Azure AD IR |
| Defender for Cloud App Logs | CloudApp | Azure AD IR |

### Hunting Queries

#### Summarize both Application and Delegated permissions by Client Id

```kql
union AADSPApplicationPermissions, AADSPDelegatedPermissions
| summarize AppPermissions=make_set_if(Permission, PermissionType == "Application"), DelegatedPermissions=make_set_if(Permission, PermissionType == "Delegated") by ClientAppId, ClientDisplayName
```

#### Find Applications with high privilege permissions

```kql
AADSPApplicationPermissions
| summarize Permissions=make_set(Permission) by ClientAppId, ClientDisplayName
| where Permissions has_any ("Directory.Read.All","Directory.ReadWrite.All","AuditLog.Read.All")
```

#### Summarize Azure AD Audit Events by activity name and user who initiated

```kql
AADAuditLogs
| summarize count()by activityDisplayName, initiatedBy_user
```

#### Find Application Consent events

```kql
AADAuditLogs
| where activityDisplayName == "Consent to application"
| parse targetResources with * '{id=' AppId ';' *
| parse targetResources with * 'displayName=' AppDisplayName ';' *
| parse initiatedBy_user with * 'userPrincipalName=' Actor ';' *
| parse initiatedBy_user with * 'ipAddress=' ActorIPAddress ';' *
| project activityDateTime, activityDisplayName, Actor, ActorIPAddress, AppDisplayName, AppId
```

These events can also be found in Defender for Cloud Apps.

```kql
CloudApp
| where Category == "Grant consent for application"
| parse Description with * 'Azure Service Principal' ServicePrincipalName
| project Category, Description, User, ['User Principle Name'], ServicePrincipalName
```

#### Find Credentials Added to Application events

```kql
AADAuditLogs
| where activityDisplayName has "Update application – Certificates and secrets management"
| parse targetResources with * '{id=' AppId ';' *
| parse targetResources with * 'displayName=' AppDisplayName ';' *
| parse initiatedBy_user with * 'userPrincipalName=' Actor ';' *
| parse initiatedBy_user with * 'ipAddress=' ActorIPAddress ';' *
| project activityDateTime, activityDisplayName, Actor, ActorIPAddress, AppDisplayName, AppId
```

These events can also be found in Defender for Cloud Apps.

```kql
CloudApp
| where Category == "Add service principal credentials"
| parse Description with * 'Azure Service Principal' ServicePrincipalName
| project Category, Description, User, ['User Principle Name'], ServicePrincipalName
```

#### For any Application Consent events find sign in data from the same IP address

```kql
let ipaddresses=
AADAuditLogs
| where activityDisplayName == "Consent to application"
| parse targetResources with * '{id=' AppId ';' *
| parse targetResources with * 'displayName=' AppDisplayName ';' *
| parse initiatedBy_user with * 'userPrincipalName=' Actor ';' *
| parse initiatedBy_user with * 'ipAddress=' ActorIPAddress ';' *
| distinct ActorIPAddress;
AADSignInDetail
| where ipAddress in (ipaddresses)
| project createdDateTime, userPrincipalName, userAgent, isInteractive,  status_errorCode, status_failureReason, status_additionalDetails
```

#### Detect new audit events not seen before (last 5 days vs prior 30 days)

```kql
let knownactivities=
AADAuditLogs
| where activityDateTime> ago(30d) and activityDateTime < ago(5d)
| distinct activityDisplayName;
AADAuditLogs
| where activityDateTime> ago(5d)
| where activityDisplayName !in (knownactivities)
| distinct activityDisplayName, category
```

#### Detect first time legacy authentication attempt

```kql
let existinglegacyusers=
AADSignInDetail
| where createdDateTime > ago(30d) and createdDateTime < ago(1d)
| where status_errorCode == 0
| where clientAppUsed !in ("Browser","Mobile Apps and Desktop clients")
| distinct userPrincipalName;
AADSignInDetail
| where createdDateTime > ago(1d)
| where userPrincipalName !in (existinglegacyusers)
| where clientAppUsed !in ("Browser","Mobile Apps and Desktop clients")
```

#### Summarize all privileged roles held by users, groups or service prinicipals

```kql
AADRoles
| summarize UserRoles=make_set_if(RoleMemberName, RoleMemberObjectType == "User"), GroupRoles=make_set_if(RoleMemberName, RoleMemberObjectType == "Group"),SPRoles=make_set_if(RoleMemberName, RoleMemberObjectType == "ServicePrincipal") by DirectoryRole
```

#### Find users being added to privileged roles events

```kql
AADAuditLogs
| where activityDisplayName == "Add member to role"
//Exclude activations by MS PIM if you are looking for manual role additions
| where initiatedBy_app_displayName != "MS-PIM"
```

#### Summarize domain names assigned to Azure AD Tenant and whether verified or not

```kql
AADDomainInfo
| distinct DomainName, AzureADVerified
```

#### Find conditional access audit events

```kql
AADAuditLogs
| where activityDisplayName in ("Add conditional access policy","Delete conditional access policy","Update conditional access policy")
| parse targetResources with * '{id=' PolicyId ';' *
| parse targetResources with * 'displayName=' PolicyName ';' *
| parse initiatedBy_user with * 'userPrincipalName=' Actor ';' *
| parse initiatedBy_user with * 'ipAddress=' ActorIPAddress ';' *
| project activityDateTime, activityDisplayName, Actor, ActorIPAddress, PolicyName, PolicyId
```

#### Find users with more than one MFA method registered, but defaulting to using SMS

```kql
AADMFA
| where MfaAuthMethodCount > 1 and DefaultMethod == "OneWaySMS"
```

#### Find users with a usage location different to their MFA phone location

```kql
AADMFAPhoneLocation
| where UserUsageCountry != MfaPhoneNumberCountry
```

#### Correlate users with a risky sign in with MFA registration events

```kql
AADRiskySignIns
| where createdDateTime > ago (30d)
| project RiskEventTime=createdDateTime, userPrincipalName
| join kind=inner (
    AADAuditLogs
    | where activityDateTime > ago (30d)
    | where activityDisplayName  in ("User registered security info", "User deleted security info","User registered all required security info")
    | project MFATime=activityDateTime, initiatedBy_user_userPrincipalName
) on $left.userPrincipalName==$right.initiatedBy_user_userPrincipalName
| extend ['Time Between Events']=datetime_diff("minute",MFATime, RiskEventTime)
```

#### Find updates to Azure AD Authentication Method Policies

```kql
AADAuditLogs
| where activityDisplayName == "Authentication Methods Policy Update"
```

#### Find PIM activations on weekends

```kql
let Saturday = time(6.00:00:00);
let Sunday = time(0.00:00:00);
AADAuditLogs
| where activityDisplayName == "Add member to role completed (PIM activation)"
| where dayofweek(activityDateTime) in (Saturday, Sunday)
```
