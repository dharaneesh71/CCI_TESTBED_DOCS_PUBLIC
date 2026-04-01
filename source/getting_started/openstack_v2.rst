============================================
Creating an Instance (Compute / GPU / Radio)
============================================

This guide details the standard procedure for deploying a virtual machine (VM) instance through the project dashboard. This workflow applies to Compute, GPU, and Radio instances.

.. contents::
   :local:
   :depth: 2

Tutorial videos
-----------

.. raw:: html

   <div class="demo-videos">
     <h3>How to Create a Compute Node</h3>
     <video controls preload="metadata" playsinline crossorigin="anonymous" width="640">
       <source src="../_static/Compute.mp4" type="video/mp4">
       Your browser does not support the video tag.
     </video>
     <p><a href="../_static/Compute.mp4">Download video</a></p>

     <h3>How to Associate a Floating IP</h3>
     <video controls preload="metadata" playsinline crossorigin="anonymous" width="640">
       <source src="../_static/Floating-IP.mp4" type="video/mp4">
       Your browser does not support the video tag.
     </video>
     <p><a href="../_static/Floating-IP.mp4">Download video</a></p>

     <h3>How to Create a Radio Node</h3>
     <video controls preload="metadata" playsinline crossorigin="anonymous" width="640">
       <source src="../_static/Radio-Instance.mp4" type="video/mp4">
       Your browser does not support the video tag.
     </video>
     <p><a href="../_static/Radio-Instance.mp4">Download video</a></p>
   </div>



Instance Launch Procedure
=========================

1. Initiate the Launch
----------------------

- Navigate to the **Project → Compute → Overview** dashboard.
- Click the **Instances** tab in the left-hand sidebar.
- Click the **Launch Instance** button located in the top-right corner of the screen.

2. Configure Details
--------------------

The **Details** tab is the first step in the wizard.

- **Instance Name**: Enter a unique identifier for the VM (e.g., ``test_instance``).
- **Availability Zone**: Select the appropriate zone based on your workload:
  
  - Select ``compute`` for standard processing.
  - Select ``gpu`` for graphics/AI workloads.
  - Select ``radio`` for wireless/SDR workloads.

- **Count**: Ensure the count is set to 1 (or adjust as needed).
- **Action**: Click the **Next >** button or select the **Source** tab to proceed.

3. Select Source (Storage & Snapshot)
-------------------------------------

In the **Source** tab, configure the boot source.

- **Select Boot Source**: Choose **Instance Snapshot** from the dropdown menu.
- **Create New Volume**:
  
  - **Option A (Ephemeral)**: Set to **No**. This creates the root disk on the local host. This is the standard configuration for stateless workloads or testing.
  - **Option B (Persistent)**: Set to **Yes**. This creates the root disk on persistent storage (Cinder). Use this for critical data that must survive instance deletion.

- **Allocated**: Locate the desired snapshot in the "Available" list.
- **Action**: Click the **Up Arrow (↑)** button next to the snapshot to move it to the "Allocated" section.
- **Action**: Click **Next >** or select the **Flavor** tab.

4. Select Flavor (Resources)
----------------------------

In the **Flavor** tab, define the compute resources (CPU/RAM/Disk).

- Browse the list for a flavor that meets the hardware requirements (e.g., ``m4.4.32`` which provides 4GB RAM and 32GB Total Disk).
- **Action**: Click the **Up Arrow (↑)** button next to the chosen flavor to move it to the "Allocated" section.
- **Action**: Click **Next >** or select the **Networks** tab.

5. Configure Networks
---------------------

In the **Networks** tab, connect the instance to the project network.

- Locate the required network in the "Available" list (e.g., ``Int/External-Network-Matrix25``).
- **Action**: Click the **Up Arrow (↑)** button next to the network to move it to the "Allocated" section.
- **Action**: Click **Next >** or select the **Security Groups** tab.

6. Security Groups
------------------

In the **Security Groups** tab, assign firewall rules.

