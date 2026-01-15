==================
OpenStack Guide V1
==================

This page outlines the **next-iteration enhancements** planned for the OpenStack documentation and training material.  
Use this as a companion to the existing ``OpenStack Instance Launch Guide`` and as a checklist for future content and video creation.

.. contents::
   :local:
   :depth: 2


Instance Creation Workflows
===========================

Separate Blocks for Compute, Radio, and GPU Instances
-----------------------------------------------------

The existing guide already explains how to create different instance types.  
In this V1 guide, we will maintain **separate, clearly labeled blocks** that users can follow independently:

- **Compute Instance (Default AZ: ``compute``):**
  
  - Focus on standard CPU-only workloads.
  - Emphasize choosing **Create New Volume = Yes** for data safety.
  - Include a compact step-by-step table or checklist (to be added) derived from the main guide.

- **Radio Instance (Default AZ: ``radio``):**
  
  - Explain that radio instances are used with **USRP-attached networks**.
  - Highlight the additional **USRP network port** and the need for:
    - Admin approval for USRP access.
    - Network configuration inside the instance (netplan, jumbo MTU, etc.).
  - Link back to the **CLI radio instance example** in the main OpenStack guide.

- **GPU Instance (Default AZ: ``gpu``):**
  
  - Clarify quota and approval requirements for GPU usage and make sure you have access to Nvidia GPU.
  - Provide a short checklist for:
    - Selecting a GPU flavor.
    - Verifying CUDA / GPU visibility inside the instance.


Instance Creation Videos
========================

Video: Creating an Instance **Without** a Volume
-----------------------------------------------

This planned video will demonstrate:

- Launching a small **ephemeral instance** with **Create New Volume = No**.
- Showing that:
  - The root disk is tied to the host.
  - Deleting the instance **permanently deletes** all data on that instance.
- Recommended use cases:
  - Short-lived test workloads.
  - Stateless services where data is stored externally (e.g., object storage, Git).

Video: Creating an Instance **With** a Volume
--------------------------------------------

A second video will demonstrate:

- Launching an instance with **Create New Volume = Yes** and explaining the two options:
  - **Delete Volume on Instance Delete = Yes** (cleanup-oriented).
  - **Delete Volume on Instance Delete = No** (data re-use / long-term storage).
- Pros and cons:
  - **Pros:**
    - Data survives host failures (stored on Cinder).
    - Volumes can be re-attached to new instances.
  - **Cons:**
    - Slightly more overhead in provisioning.
    - Requires quota management for volumes.

.. note::

   Once the videos are recorded, they can be embedded here similar to the
   existing videos in ``getting_started/openstack.rst`` using ``.. raw:: html``
   and the ``<video>`` tag.


Accessing Instances Through the Gateway
=======================================

Overview
--------

Most OpenStack projects are only reachable from within the **CCI xG Testbed network**.  
Access typically happens in **two hops**:

1. SSH into the **gateway node** from your local machine.
2. From the gateway node, SSH into your OpenStack instance.

Step-by-Step Access Flow
------------------------

1. **SSH to the Gateway Node:**

   .. code-block:: bash

      ssh <username>@<gateway-ip>

2. **From the Gateway, SSH to the Instance:**

   - If your instance has a **Floating IP**:

     .. code-block:: bash

        ssh ubuntu@<floating-ip>

   - If your instance only has an **internal IP**:

     .. code-block:: bash

        ssh ubuntu@<internal-ip>

3. **Key Management and Security:**

   - Use SSH keys instead of passwords where possible.
   - Keep keys private and revoke them immediately if compromised.

.. note::

   See the main OpenStack guide section on **Floating IPs** and **Security Groups**
   for details on making sure SSH (port 22) is allowed from the gateway or other
   management nodes.


Creating Zun Containers
=======================

Zun is the OpenStack service for running application containers. This section provides detailed instructions for creating containers, configuring networking, and enabling SSH access.

1. Creating a Container
----------------------

Use the following command to create an Ubuntu container in interactive mode and attach it to a specific network:

.. code-block:: bash

   zun create --name ubuntu-shell --net network=Zun-Network --interactive ubuntu:22.04 bash

**Explanation:**

- ``--name ubuntu-shell``: Container name
- ``--net network=Zun-Network``: Attach to OpenStack network
- ``--interactive``: Start an interactive shell
- ``ubuntu:22.04``: Base image
- ``bash``: Default shell

2. Identifying Container Port
------------------------------

Find the Neutron port created for your container:

.. code-block:: bash

   openstack port list --device-id <container-id>

Example output shows the port ID and fixed IP.

3. Creating and Associating a Floating IP
------------------------------------------

**Step 1: Create a Floating IP**

.. code-block:: bash

   openstack floating ip create Main-Internet-Network

**Step 2: Associate Floating IP with the container port**

.. code-block:: bash

   openstack floating ip set --port <port-id> <floating-ip-id>

4. Configuring Security Groups
------------------------------

Check attached security group and ensure SSH (22) and ICMP are allowed:

