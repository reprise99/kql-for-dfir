# Table Of Contents

- [Collecting Data](#Collecting-Data)
    - [Office 365 Unified Audit Log](#Azure-AD-Incident-Response-PowerShell)
    - [Defender for Cloud Apps](#Defender-for-Cloud-Apps)
    - [Security and Compliance Centre](#Security-and-Compliance-Centre)
- [Ingesting Data](#Ingesting-Data)
- [Hunting](#Hunting)

## Collecting Data

Some of the data sources below are event driven data, and some export current configuration. It is important to note the difference. Log data such as activity logs or sign in logs will show particular events. For example, when a user signs in, or when a setting is changed. You may not retain enough data to see when particular events occured. So it is important to also include configuration data, such as a list of permissions a service principal has. That data will allow you to query on exactly what a tenant looks like at the point of export, even if you don't have the event data to show when the changes happened.

With Office 365 incidents, it is likely you will also want the logs and forensic data from [Azure Active Directory](https://github.com/reprise99/kql-for-dfir/tree/main/Azure%20Active%20Directory). Azure Active Directory is the authentication and authorization platform for Office 365 so the the data from both is important during incident response.

### Offfice 365 Unified Audit Log

### Defender for Cloud Apps

### Security and Compliance Centre
