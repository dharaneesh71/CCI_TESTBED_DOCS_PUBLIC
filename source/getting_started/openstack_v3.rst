===================================
OpenStack Instance Deployment Guide
===================================

This guide details the standard procedure for deploying virtual machine (VM)
instances through the project dashboard. The workflow is split into three
sections based on the availability zone of the instance:

* :ref:`compute-deployment` — standard CPU workloads.
* :ref:`gpu-deployment` — GPU / AI workloads.
* :ref:`radio-deployment` — wireless / SDR workloads (includes USRP booking).

.. contents::
   :local:
   :depth: 2


New User Onboarding & Quotas
----------------------------

When a new user is approved, they receive their login credentials via email.

* **Initial Login**: Users must enter their Username, Password, and the specific
  Domain (e.g., ``default``) assigned by the OpenStack admin.
* **Resource Quotas**: The **Overview** tab displays the project's limits.
  For example, a project may be restricted to 10 instances, 20 VCPUs, and
  50 GB of RAM.
* **Quota Management**: Once these limits are reached, the system will prevent
  the creation of new instances.

.. note::

   Default SSH credentials for all Ubuntu-based snapshots in this guide are
   ``Username: ubuntu`` / ``Password: CCI@2025``. Change the password on
   first login for production workloads.


.. _compute-deployment:

===========================
Compute Instance Deployment
===========================

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
   </div>


1. Launch Procedure
-------------------

* Navigate to the **Project → Compute → Overview** dashboard.
* Click the **Instances** tab in the left-hand sidebar, then click
  **Launch Instance**.
* **Details**: Enter an Instance Name (e.g., ``test_compute``).
  Set the Availability Zone to ``compute``.
* **Source**: Choose **Instance Snapshot** from the dropdown menu.

  * Set **Create New Volume** to **No** (Ephemeral) for standard testing, or
    **Yes** (Persistent) to save critical data to Cinder storage.
  * Select your desired snapshot and click the **Up Arrow (↑)**.

* **Flavor**: Choose a flavor (e.g., ``m4.4.32``) and click the
  **Up Arrow (↑)**.
* **Networks**: Allocate the required network
  (e.g., ``Int/External-Network-Matrix25``).
* **Security Groups**: Allocate the ``default`` security group.
* **Launch**: Click **Launch Instance** and wait for the status to change to
  ``Active`` and power state to ``Running``.

.. tip::

   Monitor the instance row through the sequence
   ``Build`` → ``Spawning`` → ``Active``. If it stalls on ``Build`` for more
   than a few minutes, check your project quotas on the Overview tab.


2. Expanding Instance Storage
-----------------------------

If you launched with a disk size larger than the default Instance Snapshot,
you must manually expand the storage:

.. code-block:: bash

   sudo apt update
   sudo apt install -y cloud-guest-utils lvm2
   sudo growpart /dev/vda 3
   sudo pvresize /dev/vda3
   sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv


3. Network Connectivity & Security
----------------------------------

* **Floating IP**: Navigate to **Project → Compute → Instances**.
  Click **Associate Floating IP** next to your instance. Click **+** to
  allocate an IP from the ``Main-Internet-Network`` pool, then associate it.
* **Security Rules**: Navigate to **Network → Security Groups** and click
  **Manage Rules** for the ``default`` group. Add an **SSH** rule for the
  ``0.0.0.0/0`` CIDR.

.. warning::

   Allowing SSH from ``0.0.0.0/0`` exposes port 22 to the entire internet.
   For long-running instances, restrict the CIDR to a known IP range or use
   a key pair instead of password authentication.


4. Terminal Access & Verification
---------------------------------

Use a gateway node to SSH into the floating IP (gateway credentials provided
by admin):

.. code-block:: bash

   ssh ubuntu@[Floating_IP]

* **Default Password**: ``CCI@2025``
* **Verification Commands**:

  .. code-block:: bash

     ip a              # check assigned addresses
     ping 8.8.8.8      # internet connectivity test
     lsblk             # verify expanded block storage


.. _gpu-deployment:

