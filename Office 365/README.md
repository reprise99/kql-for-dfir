# Table Of Contents

- [Collecting Data](#Collecting-Data)
    - [Office 365 Unified Audit Log](#Azure-AD-Incident-Response-PowerShell)
    - [Defender for Cloud Apps](#Defender-for-Cloud-Apps)
    - [Message Tracking Log](#Security-and-Compliance-Centre)
- [Ingesting Data](#Ingesting-Data)
- [Hunting](#Hunting)

## Collecting Data

Some of the data sources below are event driven data, and some export current configuration. It is important to note the difference. Log data such as activity logs or sign in logs will show particular events. For example, when a user signs in, or when a setting is changed. You may not retain enough data to see when particular events occured. So it is important to also include configuration data, such as a list of permissions a service principal has. That data will allow you to query on exactly what a tenant looks like at the point of export, even if you don't have the event data to show when the changes happened.

With Office 365 incidents, it is likely you will also want the logs and forensic data from [Azure Active Directory](https://github.com/reprise99/kql-for-dfir/tree/main/Azure%20Active%20Directory). Azure Active Directory is the authentication and authorization platform for Office 365 so the the data from both is important during incident response.

### Office 365 Unified Audit Log

You can export events from the Unified Audit Log found [here](https://security.microsoft.com/auditlogsearch).

There is a limit to how many items you can export, so you can filter on times, activities, users or particular sites or files.

![O365 1](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/o365ir1.png?raw=true)

Once you have searched you can export to CSV ready to ingest.

### Defender for Cloud Apps

You can export the Activity Log directly to CSV from the Defender for Cloud Apps (previous Cloud App Security) portal into CSV ready to ingest.

There is a limit to the number of records you can download so you should filter your query first. If you are investigating particular users you could export all events for those users over the time frame you were interested in.

Office 365 is available as a preset filter if you want to select that.

![O365 2](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/o365ir2.png?raw=true)

There is a limit of 5000 records that can be exported from the UI.

There will be overlap with the data taken from the Office 365 Unified Audit Logs exports, but you may also get additional details from Defender for Cloud App useful to your investigation.

### Message Tracking Log

You can export the message tracking log from Exchange Online from the Exchange Admin Center here

![O365 3](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/o365ir3.png?raw=true)

This will give you an export of all email traffic over the period you select.

There will be overlap with the data taken from the other sources, but more data for investigations is always useful.

## Ingesting Data

You can create a free instance of Azure Data Explorer here(https://aka.ms/kustofree). Any Microsoft account, even a personal one, will suffice. If you already have an instance you can of course use that too.

When you first sign in you will need to create a cluster and a database. You can follow the instruction here(https://docs.microsoft.com/en-us/azure/data-explorer/start-for-free-web-ui)

You can call your cluster whatever you like. When you name your database you can also choose whatever you like, for these examples I have named my database 'Office365IR'. If you already have a database for other incident response you can use that too of course. Especially if you want to query easily between sources.

Once your cluster and database are ready, you can ingest your data.

You can see your database name listed here, then click 'Ingest Data'. We will just use the GUI to ingest our data.

![O365 4](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/o365ir4.png?raw=true)

Once you click Ingest Data, you need to create a Table. If you are used to Log Analytics or Microsoft Sentinel, this will be how you query your data.

For our example, we are going to ingest our Unified Audit Log output. Occasionally the CSV files exported will cause an error on ingestion. They are saved as UTF-8 format by default, if you save as a standard CSV and then import they should work fine.

![O365 5](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/o365ir5.png?raw=true)

Upload your file, then when you select next you can choose what type of file it is. You will be given a preview of your data prior to ingestion. For a CSV you can ignore the first record if it has column headers already.
