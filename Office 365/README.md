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

### Office 365 Unified Audit Log

### Defender for Cloud Apps

You can export the Activity Log directly to CSV from the Defender for Cloud Apps (previous Cloud App Security) portal into CSV ready to ingest.

There is a limit to the number of records you can download so you should filter your query first. If you are investigating particular users you could export all events for those users over the time frame you were interested in.

Office 365 is available as a preset filter if you want to select that.

![O365 1](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/o365ir1.png?raw=true)

There is a limit of 5000 records that can be exported from the UI.

There will be  overlap with the data taken from the Office 365 Unified Audit Logs exports, but you may also get additional details from Defender for Cloud App useful to your investigation.

### Security and Compliance Centre