=======================
GPU Instance Deployment
=======================

.. raw:: html

   <div class="demo-videos">
     <h3>How to Create and Configure a GPU Node</h3>
     <video controls preload="metadata" playsinline crossorigin="anonymous" width="640">
       <source src="../_static/GPU-instance.mp4" type="video/mp4">
       Your browser does not support the video tag.
     </video>
     <p><a href="../_static/GPU-instance.mp4">Download video</a></p>
   </div>

.. important::

   Before creating a GPU instance, you must request a GPU flavor by
   specifying the required GPU version via a ticket to the admin team on
   **Redmine**. Without an approved flavor, the launch step below will fail.


1. Launch Procedure
-------------------

* Navigate to the **Project → Compute → Overview** dashboard.
* Click the **Instances** tab in the left-hand sidebar, then click
  **Launch Instance**.
* **Details**: Enter an Instance Name (e.g., ``test_gpu``).
  Set the Availability Zone to ``gpu``.
* **Source**: Choose **Instance Snapshot** from the dropdown menu. Select
  your desired snapshot and click the **Up Arrow (↑)**.

  .. note::

     Use **Persistent** storage (Create New Volume = Yes) if you plan to
     retain large datasets across instance rebuilds.

* **Flavor**: Choose your approved GPU flavor and click the **Up Arrow (↑)**.
* **Networks**: Allocate the required network
  (e.g., ``Int/External-Network-Matrix25``).
* **Security Groups**: Allocate the ``default`` security group.
* **Launch**: Click **Launch Instance** and wait for the status to change to
  ``Active``.


2. Expanding Instance Storage
-----------------------------

.. code-block:: bash

   sudo apt update
   sudo apt install -y cloud-guest-utils lvm2
   sudo growpart /dev/vda 3
   sudo pvresize /dev/vda3
   sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv


3. Network Connectivity & Security
----------------------------------

* **Floating IP**: Navigate to **Project → Compute → Instances**. Click
  **Associate Floating IP**. Allocate an IP from the
  ``Main-Internet-Network`` pool, then associate it.
* **Security Rules**: Navigate to **Network → Security Groups** and click
  **Manage Rules** for the ``default`` group. Add an **SSH** rule for the
  ``0.0.0.0/0`` CIDR.


4. Terminal Access
------------------

SSH into the floating IP using the gateway node
(``Username: ubuntu`` / ``Password: CCI@2025``):

.. code-block:: bash

   ssh ubuntu@[Floating_IP]


5. GPU Configuration & Validation
---------------------------------

5.1 Install Drivers
~~~~~~~~~~~~~~~~~~~

Install the kernel headers, build tools, and NVIDIA driver stack
(version 535):

.. code-block:: bash

   sudo apt install -y linux-headers-$(uname -r) \
                       linux-modules-extra-$(uname -r) \
                       build-essential dkms ubuntu-drivers-common
   sudo apt install -y nvidia-driver-535 nvidia-dkms-535 \
                       nvidia-utils-535 libnvidia-compute-535
   sudo dkms autoinstall -k $(uname -r)
   sudo update-initramfs -u
   sudo reboot

.. important::

   The instance **must** reboot after driver installation. Reconnect via SSH
   once it returns to ``Active``.


5.2 Verify Drivers
~~~~~~~~~~~~~~~~~~

After reconnecting, confirm the GPU is detected:

.. code-block:: bash

   nvidia-smi

Expected output: a table listing the GPU model (e.g., Tesla V100), driver
version 535.x, and CUDA version.


5.3 Install PyTorch and Validate CUDA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   sudo apt update
   sudo apt install -y python3-pip
   python3 -m pip install --upgrade pip numpy
   python3 -m pip install torch --index-url https://download.pytorch.org/whl/cu121

Quick sanity check:

.. code-block:: bash

   python3 - <<'PY'
   import torch
   print("CUDA available:", torch.cuda.is_available())
   print("GPU name:", torch.cuda.get_device_name(0))
   PY


5.4 GPU Stress Test
~~~~~~~~~~~~~~~~~~~