.. code-block:: bash

   openstack port show <port-id>
   openstack security group show <sg-id>
   openstack security group rule create --proto icmp <sg-id>
   openstack security group rule create --proto tcp --dst-port 22 <sg-id>

5. Installing and Enabling SSH Inside the Container
----------------------------------------------------

Access the container and install SSH:

.. code-block:: bash

   zun exec -it ubuntu-shell bash
   apt update
   apt install -y openssh-server
   service ssh start
   passwd root

To allow root SSH login:

.. code-block:: bash

   echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
   service ssh restart

6. Testing External SSH Access
-------------------------------

From a public machine:

.. code-block:: bash

   ssh root@<floating_ip>

Summary Table
-------------

.. list-table:: Zun Container Setup Steps
   :header-rows: 1
   :widths: 10 30 60

   * - Step
     - Action
     - Command
   * - 1
     - Create container
     - ``zun create --name ubuntu-shell --net network=Zun-Network --interactive ubuntu:22.04 bash``
   * - 2
     - Get port ID
     - ``openstack port list --device-id <container-id>``
   * - 3
     - Create Floating IP
     - ``openstack floating ip create Main-Internet-Network``
   * - 4
     - Associate Floating IP
     - ``openstack floating ip set --port <port-id> <fip-id>``
   * - 5
     - Install SSH
     - ``apt install -y openssh-server; service ssh start``
   * - 6
     - Connect via SSH
     - ``ssh root@<floating_ip>``


Managing Security Group Protocols
=================================

This section builds on the **Verifying Security Groups in OpenStack** part of the main guide,
but focuses specifically on **protocol and port management**.

Adding Protocol Rules
---------------------

1. Navigate to **Project → Network → Security Groups**.
2. Select your security group and open the **Manage Rules** page.
3. Click **Add Rule** and choose:

   - **Rule Type**: Custom TCP, Custom UDP, or Other Protocol.
   - **Port Range**:
     - e.g., ``22`` for SSH, ``80``/``443`` for HTTP/HTTPS,
       or a range like ``49152-65535`` for high ports.
   - **Remote**:
     - CIDR (e.g., ``0.0.0.0/0`` for all, or a restricted subnet).

4. Save the rule and confirm that it appears in the rule list.

Best Practices
--------------

- Allow **only the minimum required ports**.
- Prefer restricting **source CIDR** instead of opening to the entire internet.
- Reuse common security groups (e.g., ``ssh-only``, ``http-only``) across projects.


Controlling USRPs from OpenStack
================================

This section explains how to create a radio instance and attach a USRP port to enable communication with USRP devices.

Creating a Radio Instance
-------------------------

Create the instance as usual, but select **radio** in the availability zone when launching the instance through the dashboard or CLI.

Finding the Instance ID
------------------------

After creating the radio instance, list all instances to find the instance ID:

.. code-block:: bash

   openstack server list

Example output:

.. code-block:: text

   +--------------------------------------+---------+--------+------------------------------------------------------------+-----------------+------------+
   | ID                                   | Name    | Status | Networks                                                    | Image           | Flavor     |
   +--------------------------------------+---------+--------+------------------------------------------------------------+-----------------+------------+
   | 80ae757e-11c9-465b-a582-bf6f9c63be84 | Main    | ACTIVE | Internal-Project-Harshit-Network=10.0.0.32, 172.167.0.210 | Main-Harshit    | m1.8xlarge |
   | a4c38d16-6d56-43c3-b306-fdbd293f885a | Radio-2 | ACTIVE | Internal-Project-Harshit-Network=10.0.0.151, 172.167.0.236 | Radio-2-Harshit | m1.8xlarge |
   | 39d285c1-ccec-48ee-ab92-7dbe44f2f1bd | Radio-1 | ACTIVE | Internal-Project-Harshit-Network=10.0.0.49, 172.167.2.110  | Radio-1-Harshit | m1.8xlarge |
   +--------------------------------------+---------+--------+------------------------------------------------------------+-----------------+------------+

Copy the **Instance ID** of the instance created using the "Radio" availability zone. In the example above, it is ``39d285c1-ccec-48ee-ab92-7dbe44f2f1bd`` for Radio-1.

Finding the USRP Port
---------------------

Next, look for the port assigned to your project:

.. code-block:: bash

   openstack port list

Example output:

.. code-block:: text

   | dbbf0c39-4d5e-4873-aca2-1eb64aadf8ae | USRP-101 subnet_id='c137a67b-de3b-48ec-9842-7ba0bf32403e' | DOWN | fa:16:3e:e9:cc:5d | ip_address='192.168.101.8' |

Copy the **Port ID** (first column) for the USRP port you want to attach.

Attaching the USRP Port to the Instance
----------------------------------------

Attach the port to your radio instance:

.. code-block:: bash

   openstack server add port <Instance-ID> <PORT-ID>

Replace ``<Instance-ID>`` with your radio instance ID and ``<PORT-ID>`` with the USRP port ID.

Verification
------------

Check the OpenStack dashboard to verify that the port has been attached to the instance. Navigate to **Project → Compute → Instances**, select your radio instance, and view the **Interfaces** tab to confirm the USRP port is listed.