- Verify that the default security group is listed under the "Allocated" section.
- If it is not allocated, locate it in the "Available" list and click the **Up Arrow (↑)** button.

.. note::

   Optional tabs such as Key Pair, Configuration, and Server Groups may be skipped unless specific configurations are required.

7. Launch and Verify
--------------------

- **Action**: Click the blue **Launch Instance** button in the bottom-right corner of the dialog window.
- You will be redirected to the Instances list.
- **Verify Status**: Monitor the instance row as it initializes.
  
  - **Status**: Wait for the sequence ``Build`` → ``Spawning`` → ``Active``.
  - **Power State**: Ensure the state reads ``Running``.


Network Connectivity & SSH Access
=================================

8. Assigning a Floating IP
--------------------------

Once an instance is launched, additional steps are required to enable external access via SSH:

- **Allocate a Floating IP**: Navigate to **Project → Compute → Instances**.
- **Assign Floating IP**: Select **Associate Floating IP** from the actions dropdown menu.
- **Select Pool**: Click the **+** button to allocate an IP from the ``Main-Internet-Network`` pool.
- **Associate**: Choose the newly allocated IP (e.g., ``172.167.0.221``) and associate it with the internal IP of your instance.

9. Assigning Firewall Rules
---------------------------

To allow SSH traffic, you must update the Security Group rules:

- **Manage Rules**: Navigate to **Network → Security Groups** and click **Manage Rules** for the default group.
- **Add SSH Rule**: Click **Add Rule**, select **SSH** from the Rule dropdown, and ensure it is set for the ``0.0.0.0/0`` CIDR to allow access.


Advanced Storage & Verification
===============================

10. Creating and Attaching Persistent Volumes
---------------------------------------------

For workloads requiring data to survive instance deletion, use Option B (Persistent) during the source selection or create a volume manually:

- **Create Volume**: Navigate to **Project → Volumes → Volumes** and click **Create Volume**.
- **Configure Volume**: Provide a name (e.g., ``Demo-Volume``), select a source image (e.g., ``Ubuntu-22``), and set the size to a minimum of 32GB (matching the image size).
- **Attach to Instance**: During the Launch Instance wizard, under the **Source** tab, select **Volume** as the Boot Source and allocate your created volume.
- **Deletion Policy**: Set **Delete Volume on Instance Delete** to **No** if you wish to retain the data after the VM is removed.

11. Verification and Terminal Access
------------------------------------

To verify the setup and access the instance terminal:

- **Console Access**: From the Instances list, select **Console** from the dropdown to access the web-based terminal.
- **SSH Login**: Use the provided credentials to log in. Default credentials for Ubuntu images are often:
  
  - **User**: ``ubuntu``
  - **Password**: ``CCI@2025``

- **Network Check**: Run the following commands to check assigned addresses or verify internet connectivity:

  .. code-block:: bash

     ip a
     ping 8.8.8.8

  To verify attached block storage volumes:

  .. code-block:: bash

     lsblk


New User Onboarding & Quotas
============================

12. Dashboard Overview
----------------------

When a new user is approved, they receive their login credentials via email.

- **Initial Login**: Users must enter their Username, Password, and the specific Domain (e.g., ``default``) assigned by the OpenStack admin.
- **Resource Quotas**: The **Overview** tab displays the project's limits. For example, a project may be restricted to 10 instances, 20 VCPUs, and 50GB of RAM.
- **Quota Management**: Once these limits are reached, the system will prevent the creation of new instances.


Remote Access Security Configuration
====================================

13. Terminal Access via Gateway
-------------------------------

To access the instance from an external terminal or copy-paste commands, you must configure a Floating IP and Security Group Rules (detailed in sections 8 and 9). You can then access it via a remote terminal:

- **Gateway access**: Use a gateway node to SSH into the floating IP. Gateway credentials will be shared by the admin.

  .. code-block:: bash

     ssh ubuntu@[Floating_IP]

- **System Verification**: 
  Log in using ``Username: ubuntu`` and ``Password: CCI@2025``.