Execute a 5-minute matrix multiplication loop to verify hardware stability:

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
   PY

In a second terminal, monitor utilization and check for hardware errors:

.. code-block:: bash

   watch -n 1 nvidia-smi
   sudo dmesg | grep -i "xid"

.. note::

   A successful test returns **no output** from the ``xid`` grep, meaning
   zero Xid errors were detected. GPU utilization should stay between
   90–100 % during the run.


.. _radio-deployment:

=========================
Radio Instance Deployment
=========================

.. raw:: html

   <div class="demo-videos">
     <h3>How to Create a Radio Node</h3>
     <video controls preload="metadata" playsinline crossorigin="anonymous" width="640">
       <source src="../_static/Radio-Instance.mp4" type="video/mp4">
       Your browser does not support the video tag.
     </video>
     <p><a href="../_static/Radio-Instance.mp4">Download video</a></p>
   </div>

Deploying a radio instance is a **two-stage process**:

1. **Book a USRP** on the CCI xG Testbed booking portal (Section 1 below).
2. **Launch a Radio VM** in the ``radio`` availability zone to act as the
   host for the booked USRP (Section 2 onward).

.. important::

   You must complete the USRP booking **before** launching the radio VM.
   The VM will not be able to communicate with an SDR unit you have not
   reserved.


1. Booking USRP Resources (SDR Booking)
---------------------------------------

This section walks through reserving — and cancelling — USRP nodes on the
CCI xG Testbed SDR Booking portal.

.. raw:: html

   <div class="demo-videos">
     <h3>How to Book a USRP</h3>
     <video controls preload="metadata" playsinline crossorigin="anonymous" width="640">
       <source src="../_static/usrp-booking.mp4" type="video/mp4">
       Your browser does not support the video tag.
     </video>
     <p><a href="../_static/usrp-booking.mp4">Download video</a></p>
   </div>

**Prerequisites**

* Your CCI xG Testbed login credentials (email/username and password).
* Your designated OpenStack project name (e.g., ``Internal-project-demo``).


1.1 Log In to the Portal
~~~~~~~~~~~~~~~~~~~~~~~~

* Navigate to the **CCI xG Testbed** login page.
* Enter your **Email/Username** and **Password**.
* Click the blue **Log In** button.


1.2 Open the SDR Booking Application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

* After login, you land on the **My Applications** dashboard.
* Click the **SDR Booking** tile to launch the booking system.


1.3 Validate Your OpenStack Project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A pop-up will appear prompting you to **Enter Your OpenStack Project**.

* Type your designated project name (e.g., ``Internal-project-demo``).
* Click **Validate** and wait for the green checkmark.
* Click **Continue**.

.. note::

   The project name is case-sensitive and must match the OpenStack project
   exactly. If validation fails, double-check the name against your
   OpenStack dashboard.


1.4 Select Date Range and Device
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On the **Book USRP Resources** page:

* Under **Select Date Range and Time**, use the calendar icons to pick a
  **From Date** and **To Date**.
* Scroll down to the **Available USRPs** section. Devices available for
  the selected dates appear as **green boxes**.
* Click the specific USRP unit you want to reserve (e.g., ``node 131``).


1.5 Configure and Confirm Your Booking
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A booking configuration window will pop up:

* Under **Select Time Slot**, click the 4-hour block you need
  (e.g., 4:00 PM – 8:00 PM).
* Confirm your **OpenStack project name** in the dropdown.
* Open **Select Frequency Band** and choose the operating frequency for
  your experiment (e.g., ``ISM: 2400–2483.5 MHz``).
* Tick the **I agree to the Acceptable Use Policy** checkbox.
* Click the green **Book USRP** button (bottom right).
* A **Reservation confirmed** screen will summarize the booking. Review
  the details and click **Close**.

.. tip::

   Slots are issued in 4-hour blocks. If you need a longer experiment,
   book consecutive blocks on the same USRP node to avoid having to
   re-flash or reconfigure between sessions.


1.6 Managing and Cancelling Reservations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To view or cancel an existing reservation:

