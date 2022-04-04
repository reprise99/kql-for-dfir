# kql-for-dfir

The concept behind using KQL for DFIR is simple. We want to leverage the hunting capabilities of KQL to find evidence in our audit and forensic data.

To do that, we collect forensic data using a number of different tools. That forensic data could include sign in data, event logs, current configurations and many other items. It could also come in various forms, such as JSON, CSV or just text files.

We then ingest that data into Azure Data Explorer, and we are then able to query it using KQL.

You can sign up to a free instance of Azure Data Explorer here(https://aka.ms/kustofree)

This guide will step you through using KQL for DFIR in three steps.

For each technology or device we want to investigate we will collect data, ingest that data, then hunt on it.

The following KQL for DFIR guides have been written so far.

## 1. [Azure Active Directory](https://github.com/reprise99/kql-for-dfir/tree/main/Azure%20Active%20Directory)

Use the Azure AD IR PowerShell module and other log sources to investigate Azure AD.

## 2. [Windows](https://github.com/reprise99/kql-for-dfir/tree/main/Windows)

Use Defender for Endpoint, Windows event logs and foresntic tools to investigate a particular device.

## 3. [Active Directory](https://github.com/reprise99/kql-for-dfir/tree/main/Active%20Directory)

Use Event Logs, DNS logs and Defender for Identity to investigate Active Directory.
