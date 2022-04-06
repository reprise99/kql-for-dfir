# Table Of Contents

- [Collecting Data](#Collecting-Data)
    - [Defender for Endpoint Investigation Package](#Defender-for-Endpoint-Investigation-Package)
    - [Windows Event Logs](#Windows-Event-Logs)
    - [Forensic Tools](#Forensic-Tools)
- [Ingesting Data](#Ingesting-Data)
- [Hunting](#Hunting)

## Collecting Data

Some of the data sources below are event driven data, and some export current configuration. It is important to note the difference. Data such as event viewer logs will show particular events. For example, when a user logs onto a device, or when a setting on a device is changed. You may not retain enough data to see when particular events occured. So it is important to also include configuration data, such as a list of current registry keys or scheduled tasks. That data will allow you to query on exactly what a device looks like at the point of export, even if you don't have the event data to show when the event happened.

### Defender for Endpoint Investigation Package

You can collect an investigation package from any device registered in Defender for Endpoint from the security portal [here](https://security.microsoft.com).

Simply search for the device you are interested in.

Then select collect investigation package.

![Windows 1](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir1.png?raw=true)

After some time the package will be available to download.

Within the ZIP file you will a list of folders and a collection summary.

![Windows 2](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir2.png?raw=true)

In each folder there are different types of files, such as text files, csv files and event viewer exports.

### Windows Event Logs

There are multiple ways to get event logs from a Windows machine so this post won't cover them all, you may already have a tool that can collect them from a remote machine for you. You can also use [PowerShell](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-eventlog?view=powershell-5.1). For ease of intergration with Azure Data Explorer I would recommend exporting them directly to CSV, and then uploading them.

### Forensic Tools

There are a number of forensic tools available in the market, some open source and free, some paid and some a mixture of both. In general they will connect remotely to the suspicious machine and collect various types of forensic data for you. That could include event logs, memory dumps, configuration items and many more.

These tools will often also overlay some kind of analysis onto that data and can point you in the direction of things that may be of interest.

For this example, we will use an output from [Cyber Triage](https://www.cybertriage.com/). Which outputs its findings into JSON format, which we then ingest.

These tools may have a bit of overlap with other data sources, but that's fine. More data is always beneficial.

Each tool is obviously going to have its own unique data structure, so the queries here may not line up with what you find using your tooling. They are just a guide to direct you to things to search for.

## Ingesting Data

You can create a free instance of Azure Data Explorer [here](https://aka.ms/kustofree). Any Microsoft account, even a personal one, will suffice. If you already have an instance you can of course use that too.

When you first sign in you will need to create a cluster and a database. You can follow the instruction [here](https://docs.microsoft.com/en-us/azure/data-explorer/start-for-free-web-ui)

You can call your cluster whatever you like. When you name your database you can also choose whatever you like, for these examples I have named my database 'WindowsIR'. If you already have a database for other incident response you can use that too of course. Especially if you want to query easily between sources.

Once your cluster and database are ready, you can ingest your data.

You can see your database name listed here, then click 'Ingest Data'. We will just use the GUI to ingest our data.

![Windows 3](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir3.png?raw=true)

Once you click Ingest Data, you need to create a Table. If you are used to Log Analytics or Microsoft Sentinel, this will be how you query your data.

For our example, we are going to ingest our CSV of the information about the services running on this device. We will send them to a table called Services.

![Windows 4](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir4.png?raw=true)

Upload your file, then when you select next you can choose what type of file it is. You will be given a preview of your data prior to ingestion. For a CSV you can ignore the first record if it has column headers already. Occasionally the CSV files taken from Defender will cause an error on ingestion. They are saved as UTF-8 format by default, if you save as a standard CSV and then import they should work fine.

![Windows 5](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir5.png?raw=true)

It should then ingest for you and let you know when complete.

One note about the Security Event Log, it will export from Defender as a *.evtx file which Azure Data Explorer cannot parse. You can convert to CSV using Log Parser, available [here](https://www.microsoft.com/en-au/download/details.aspx?id=24659). As example command to convert an evtx file to csv is.

```cmd
logparser "Select * into C:\temp\SecEvents.csv from C:\temp\Security.evtx" -i:evt -o:csv
```

Once you have ingested all your data you should have a number of tables depending on what sources you are using.

![Windows 7](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir7.png?raw=true)

You can then query your data.

![Windows 6](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir6.png?raw=true)

## Hunting

Once your data has been loaded you can query it via KQL, the same as Log Analytics or Microsoft Sentinel. Some example queries are below. The following queries assume you have loaded the data into the following tables. Adjust them if you have named your tables differently.

Depending on your source for your data, the schema may not exactly match these examples, they are just to be used as a guide to what actions may be interesting in terms of forensics and incident response.

| Data| Table Name | Log Source |
| --- | --- | --- |
| Installed Programs | InstalledPrograms | DfE Investigation Package
| Processes | Processes | DfE Investigation Package
| Scheduled Tasks | ScheduledTasks | DfE Investigation Package
| Windows Security Event Log | SecurityEvent | DfE Investigation Package
| Services | Services | DfE Investigation Package
| Cyber Triage Output | CyberTriage | Forensic Tooling

#### Summarize scheduled tasks by run as account

```kql
ScheduledTasks
| summarize count()by ['Run As User']
```

#### Find local user creation events

```kql
SecurityEvent
| where EventID == "4720"
```

#### Find scheduled task creation events

```kql
SecurityEvent
| where EventID == "4698"
```

#### Find services running as non system accounts

```kql
Services
| where StartName !in ("NT AUTHORITY\\LocalService","LocalSystem","NT Authority\\LocalService", "NT AUTHORITY\\NetworkService","localSystem","NT Authority\\NetworkService")
```

#### Find all logon events and parse the user, domain and logon type

```kql
SecurityEvent
| where EventID == "4624"
| extend LogonDetails = split(Strings,"|")
| extend UserName = LogonDetails.[5]
| extend Domain = LogonDetails.[6]
| extend LogonType = LogonDetails.[8]
| project TimeGenerated, UserName, Domain, LogonType
```

#### Find logon events from users in the last day not previously seen in the prior 30 days

```kql
let knownusers=
SecurityEvent
| where TimeGenerated > ago(30d) and TimeGenerated < ago(1d)
| where EventID == "4624"
| extend LogonDetails = split(Strings,"|")
| extend UserName = LogonDetails.[5]
| extend Domain = LogonDetails.[6]
| extend LogonType = LogonDetails.[8]
| distinct UserName;
SecurityEvent
| where TimeGenerated > ago(1d)
| where EventID == "4624"
| extend LogonDetails = split(Strings,"|")
| extend UserName = LogonDetails.[5]
| extend Domain = LogonDetails.[6]
| extend LogonType = LogonDetails.[8]
| where UserName !in (knownusers)
| project TimeGenerated, UserName, Domain, LogonType
```

#### Find RDP logon events

```kql
SecurityEvent
| where EventID == "4624"
| extend LogonDetails = split(Strings,"|")
| extend UserName = LogonDetails.[5]
| extend Domain = LogonDetails.[6]
| extend LogonType = LogonDetails.[8]
| where LogonDetails == 10
| project TimeGenerated, UserName, Domain, LogonType
```

#### Find attempts to install services on the device and parse the name, path and user context

```kql
SecurityEvent
| where EventID == "4697"
| extend Details = split(Strings, '|')
| extend ServiceName = Details.[4]
| extend ServicePath = Details.[5]
| extend UserContext = Details.[8]
| project TimeGenerated, ServiceName, ServicePath, UserContext
```

#### Cyber Triage notable or likely notable events

```kql
CyberTriage
| mv-expand analysisResults
| extend EventSignificance = tostring(analysisResults.significance)
| summarize count()by EventSignificance, type
| where EventSignificance in ("LikelyNotable","Notable")
```

#### Cyber Triage notable active network events

```kql
CyberTriage
| mv-expand analysisResults
| extend EventSignificance = tostring(analysisResults.significance)
| where EventSignificance in ("LikelyNotable","Notable")
| where type in ("activeNetworkConnection","listeningPort")
```

#### Cyber Triage distinct list of domains files have been downloaded from

```kql
CyberTriage
| where type == "DOWNLOAD"
| distinct remoteHostName
```