* Click the **My Reservations** tab in the top navigation bar.
* Locate the reservation in the active list.
* To cancel, tick the checkbox on the far left of the reservation row.
* Click the red **Delete Selected** button on the right.
* In the confirmation pop-up, click the red **Cancel 1 Reservation**
  button to finalize.

The dashboard will then update to show **No Active Reservations**.

.. warning::

   Cancellations are immediate and cannot be undone. If the slot is in
   high demand, you may not be able to re-book the same node/time.


2. Launch Procedure (Radio VM)
------------------------------

Once your USRP is booked, launch the radio host instance:

* Navigate to the **Project → Compute → Overview** dashboard.
* Click the **Instances** tab, then click **Launch Instance**.
* **Details**: Enter an Instance Name. Set the Availability Zone to
  ``radio``.
* **Source**: Choose **Instance Snapshot** from the dropdown menu.
  Select your snapshot and click the **Up Arrow (↑)**.
* **Flavor**: Choose an appropriate radio flavor and click the
  **Up Arrow (↑)**.
* **Networks**: Allocate the required network
  (e.g., ``Int/External-Network-Matrix25``).
* **Security Groups**: Allocate the ``default`` security group.
* **Launch**: Click **Launch Instance** and wait for the status to change
  to ``Active``.


3. Expanding Instance Storage
-----------------------------

.. code-block:: bash

   sudo apt update
   sudo apt install -y cloud-guest-utils lvm2
   sudo growpart /dev/vda 3
   sudo pvresize /dev/vda3
   sudo lvextend -r -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv


4. Network Connectivity & Security
----------------------------------

* **Floating IP**: Navigate to **Project → Compute → Instances**. Click
  **Associate Floating IP**. Allocate an IP from the
  ``Main-Internet-Network`` pool, then associate it.
* **Security Rules**: Navigate to **Network → Security Groups** and click
  **Manage Rules** for the ``default`` group. Add an **SSH** rule for the
  ``0.0.0.0/0`` CIDR.


5. Terminal Access & SDR Verification
-------------------------------------

Use a gateway node to SSH into the floating IP
(``Username: ubuntu`` / ``Password: CCI@2025``):

.. code-block:: bash

   ssh ubuntu@[Floating_IP]

Once connected, verify networking and SDR connectivity:

.. code-block:: bash

   ping 8.8.8.8          # internet connectivity
   uhd_find_devices      # discover the booked USRP node

.. note::

   ``uhd_find_devices`` should list the USRP unit you reserved in
   :ref:`Section 1 <radio-deployment>`. If it returns nothing, confirm the
   booking is still active in **My Reservations** and that your VM is in
   the ``radio`` availability zone.


=================
Volume Management
=================

Creating a Volume
-----------------

1. Navigate to the left-hand sidebar and select **Volumes**, then click on
   **Volumes** from the dropdown.
2. Click the **+ Create Volume** button located near the top right of the
   dashboard.
3. Provide a name for the volume in the designated field.
4. Select your **Volume Source**. If you are building from a backup like
   in the video, select **Volume Snapshot** and choose the specific
   snapshot from the dropdown.
5. Specify the desired capacity in the **Size (GiB)** field (e.g., ``128``).
6. Click the blue **Create Volume** button to initialize the storage.


Extending a Volume (Increasing Size)
------------------------------------

1. From the **Volumes** list, locate the specific volume you want to
   resize.
2. Click the dropdown arrow next to **Edit Volume** in the **Actions**
   column.
3. Select **Extend Volume** from the menu.
4. Enter the new, larger capacity in the **New Size (GiB)** field (for
   example, upgrading from ``32`` to ``128``).
5. Click **Extend Volume** to apply the new size limit.


Attaching a Volume (Connecting to an Instance)
----------------------------------------------

1. From the **Volumes** list, locate the volume you want to connect.
2. Click the dropdown arrow in the **Actions** column for that volume.
3. Select **Manage Attachments**.
4. Choose the target instance from the dropdown menu where you want the
   volume mounted.
5. Click **Attach Volume** to link the storage to your virtual machine.