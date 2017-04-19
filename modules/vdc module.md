## Module: Advanced VDC Creation

Virtual Data Centers (VDC) are a critical component to UCS Director, but are only involved in the consumption of virtual resources (i.e., VMware, Hyper-V, even AWS).  You can see the options for which clouds can be managed by a VDC when you create a new VDC: ![alt text][image_01]

Effectively, a VDC enables the UCSD admin to apply policy to how virtualized infrastructure can be consumed by the end user.  As such, think of a VDC as simply an envelope for policy.   Even basic deployments of a VM required that you created an VDC - along with an associated System, Compute, Network, and Storage policy.  

[image_01]: https://github.com/blairhicks/ucsd-training-lab/blob/master/images/ucsd_image_01.jpg

Two advanced features of VDCs that can be extremely valuable are **Categories** and **Service Profiles**.  Service Profiles are leveraged in the automated creation of new VDCs - which will be done as we create new Application Network Profiles.  Categories become an effective way of applying different policies while retaining membership in the same VDC.

ACI Construct | UCS Director Construct
--- | ---
Tenant | Group
ANP | VDC
EPG | Category

### Lab Exercise
1.  Clone an existing VDC `Southfield Private vCloud` to a VDC of your own choosing.  
Select the new VDC and click `Manage Categories` on the top ribbon.  Validate that you can select different policies for System, Compute, Network, etc for the different categories.
2.  On the VDC Screen, click on the vDC Service Profiles Tab.  Clone the `Generic_Tenant` vDC Profile to a new one named `<your initials>_VDC_Profile`

---

VM deployment requires the association of a standard catalog with a VDC.  You are probably familiar with creating standard catalogs and placing them into folders.  For this next step, you will want to create a new group (use the provided workflow) and create a new standard catalog and assign it to your group.
### Lab Exercise
1.  Review (clone if you wish) the `Add_UCSD_Group` workflow.  The first task is actually a custom task (written in CloupiaScript) by Hank Preston - part of his EZ Cloud Utilities package (<https://communities.cisco.com/docs/DOC-65217>).  The custom task can be viewed in the Custom Task tab.
2. Go ahead and execute the script and create a new UCSD Group.  Name you group whatever you like, but do establish a unique three character code.
3. Create (or clone) a new standard catalog - based off the `demo-CentOs` image.  Be sure to check **End User can Override Category**.  
4. Launch a deployment of your catalog - linking it to the VDC you created earlier.

---
This basic setup will prepare you to leverage a APIC controlled VDC.  The next step in the lab is to work through the ACI Deployment workflows.

## Module: Network Setup
One of the initial steps to effectively demonstrating network automation is to have a IP Subnet plan.  For the lab, the 19.168.67.0 network has been subdivided into 8 separate subnets.  When you on-board a new group, you will be reserving one of those subnets.

One of the key design decisions is around IP addressing.  That is outside the scope of automation, but UCS Director does provide you with tools, such as the Virtual IP Subnet Pool Policy that can help manage IP addressing.  It is also possible to defer all IP address assignments to a third party IPAM (such as Infoblox).

### Lab Exercise
1. Review the **IP Subnet Pool Policy** - Policies : Virtual/Hypervisor Policies : Network.  You can create your own Policy, but recognize that it may not be route-able without prior design configuration.











