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

8. Expanding Instance Storage
-----------------------------

When you launch any instance with a disk size larger than the default image, the operating system does not automatically use the extra space. You must manually instruct the system to expand into the full capacity of your disk.

Update Package Lists and Install Utilities:
Install the necessary tools for managing disk partitions.

.. code-block:: bash

   sudo apt update
   sudo apt install -y cloud-guest-utils lvm2

Grow the Partition and Resize the Physical Volume:
Stretch your main partition (/dev/vda3) to fill the empty disk space, then tell the system to recognize this newly added space.

.. code-block:: bash

   sudo growpart /dev/vda 3
   sudo pvresize /dev/vda3

Extend the Logical Volume:
Expand your main filesystem to use 100% of the available free space. The -r flag ensures the filesystem is resized immediately so you can start saving files there right away.

.. code-block:: bash

   sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv


Network Connectivity & SSH Access
=================================

9. Assigning a Floating IP
--------------------------

Once an instance is launched, additional steps are required to enable external access via SSH:

- **Allocate a Floating IP**: Navigate to **Project → Compute → Instances**.
- **Assign Floating IP**: Select **Associate Floating IP** from the actions dropdown menu.
- **Select Pool**: Click the **+** button to allocate an IP from the ``Main-Internet-Network`` pool.
- **Associate**: Choose the newly allocated IP (e.g., ``172.167.0.221``) and associate it with the internal IP of your instance.

10. Assigning Firewall Rules
----------------------------

To allow SSH traffic, you must update the Security Group rules:

- **Manage Rules**: Navigate to **Network → Security Groups** and click **Manage Rules** for the default group.
- **Add SSH Rule**: Click **Add Rule**, select **SSH** from the Rule dropdown, and ensure it is set for the ``0.0.0.0/0`` CIDR to allow access.


Advanced Storage & Verification
===============================

11. Creating and Attaching Persistent Volumes
---------------------------------------------

For workloads requiring data to survive instance deletion, use Option B (Persistent) during the source selection or create a volume manually:

- **Create Volume**: Navigate to **Project → Volumes → Volumes** and click **Create Volume**.
- **Configure Volume**: Provide a name (e.g., ``Demo-Volume``), select a source image (e.g., ``Ubuntu-22``), and set the size to a minimum of 32GB (matching the image size).
- **Attach to Instance**: During the Launch Instance wizard, under the **Source** tab, select **Volume** as the Boot Source and allocate your created volume.
- **Deletion Policy**: Set **Delete Volume on Instance Delete** to **No** if you wish to retain the data after the VM is removed.

12. Verification and Terminal Access
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


13. GPU Instance Configuration (GPU Zones Only)
-----------------------------------------------

If you launched your instance in the ``gpu`` availability zone, you must perform additional configuration steps to expand your storage for large datasets, install the necessary NVIDIA drivers, and validate the hardware stability.

13.1 GPU Driver Installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The operating system requires specific NVIDIA drivers and kernel modules to communicate with the allocated GPU hardware.

**Install Build Dependencies:**

Ensure the system has the required kernel headers and build essentials to compile the driver modules.

.. code-block:: bash

   sudo apt install -y linux-headers-$(uname -r) linux-modules-extra-$(uname -r) build-essential dkms ubuntu-drivers-common

**Install NVIDIA Drivers and Toolkit:**

Install the version 535 drivers along with the necessary compute libraries.

.. code-block:: bash

   sudo apt install -y nvidia-driver-535 nvidia-dkms-535 nvidia-utils-535 libnvidia-compute-535

**Compile Modules and Update Boot File:**

Configure Dynamic Kernel Module Support (DKMS) to ensure drivers survive kernel updates, update the boot files, and restart the instance.

.. code-block:: bash

   sudo dkms autoinstall -k $(uname -r)
   sudo update-initramfs -u
   sudo reboot

13.2 GPU Verification and Stress Testing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After the instance reboots, reconnect via SSH to validate the GPU installation and ensure hardware stability under machine learning workloads.

**Verify Driver Initialization:**

Check the NVIDIA System Management Interface to confirm the GPU (e.g., Tesla V100) and drivers are detected successfully.

.. code-block:: bash

   nvidia-smi

**Install PyTorch and Dependencies:**

Set up the Python environment required for CUDA testing.

.. code-block:: bash

   sudo apt update
   sudo apt install -y python3-pip
   python3 -m pip install --upgrade pip
   python3 -m pip install numpy
   python3 -m pip install torch --index-url https://download.pytorch.org/whl/cu121

**Validate CUDA Availability:**

Run a short Python script to verify PyTorch can communicate with the GPU.

.. code-block:: bash

   python3 - <<'PY'
   import torch
   import numpy
   print("CUDA available:", torch.cuda.is_available())
   print("GPU name:", torch.cuda.get_device_name(0))
   print("NumPy version:", numpy.__version__)
   PY

Expected Output: ``CUDA available: True``, followed by your GPU name.

**Execute a GPU Stress Test:**

Run an intensive matrix multiplication loop for 5 minutes (300 seconds) to verify hardware stability under peak load.

.. code-block:: bash

   python3 - <<'PY'
   import torch, time

   assert torch.cuda.is_available(), "CUDA is not available"

   device = "cuda"
   print("GPU:", torch.cuda.get_device_name(0))

   size = 12000
   a = torch.randn(size, size, device=device)
   b = torch.randn(size, size, device=device)

   end = time.time() + 300
   count = 0
   while time.time() < end:
       c = torch.matmul(a, b)
       torch.cuda.synchronize()
       count += 1
   print("Completed matrix multiplications:", count)
   print("GPU stress test completed successfully")
   PY

**Monitor Performance and Check for Errors:**

While the stress test is running, open a second SSH terminal and run ``watch -n 1 nvidia-smi`` to ensure GPU utilization reaches 90% - 100%. Afterward, check the system kernel logs to ensure no hardware errors (Xid) occurred.

.. code-block:: bash

   sudo dmesg | grep -i "xid"

Note: A successful test will return no output, meaning zero Xid errors were detected.


New User Onboarding & Quotas
============================

14. Dashboard Overview
----------------------

When a new user is approved, they receive their login credentials via email.

- **Initial Login**: Users must enter their Username, Password, and the specific Domain (e.g., ``default``) assigned by the OpenStack admin.
- **Resource Quotas**: The **Overview** tab displays the project's limits. For example, a project may be restricted to 10 instances, 20 VCPUs, and 50GB of RAM.
- **Quota Management**: Once these limits are reached, the system will prevent the creation of new instances.


Remote Access Security Configuration
====================================

15. Terminal Access via Gateway
-------------------------------

To access the instance from an external terminal or copy-paste commands, you must configure a Floating IP and Security Group Rules (detailed in sections 9 and 10). You can then access it via a remote terminal:

- **Gateway access**: Use a gateway node to SSH into the floating IP. Gateway credentials will be shared by the admin.

  .. code-block:: bash

     ssh ubuntu@[Floating_IP]

- **System Verification**: 
  Log in using ``Username: ubuntu`` and ``Password: CCI@2025``.