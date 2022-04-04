# Table Of Contents

- [Collecting Data](#Collecting-Data)
    - [Defender for Identity](#Defender-for-Identity)
    - [Windows Event Logs](#Windows-Event-Logs)
    - [Forensic Tools](#Forensic-Tools)
- [Ingesting Data](#Ingesting-Data)
- [Hunting](#Hunting)

## Collecting Data

Some of the data sources below are event driven data, and some export current configuration. It is important to note the difference. Data such as event viewer logs will show particular events. For example, when a user logs onto a device, or when a setting on a device is changed. You may not retain enough data to see when particular events occured. So it is important to also include configuration data, such as a list of current registry keys or scheduled tasks. That data will allow you to query on exactly what a device looks like at the point of export, even if you don't have the event data to show when the event happened.

### Defender for Identity

If you use Defender for Identity you can export the activies from Domain Controllers and then save as a CSV and ingest them to Azure Data Explorer.

In the Defender for Identity [portal](https://portal.atp.azure.com), you can browse to a Domain Controller. From that screen you can then download activites for a selected period.

![AD 1](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/adir1.png?raw=true)

![AD 2](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/adir2.png?raw=true)

Once downloaded you will notice that it is downloaded in as an Excel spreadsheet, and not CSV. You will just to remove the logo and graphics from the top of the spreadsheet and save as CSV. Each file has 3 tabs. One for activities, one for directory service events and one for alerts. If you want to ingest those into different tables, just save them as three separate CSV's.

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

You can call your cluster whatever you like. When you name your database you can also choose whatever you like, for these examples I have named my database 'ActiveDirectoryIR'. If you already have a database for other incident response you can use that too of course. Especially if you want to query easily between sources.

Once your cluster and database are ready, you can ingest your data.

You can see your database name listed here, then click 'Ingest Data'. We will just use the GUI to ingest our data.

![AD 3](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/adir3.png?raw=true)

Once you click Ingest Data, you need to create a Table. If you are used to Log Analytics or Microsoft Sentinel, this will be how you query your data.

For our example, we are going to ingest our CSV showing Domain Controller activities taken from Defender for Identity. We will send them to a table called DFIActivities.

![AD 4](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/adir4.png?raw=true)

Upload your file, then when you select next you can choose what type of file it is. You will be given a preview of your data prior to ingestion. For a CSV you can ignore the first record if it has column headers already.

Once you have ingested all your data you should have a number of tables depending on what sources you are using.

![AD 5](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/adir5.png?raw=true)

You can then query your data.

![AD 6](https://github.com/reprise99/kql-for-dfir/blob/main/.Images/windowsir6.png?raw=true)

## Hunting