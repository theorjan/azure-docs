---
title: Troubleshoot assessments in Azure Migrate
description: Get help with assessment in Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: troubleshooting
ms.date: 01/02/2020
---

# Troubleshoot assessment

This article helps you troubleshoot issues with assessment and dependency visualization with [Azure Migrate: Discovery and assessment](migrate-services-overview.md#azure-migrate-discovery-and-assessment-tool).


## Assessment readiness issues

Fix assessment readiness issues as follows:

**Issue** | **Fix**
--- | ---
Unsupported boot type | Azure doesn't support VMs with an EFI boot type. We recommend that you convert the boot type to BIOS before you run a migration. <br/><br/>You can use Azure Migrate Server Migration to handle the migration of such VMs. It will convert the boot type of the VM to BIOS during the migration.
Conditionally supported Windows operating system | The operating system has passed its end-of-support date, and needs a Custom Support Agreement (CSA) for [support in Azure](/troubleshoot/azure/virtual-machines/server-software-support). Consider upgrading before you migrate to Azure. Review information about [preparing servers running Windows Server 2003](prepare-windows-server-2003-migration.md) for migration to Azure.
Unsupported Windows operating system | Azure supports only [selected Windows OS versions](/troubleshoot/azure/virtual-machines/server-software-support). Consider upgrading the server before you migrate to Azure.
Conditionally endorsed Linux OS | Azure endorses only [selected Linux OS versions](../virtual-machines/linux/endorsed-distros.md). Consider upgrading the server before you migrate to Azure. Also refer [here](#linux-vms-are-conditionally-ready-in-an-azure-vm-assessment) for more details.
Unendorsed Linux OS | The server might start in Azure, but Azure provides no operating system support. Consider upgrading to an [endorsed Linux version](../virtual-machines/linux/endorsed-distros.md) before you migrate to Azure.
Unknown operating system | The operating system of the VM was specified as "Other" in vCenter Server. This behavior blocks Azure Migrate from verifying the Azure readiness of the VM. Make sure that the operating system is [supported](./migrate-support-matrix-vmware-migration.md#azure-vm-requirements) by Azure before you migrate the server.
Unsupported bit version | VMs with a 32-bit operating systems might boot in Azure, but we recommended that you upgrade to 64-bit before you migrate to Azure.
Requires a Microsoft Visual Studio subscription | The server is running a Windows client operating system, which is supported only through a Visual Studio subscription.
VM not found for the required storage performance | The storage performance (input/output operations per second [IOPS] and throughput) required for the server exceeds Azure VM support. Reduce storage requirements for the server before migration.
VM not found for the required network performance | The network performance (in/out) required for the server exceeds Azure VM support. Reduce the networking requirements for the server.
VM not found in the specified location | Use a different target location before migration.
One or more unsuitable disks | One or more disks attached to the VM don't meet Azure requirements.A<br/><br/> Azure Migrate: Discovery and assessment currently doesn't support Ultra SSD disks, and assesses the disks based on the disk limits for premium managed disks (32 TB).<br/><br/> For each disk attached to the VM, make sure that the size of the disk is < 64 TB (supported by Ultra SSD disks).<br/><br/> If it isn't, reduce the disk size before you migrate to Azure, or use multiple disks in Azure and [stripe them together](../virtual-machines/premium-storage-performance.md#disk-striping) to get higher storage limits. Make sure that the performance (IOPS and throughput) needed by each disk is supported by Azure [managed virtual machine disks](../azure-resource-manager/management/azure-subscription-service-limits.md#storage-limits).
One or more unsuitable network adapters. | Remove unused network adapters from the server before migration.
Disk count exceeds limit | Remove unused disks from the server before migration.
Disk size exceeds limit | Azure Migrate: Discovery and assessment currently doesn't support Ultra SSD disks, and assesses the disks based on premium disk limits (32 TB).<br/><br/> However, Azure supports disks with up to 64-TB size (supported by Ultra SSD disks). Shrink disks to less than 64 TB before migration, or use multiple disks in Azure and [stripe them together](../virtual-machines/premium-storage-performance.md#disk-striping) to get higher storage limits.
Disk unavailable in the specified location | Make sure the disk is in your target location before you migrate.
Disk unavailable for the specified redundancy | The disk should use the redundancy storage type defined in the assessment settings (LRS by default).
Could not determine disk suitability because of an internal error | Try creating a new assessment for the group.
VM with required cores and memory not found | Azure couldn't find a suitable VM type. Reduce the memory and number of cores of the on-premises server before you migrate.
Could not determine VM suitability because of an internal error | Try creating a new assessment for the group.
Could not determine suitability for one or more disks because of an internal error | Try creating a new assessment for the group.
Could not determine suitability for one or more network adapters because of an internal error | Try creating a new assessment for the group.
No VM size found for offer currency Reserved Instance | Server marked Not suitable because the VM size was not found for the selected combination of RI, offer and currency. Edit the assessment properties to choose the valid combinations and recalculate the assessment. 
Conditionally ready Internet Protocol | Only applicable to Azure VMware Solution (AVS) assessments. AVS does not support IPv6 internet addresses factor. Contact the AVS team for remediation guidance if your server is detected with IPv6.

## Suggested migration tool in import-based AVS assessment marked as unknown

For servers imported via a CSV file, the default migration tool in and AVS assessment is unknown. Though, for servers in VMware environment, its is recommended to use the VMware Hybrid Cloud Extension (HCX) solution. [Learn More](../azure-vmware/configure-vmware-hcx.md).

## Linux VMs are "conditionally ready" in an Azure VM assessment

In the case of VMware and Hyper-V VMs, Azure VM assessment marks Linux VMs as "Conditionally ready" due to a known gap. 

- The gap prevents it from detecting the minor version of the Linux OS installed on the on-premises VMs.
- For example, for RHEL 6.10, currently Azure VM assessment detects only RHEL 6 as the OS version. This is because the vCenter Server ar the Hyper-V host do not provide the kernel version for Linux VM operating systems.
-  Because Azure endorses only specific versions of Linux, the Linux VMs are currently marked as conditionally ready in Azure VM assessment.
- You can determine whether the Linux OS running on the on-premises VM is endorsed in Azure by reviewing [Azure Linux support](../virtual-machines/linux/endorsed-distros.md).
-  After you've verified the endorsed distribution, you can ignore this warning.

This gap can be addressed by enabling [application discovery](./how-to-discover-applications.md) on the VMware VMs. Azure VM assessment uses the operating system detected from the VM using the guest credentials provided. This operating system data identifies the right OS information in the case of both Windows and Linux VMs.

## Operating system version not available

For physical servers, the operating system minor version information should be available. If not available, contact Microsoft Support. For servers in VMware environment, Azure Migrate uses the operating system information specified for the VM in vCenter Server. However, vCenter Server doesn't provide the minor version for operating systems. To discover the minor version, you need to set up [application discovery](./how-to-discover-applications.md). For Hyper-V VMs, operating system minor version discovery is not supported. 

## Azure SKUs bigger than on-premises in an Azure VM assessment

Azure VM assessment might recommend Azure VM SKUs with more cores and memory than current on-premises allocation based on the type of assessment:

- The VM SKU recommendation depends on the assessment properties.
- This is affected by the type of assessment you perform in Azure VM assessment: *Performance-based*, or *As on-premises*.
- For performance-based assessments, Azure VM assessment considers the utilization data of the on-premises VMs (CPU, memory, disk, and network utilization) to determine the right target VM SKU for your on-premises VMs. It also adds a comfort factor when determining effective utilization.
- For on-premises sizing, performance data is not considered, and the target SKU is recommended based on-premises allocation.

To show how this can affect recommendations, let's take an example:

We have an on-premises VM with four cores and eight GB of memory, with 50% CPU utilization and 50% memory utilization, and a specified comfort factor of 1.3.

-  If the assessment is **As on-premises**, an Azure VM SKU with four cores and 8 GB of memory is recommended.
- If the assessment is performance-based, based on effective CPU and memory utilization (50% of 4 cores * 1.3 = 2.6 cores and 50% of 8-GB memory * 1.3 = 5.3-GB memory), the cheapest VM SKU of four cores (nearest supported core count) and eight GB of memory (nearest supported memory size) is recommended.
- [Learn more](concepts-assessment-calculation.md#types-of-assessments) about assessment sizing.

## Why is the recommended Azure disk SKUs bigger than on-premises in an Azure VM assessment?

Azure VM assessment might recommend a bigger disk based on the type of assessment.
- Disk sizing depends on two assessment properties: sizing criteria and storage type.
- If the sizing criteria is **Performance-based**, and the storage type is set to **Automatic**, the IOPS, and throughput values of the disk are considered when identifying the target disk type (Standard HDD, Standard SSD, or Premium). A disk SKU from the disk type is then recommended, and the recommendation considers the size requirements of the on-premises disk.
- If the sizing criteria is **Performance-based**, and the storage type is **Premium**, a premium disk SKU in Azure is recommended based on the IOPS, throughput, and size requirements of the on-premises disk. The same logic is used to perform disk sizing when the sizing criteria is **As on-premises** and the storage type is **Standard HDD**, **Standard SSD**, or **Premium**.

As an example, if you have an on-premises disk with 32 GB of memory, but the aggregated read and write IOPS for the disk is 800 IOPS, Azure VM assessment recommends a premium disk (because of the higher IOPS requirements), and then recommends a disk SKU that can support the required IOPS and size. The nearest match in this example would be P15 (256 GB, 1100 IOPS). Even though the size required by the on-premises disk was 32 GB, Azure VM assessment recommends a larger disk because of the high IOPS requirement of the on-premises disk.

## Why is performance data missing for some/all VMs in my assessment report?

For "Performance-based" assessment, the assessment report export says 'PercentageOfCoresUtilizedMissing' or 'PercentageOfMemoryUtilizedMissing' when the Azure Migrate appliance cannot collect performance data for the on-premises VMs. Please check:

- If the VMs are powered on for the duration for which you are creating the assessment
- If only memory counters are missing and you are trying to assess Hyper-V VMs, check if you have dynamic memory enabled on these VMs. There is a known issue currently due to which Azure Migrate appliance cannot collect memory utilization for such VMs.
- If all of the performance counters are missing, ensure the port access requirements for assessment are met. Learn more about the port access requirements for [VMware](./migrate-support-matrix-vmware.md#port-access-requirements), [Hyper-V](./migrate-support-matrix-hyper-v.md#port-access) and [physical](./migrate-support-matrix-physical.md#port-access) assessment.
Note- If any of the performance counters are missing, Azure Migrate: Discovery and assessment falls back to the allocated cores/memory on-premises and recommends a VM size accordingly.

## Why is performance data missing for some/all servers in my Azure VM and/or AVS assessment report?

For "Performance-based" assessment, the assessment report export says 'PercentageOfCoresUtilizedMissing' or 'PercentageOfMemoryUtilizedMissing' when the Azure Migrate appliance cannot collect performance data for the on-premises servers. Please check:

- If the servers are powered on for the duration for which you are creating the assessment
- If only memory counters are missing and you are trying to assess servers in Hyper-V environment. In this scenario, please enable dynamic memory on the servers and 'Recalculate' the assessment to reflect the latest changes. The appliance can collect memory utilization values for severs in Hyper-V environment only when the server has dynamic memory enabled.

- If all of the performance counters are missing, ensure that outbound connections on ports 443 (HTTPS) are allowed.

    > [!Note]
    > If any of the performance counters are missing, Azure Migrate: Discovery and assessment falls back to the allocated cores/memory on-premises and recommends a VM size accordingly.


## Why is performance data missing for some/all SQL instances/databases in my Azure SQL assessment?

To ensure performance data is collected, please check:

- If the SQL Servers are powered on for the duration for which you are creating the assessment
- If the connection status of the SQL agent in Azure Migrate is 'Connected' and check the last heartbeat 
- If Azure Migrate connection status for all SQL instances is 'Connected' in the discovered SQL instance blade
- If all of the performance counters are missing, ensure that outbound connections on ports 443 (HTTPS) are allowed

If any of the performance counters are missing, Azure SQL assessment recommends the smallest Azure SQL configuration for that instance/database.

## Why is the confidence rating of my assessment low?

The confidence rating is calculated for "Performance-based" assessments based on the percentage of [available data points](./concepts-assessment-calculation.md#ratings) needed to compute the assessment. Below are the reasons why an assessment could get a low confidence rating:

- You did not profile your environment for the duration for which you are creating the assessment. For example, if you are creating an assessment with performance duration set to one week, you need to wait for at least a week after you start the discovery for all the data points to get collected. If you cannot wait for the duration, please change the performance duration to a smaller period and **Recalculate** the assessment.
 
- Assessment is not able to collect the performance data for some or all the servers in the assessment period. For a high confidence rating, please ensure that: 
    - Servers are powered on for the duration of the assessment
    - Outbound connections on ports 443 are allowed
    - For Hyper-V Servers dynamic memory is enabled 
    - The connection status of agents in Azure Migrate are 'Connected' and check the last heartbeat
    - For For Azure SQL assessments, Azure Migrate connection status for all SQL instances is "Connected" in the discovered SQL instance blade

    Please **Recalculate** the assessment to reflect the latest changes in confidence rating.

- For Azure VM and AVS assessments, few servers were created after discovery had started. For example, if you are creating an assessment for the performance history of last one month, but few servers were created in the environment only a week ago. In this case, the performance data for the new servers will not be available for the entire duration and the confidence rating would be low. [Learn more](./concepts-assessment-calculation.md#confidence-ratings-performance-based)

- For Azure SQL assessments, few SQL instances or databases were created after discovery had started. For example, if you are creating an assessment for the performance history of last one month, but few SQL instances or databases were created in the environment only a week ago. In this case, the performance data for the new servers will not be available for the entire duration and the confidence rating would be low. [Learn more](./concepts-azure-sql-assessment-calculation.md#confidence-ratings)

## Is the operating system license included in an Azure VM assessment?

Azure VM assessment currently considers the operating system license cost only for Windows servers. License costs for Linux servers aren't currently considered.

## How does performance-based sizing work in an Azure VM assessment?

Azure VM assessment continuously collects performance data of on-premises servers and uses it to recommend the VM SKU and disk SKU in Azure. [Learn how](concepts-assessment-calculation.md#calculate-sizing-performance-based) performance-based data is collected.

## Why is my assessment showing a warning that it was created with an invalid combination of Reserved Instances, VM uptime and Discount (%)?
When you select 'Reserved instances', the 'Discount (%)' and 'VM uptime' properties are not applicable. As your assessment was created with an invalid combination of these properties, the edit and recalculate buttons are disabled. Please create a new assessment. [Learn more](./concepts-assessment-calculation.md#whats-an-assessment).

## I do not see performance data for some network adapters on my physical servers

This can happen if the physical server has Hyper-V virtualization enabled. On these servers, due to a product gap, Azure Migrate currently discovers both the physical and virtual network adapters. The network throughput is captured only on the virtual network adapters discovered.

## Recommended Azure VM SKU for my physical server is oversized

This can happen if the physical server has Hyper-V virtualization enabled. On these servers, Azure Migrate currently discovers both the physical and virtual network adapters. Hence, the no. of network adapters discovered is higher than actual. As Azure VM assessment picks an Azure VM that can support the required number of network adapters, this can potentially result in an oversized VM. [Learn more](./concepts-assessment-calculation.md#calculating-sizing) about the impact of no. of network adapters on sizing. This is a product gap that will be addressed going forward.

## Readiness category "Not ready" for my physical server

Readiness category may be incorrectly marked as "Not Ready" in the case of a physical server that has Hyper-V virtualization enabled. On these servers, due to a product gap, Azure Migrate currently discovers both the physical and virtual adapters. Hence, the no. of network adapters discovered is higher than actual. In both as-on-premises and performance-based assessments, Azure VM assessment picks an Azure VM that can support the required number of network adapters. If the number of network adapters is discovered to be being higher than 32, the maximum no. of NICs supported on Azure VMs, the server will be marked “Not ready”.  [Learn more](./concepts-assessment-calculation.md#calculating-sizing) about the impact of no. of NICs on sizing.


## Number of discovered NICs higher than actual for physical servers

This can happen if the physical server has Hyper-V virtualization enabled. On these servers, Azure Migrate currently discovers both the physical and virtual adapters. Hence, the no. of NICs discovered is higher than actual.


## Capture network traffic

Collect network traffic logs as follows:

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Press F12 to start Developer Tools. If needed, clear the  **Clear entries on navigation** setting.
3. Select the **Network** tab, and start capturing network traffic:
   - In Chrome, select **Preserve log**. The recording should start automatically. A red circle indicates that traffic is being captured. If the red circle doesn't appear, select the black circle to start.
   - In Microsoft Edge and Internet Explorer, recording should start automatically. If it doesn't, select the green play button.
4. Try to reproduce the error.
5. After you've encountered the error while recording, stop recording, and save a copy of the recorded activity:
   - In Chrome, right-click and select **Save as HAR with content**. This action compresses and exports the logs as a .har file.
   - In Microsoft Edge or Internet Explorer, select the **Export captured traffic** option. This action compresses and exports the log.
6. Select the **Console** tab to check for any warnings or errors. To save the console log:
   - In Chrome, right-click anywhere in the console log. Select **Save as**, to export, and zip the log.
   - In Microsoft Edge or Internet Explorer, right-click the errors and select **Copy all**.
7. Close Developer Tools.


## Where is the operating system data in my assessment discovered from?

- For VMware VMs, by default, it is the operating system data provided by the vCenter. 
   - For VMware linux VMs, if application discovery is enabled, the OS details are fetched from the guest VM. To check which OS details in the assessment, go to the Discovered servers view, and mouse over the value in the "Operating system" column. In the text that pops up, you would be able to see whether the OS data you see is gathered from vCenter server or from the guest VM using the VM credentials. 
   - For Windows VMs, the operating system details are always fetched from the vCenter Server.
- For Hyper-V VMs, the operating system data is gathered from the Hyper-V host
- For physical servers, it is fetched from the server.

## Next steps

[Create](how-to-create-assessment.md) or [customize](how-to-modify-assessment.md) an assessment.
