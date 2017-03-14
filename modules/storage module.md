## Automation of Storage with UCS Director

One of the most powerful use cases for UCS Director is it's ability to automate across storage systems from many different vendors.  The UCS Director Engineering team manages a host of storage vendor integrations - check the latest [Hardware Compatibility Matrix](http:/​/​www.cisco.com/​c/​en/​us/​support/​servers-unified-computing/​ucs-director/​products-device-support-tables-list.html).  As a single UCS Director license covers a connection to a storage device (think a controller that has a unique network ID) regardless of the amount of disk attached to that storage device - even a few dozen UCS Director licenses can automate a significant portion of a customer's storage environment.  In addition, a number of partners (such as Pure and Hitachi) have partnered with Cisco to develop their own UCS Director plug-ins.  

### Example system with multiple storage vendors ###
Insert Image

## Working with Storage Systems
Storage systems are generally registered as Physical Accounts.  Two exceptions are EMC RecoverPoint and EMC VPLEX which are treated as Multi-Domain Managers.

### Lab Exercises
In your lab system, browse to a connected storage system under Administration / Physical Accounts.  Review the different connection configurations for EMC VNX, EMC XtremIO, NetApp, etc.  Note that some storage systems require helper applications (such as NaviSecCli) to be in place.  
* Notice the Connection Status column for each storage device.  This indicator can be misleading.  A 'Success' means that the last connection validation test was successful.  It does not mean that UCS Director is currently able to access the storage device.  Run `Test Connection` to validate the connection.  If the connection fails (reasons can include authentication failure, network connectivity, etc) only then will UCS Director actually update the connection status to `Failed`.
* Storage devices can consume one or many UCS Director licenses, based on the actual number of controllers associated with the storage device.  Go to the `License Utilization` screen to determine how many actual licenses are being consumed.  Go to the `Resource Usage Data` screen to see how those licenses are allocated across different storage devices.
* Check the Overview table for the storage device under Physical / Storage.  This screen can identify when the device was last polled by the UCS Director system.
* In order to troubleshoot possible connectivity problems between UCS Director and a storage system, look to the `/opt/infra/inframgr/logfile.txt ` file on the UCS Director appliance.  Capture the login attempt by running `tail -f /opt/infra/inframgr/logfile.txt | tee /tmp/logoutput.txt`.  

## Storage Dashboard
Prior to developing automation workflows for a storage system, it is useful to perform various operations through the Operations interface.  For many storage systems, UCS Director provides the ability to perform manual operations without having to log into the element manager.  The downside of this approach (as with any manual approach) is that the work is not automated and is not tied to a Service Request.  
### Lab Exercises
Using the NetApp array, perform the following steps:
* Create an Export Policy - Name it something unique
* Create an Export Rule for your Policy.  Set the access so that only your assigned ESXi storage vmk can access.
* Create a NFS volume (in the nfs SVM) and attach it to the **Netapp 02** Aggregate.  Use a random size in GB and include your initials in the Name.  Use your Export Policy.
* Mount the NFS volume - keep track of the Junction Path.
* (Optional) - Go into vSphere and attempt to attach the NFS datastore to your assigned ESXi host.
* Now - go back and unmount and destroy the NFS volume.

The point of that exercise was to make sure that you understood the steps needed to create and present a new NFS share to vSphere.  Now repeat that process by building a workflow and using the following OOTB tasks:
+ Create Cluster Flexible Volume
+ Mount Cluster Volume
+ Associate Cluster Volume as NFS Datastore

Execute your workflow and verify that the new datastore is attached to your ESXi host (either from the vSphere Web Client or from UCS Director)
Examine the associated Service Request - pay close attention to the Log and to the Task input/output.
**Roll back your workflow.**

## Managing Fibre Channel Storage
Managing Fibre Channel storage with UCS Director is an extremely powerful use case for automation.  Whenever possible, incorporate this use case in any enablement / proof-of-value session that you are conducting with customers.  Preparation is key - if the SAN design isn't fully baked, your automation workflows are not going to be valid and you can wind up trying to troubleshoot in an unfamiliar lab environment.  Be cautious of entering into a lab environment where the storage setup is questionable.  Make sure that the storage can be zoned to the test servers manually before attempting to do it through a workflow.

### Lab Exercise
Time to work with a Fibre Channel workflow.  First - review the configuration of your ESXi host - verify that it has two HBAs.  By default, Fibre Channel is not used in the SFL lab (the NetApp array is considered non-production - it apparently has a habit of shutting down due to power constraints).  **Shout out to Tyson Scott and Dan Williams for helping me get FC working in Southfield!**
For the lab exercise - you need to build a workflow that addresses the following tasks:
* Select UCS Service Profile
* Generic Configure SAN Zoning
* Create Cluster Flexible Volume
* Create Cluster LUN
* Create Cluster Initiator Group
* Add the Server HBAs to the Initiator Group (one task for each HBA)
* Map the LUN to the Initiator Group

