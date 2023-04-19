---
title: Known issues in Azure Stack HCI 2303 Supplemental Package (preview)
description: Read about the known issues in Azure Stack HCI 2303 Supplemental Package (preview).
author: alkohli
ms.topic: conceptual
ms.date: 04/19/2023
ms.author: alkohli
ms.reviewer: alkohli
ms.subservice: azure-stack-hci
---

# View known issues in Azure Stack HCI, 2303 Supplemental Package release (preview)

[!INCLUDE [applies-to](../includes/hci-applies-to-supplemental-package.md)]

This article identifies the critical known issues and their workarounds in Azure Stack HCI.

The release notes are continuously updated, and as critical issues requiring a workaround are discovered, they're added. Before you deploy your Azure Stack HCI, carefully review the information contained in the release notes.

This article applies to Azure Stack HCI, Supplemental Package, for 2303 release. This release maps to software version number **10.2303.0.36**. This release supports only brand new software installations using a deployment tool.

For more information, see [What's new in 22H2](whats-new.md#azure-stack-hci-supplemental-package-preview) and [What's in preview](./manage/whats-new-2301-preview.md#azure-stack-hci-2301-supplemental-package-preview).

[!INCLUDE [important](../includes/hci-preview.md)]

## Known issues in this release

Here are the known issues in the current Azure Stack HCI supplemental package release:

|#|Feature|Issue|Workaround|
|-|------|------|----------|
|1|Azure Arc |Azure Arc enabled on Azure Stack HCI cluster may fail. |For Arc enablement failures, manually repair the registration on your cluster: <br> Run this command: `Register-AzStackHCI  -SubscriptionId "<subscription_ID>" -ComputerName Server1 -RepairRegistration` <br> For more information, see [Troubleshoot Azure Stack HCI registration issues and errors](./deploy/troubleshoot-hci-registration.md).|
|2|Host networking |Updating jumbo packet configuration on existing network intents has no effect. |Make sure to configure overrides to default values during the network intent creation. After your cluster is deployed, you can create a network intent that uses a customized jumbo packet. <br><br> If the cluster is in Windows Defender Application Control (WDAC) enforcement mode, switch the node from where you'll set the override in `Audit` mode. To switch the local node to audit, run the following command: <br> `Enable-ASLocalWDACPolicy -Mode Audit` <br>For more information, see [Enable WDAC policy modes](./concepts/security-windows-defender-application-control.md#enable-wdac-policy-modes). <br><br>To create a new compute intent `NewComputeIntent` on adapters `NIC1` and `NIC2` that use `JumboPacket` as **9014**, run the following commands:<br> 1. `$adapterOverrides = New-NetIntentAdapterPropertyOverrides`<br/><br>2. `$adapterOverrides.JumboPacket = 9014`<br/><br/>3.`Add-NetIntent -Name NewComputeIntent -Compute -AdapterName @("NIC1", "NIC2") -AdapterPropertyOverrides $adapterOverrides`|
|3|Host networking |After the Azure Stack HCI cluster deployment, if you try to create a new network ATC compute intent with overrides, the operation fails with the following error: *Cannot set property. Property setting is supported only on core types in this language mode*.|Property setting is not allowed in Constrained Language mode. |
|4|Deployment|Steps 0.1.13.1 fail with type **ValidateWindowsFeatureInstallation** of role **BareMetal**.|During the operating system (OS) installation phase, on the first OS boot, run the following commands on each of the host machines. These commands ensure that Windows updates won't be downloaded and installed during the deployment.<br />Run the following three commands right after the OS deployment:<br /><br />1. `reg add HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU /v NoAutoUpdate /t REG_DWORD /d 1 /f`<br />2. `reg add HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU /v AUOptions /t REG_DWORD /d 3 /f`<br />3. `Set-Service "WUAUSERV" -StartupType Disabled`|

## Known issues from previous releases

Here are the known issues that have carried over from the previous releases in Azure Stack HCI supplemental package:

|#|Feature|Issue|Workaround|
|-|------|------|----------|
|1|Deployment |Failure in ECE – *Set-RoleDefinition: Can't find the element 'NodeDefinition' for the role NC*|Make sure that a DVD isn't inserted in the physical machine or mounted via the Baseboard Management Controller (BMC).|
|2|Deployment |During deployment, an error is seen in Windows Admin Center: Remote Exception "GetCredential" with "1".|Reboot the staging server and run the bootstrap script again. Make sure that the Azure credentials for the subscription haven't expired and are correct.|
|3|Deployment |Renaming the network adapter in the deployment tool and using the **Back** and **Next** buttons causes it to hang.|There's no workaround for this is in the preview release.|
|4|Arc VM and AKS Hybrid workload deployment |In this release, Windows Defender App Control (WDAC) is enabled by default on Azure Stack HCI servers. If you're deploying Arc VM or AKS Hybrid workloads, you would see an error while importing the Arc Resource Bridge PowerShell module. |The workaround is to switch WDAC policy mode to `Audit` instead of `Enforced`. For more information, see [Enable WDAC policy modes](./concepts/security-windows-defender-application-control.md#enable-wdac-policy-modes). <br> After the workload deployment is complete, you can run the cmdlet to switch WDAC policy mode back to the default enforced mode.|
|5|Deployment |If you have an Azure policy assigned to the Azure subscription that enforces tags, the deployment fails.||
|6|OS update|After deployment, scanning for OS updates using *SConfig* or Cluster-Aware Updating may fail to scan Windows Update.|Manually enable and start the *wuauserv* service. Run the following PowerShell commands: <br>`Set-Service -Name WUAUServ -StartupType Auto -Verbose`<br/>`Start-Service -Name WUAUServ`|
|7|Deployment|Cluster creation fails with the error "Microsoft-Windows-Kerberos-Key-Distribution-Center Event ID 14 error event" in the System section of Event Log on your domain controller.|Search for and install the following cumulative updates:<br>Windows Server 2022: [KB5021656](https://support.microsoft.com/help/5021656)<br>Windows Server 2019: [KB5021655](https://support.microsoft.com/help/5021655)<br>Windows Server 2016: [KB5021654](https://support.microsoft.com/help/5021654)<br>To get the standalone package for these out-of-band updates, search for the KB number in the Microsoft Update Catalog. You can manually import these updates into Windows Server Update Services (WSUS) and Microsoft Endpoint Configuration Manager. For WSUS instructions, see [WSUS and the Catalog Site](/windows-server/administration/windows-server-update-services/manage/wsus-and-the-catalog-site#the-microsoft-update-catalog-site). For Configuration Manger instructions, see [Import updates from the Microsoft Update Catalog](/mem/configmgr/sum/get-started/synchronize-software-updates#import-updates-from-the-microsoft-update-catalog). |
|8|Environment Checker| If SSL inspection is turned on in your Azure Stack HCI system, the connectivity validator fails with the certificate validation error message.  | For information about the error and how to troubleshoot it, see [Potential failure scenario for connectivity validator](./manage/use-environment-checker.md#potential-failure-scenario-for-connectivity-validator).|
|9|Diagnostics | Deployment was configured with **Diagnostic data** set to ON in the deployment tool. However during the deployment and after the deployment is complete, the diagnostic data isn't collected.|You can run the `Send-DiagnosticsData`command on Azure Stack HCI cluster node to collect diagnostic logs. |



## Next steps

- Read the [Deployment overview](./deploy/deployment-tool-introduction.md)