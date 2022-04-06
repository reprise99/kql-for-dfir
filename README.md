# kql-for-dfir

The concept behind using KQL for DFIR is simple. We want to leverage the hunting capabilities of KQL to find evidence in our audit and forensic data.

To do that, we collect forensic data using a number of different tools. That forensic data could include sign in data, event logs, current configurations and many other items. It could also come in various forms, such as JSON, CSV or just text files.

We then ingest that data into Azure Data Explorer, and we are then able to query it using KQL.

You can sign up to a free instance of Azure Data Explorer here(https://aka.ms/kustofree)

This guide will step you through using KQL for DFIR in three steps.

For each technology or device we want to investigate we will collect data, ingest that data, then hunt on it.

For some investigations you may want the data from many sources. For an Active Directory incident you likely want Active Directory specific data as well as information about the Windows host itself. For an Office 365 breach, you may want both Azure Active Directory and Active Directory logs depending on your identity configuration.

You may also have a SIEM such as Microsoft Sentinel. This is not designed to replace that, but to be used as another data source during incident response. Your SIEM may not have complete coverage of all data required for your investigation. We will also be taking forensic information directly from devices, which won't be in your SIEM.

The following KQL for DFIR guides have been written so far.

## 1. [Azure Active Directory](https://github.com/reprise99/kql-for-dfir/tree/main/Azure%20Active%20Directory)

Use the Azure AD IR PowerShell module and other log sources to investigate Azure AD.

## 2. [Windows](https://github.com/reprise99/kql-for-dfir/tree/main/Windows)

Use Defender for Endpoint, Windows event logs and foresntic tools to investigate a particular device.

## 3. [Active Directory](https://github.com/reprise99/kql-for-dfir/tree/main/Active%20Directory)

Use Event Logs, DNS logs and Defender for Identity to investigate Active Directory.

## 4. [Office 365](https://github.com/reprise99/kql-for-dfir/tree/main/Office%20365)

Use data from Exchange Online, Security and Compliance centre and Azure Active Directory.

# Coming Soon

## 5. Active Directory Certificate Services

## 6. Windows Network Policy Server