#### Select UCS Service Profile
Use the server that has been assigned - and the one where you have validated that the HBAs are attached.  Create a Workflow User Input labeled `Server Name`.  You can set an Admin Value, but do not encode the Server Selection into the task (Task Inputs).

#### Generic Configure SAN Zoning
This task is the key to most zoning operations.  There are actually two Generic Configure SAN Zoning tasks - one for MDS and one for Brocade.  Be sure to select the right one (we are using MDS switches in Southfield).
* For Service Profile, you should select the SERVICE_PROFILE_IDENTITY from the Select UCS Service Profile task.
* Select vHBA (first one) - use SP_VHBA1 from the Select Server task.
* VSAN ID (first one) - use SP_VSAN1 from the Select Server task.
* Storage FC Adapter (Primary) (first one) - create a Workflow User Input for this variable.  Label the variable "Fabric A Storage Adapter".  Use the **fcp_lif_netapp01_1d**.
* Select Device - create a Workflow User Input for this variable.  Label the variable "SAN FABRIC SWITCH A" and set it to cdc-r4-mds1
* ---
* Select vHBA (second one) - use SP_VHBA2 from the Select Server task.
* VSAN ID (second one) - use SP_VSAN2 from the Select Server task.
* Storage FC Adapter (Primary) (second one) - create a Workflow User Input for this variable.  Label the variable "Fabric B Storage Adapter".  Use the **fcp_lif_netapp01_1b**.
* Select Device - create a Workflow User Input for this variable.  Label the variable "SAN FABRIC SWITCH B" and set it to cdc-r4-mds2
*  ---
* On the **Task Inputs** screen, configure the following:
[Insert Image]
  * Service Profile
  * Check `Activate Zone Set` and `Commit Zone`
  * For **Device Alias Fab A vHBA** input `${Server Name}_${SR_ID}_HBA1`
  * For **Zone Name** input `${Server Name}_${SR_ID}_ZoneA`
  * Storage Account Type - `NetApp ONTAP`
  * Storage Account Name (Primary) - `cdc-r4-netapp`
  
  Be sure to check Configure Fabric B
-
  * For **Device Alias Fab B vHBA** input `${Server Name}_${SR_ID}_HBA2`
  * For **Zone Name** input `${Server Name}_${SR_ID}_ZoneB`
  * Storage Account Type - `NetApp ONTAP`
  * Storage Account Name (Primary) - `cdc-r4-netapp`
  * Check `Copy Running configuration to Startup configuration
  
  **One thing to bear in mind - multiple concurrent operations on the same MDS switch can lead to problems.  We will address this. **
  
#### Create Cluster Flexible Volume
  * Safe to go ahead and put these on the Task Inputs screen.
  [Insert Screen Shot]
  
#### Create Cluster LUN
  * For Volume Name, use OUTPUT_CLUSTER_VOLUME_IDENTITY from previous task.
  
  Task Input Screen
  * LUN Name should be `${Server Name}_${SR_ID}_LUN`
  * Size (use a value smaller than the volume size
  * OS Type - vmware
  
#### Create Cluster Initiator Group
  * SVM Name - fcp
  * Initiator Group Name - `${Server Name}_${SR_ID}_IG`
  * Group Type - FCP
  * OS Type - vmware
  * Portset Name `BH_Test1_Portset`
  
#### Add Initiator to Cluster Initiator Group (twice - once for each HBA)
* Select the OUTPUT_CLUSTER_IGROUP_IDENTITY from the previous task
* Use **SP_VHBA1_WWPN** or **SP_VHBA2_WWPN** depending on which HBA you are adding 

#### Map Cluster LUN to iGroup
* Select OUTPUT_CLUSTER_LUN_IDENTITY
* Select OUTPUT_CLUSTER_IGROUP_IDENTITY
* Do not specify a LUN ID (this is not a boot lun)

**Run your workflow - troubleshoot and validate by rescanning for new LUNs on the ESXi host**

**Roll back your workflow when all is good**

## Custom Approval
Now is a good time to insert a Custom Approval into your workflow.  Custom Approvals are particularly useful with Storage Automation tasks where someone wants to manual assign different elements - such as the network interface.

### Lab Exercise 
Create a new Custom Approval Task.  Add two Input Fields - both for Generic Storage FC Adapter List - label them `Fabric A Target Port` and `Fabric B Target Port`
Insert the Custom Approval Task into your previous workflow - before the **Generic Configure SAN Zoning** task
Use blahicks@cisco.com as the Approver (during the class).  This will help ensure that we are not overwriting each other's MDS updates. 



  
 








