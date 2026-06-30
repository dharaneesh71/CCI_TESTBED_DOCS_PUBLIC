=================================
 Getting Started with CCI xG Testbed
=================================

CCI xG Testbed provides a platform consisting of wireless and computing
resources for researchers to carry out advanced wireless
research. Users can access the platform by completing the User Sign-up Form with their personal and project details.
Once approved, users can utilize the allocated resources for executing their wireless experiments.

Access and Account Creation
==========================

User Sign-up Process
-----------------

To gain access to the CCI xG Testbed, you need to complete the User Sign-up Form:

`CCI xG Testbed User Sign-up Form <https://docs.google.com/forms/d/e/1FAIpQLSdabgove9qaSd6HdAFQQRSCwPfLcizga8na9gwxjZaWukF9qQ/viewform>`_

.. note:: Account requests typically take 2 business days for approval.


Once your submission is reviewed and approved, the Administrative team will send you an email 
containing your access instructions. All CCI xG Testbed resources are centralized through our 
Single Sign-On (SSO) portal.

How to Access Your Dashboard
----------------------------

1. **Open the Portal**

   You can access the CCI xG Testbed portal in either of the following ways:

   * Go to the CCI xG Testbed website and select **User → Login** from the top navigation menu.
   * Or directly open the portal at `CCI xG Testbed Portal <https://authentik.ccixgtestbed.org>`_.

   .. figure:: /_static/1.png
      :alt: CCI xG Testbed website showing the User Login menu
      :width: 60%
      :align: center

      Accessing the portal from the CCI xG Testbed website.

2. **Sign In**

   On the login page, select **Continue with CILogon**. CILogon allows you to securely authenticate using your university or institutional identity provider.

   .. figure:: /_static/2.png
      :alt: CCI xG Testbed login page with CILogon option
      :width: 60%
      :align: center

      Login page with the CILogon authentication option.

3. **Select Your Institution**

   On the CILogon page, choose your identity provider, such as **Virginia Tech**, and click **Log On**.

   .. figure:: /_static/3.png
      :alt: CILogon identity provider selection page
      :width: 60%
      :align: center

      Selecting an institutional identity provider in CILogon.

4. **Approve Access**

   CILogon may ask you to approve the release of basic account information, such as your name, email address, username, and institutional affiliation. Review the information and continue only if you approve.

5. **Accept the Usage Policy**

   On your first login, you may be asked to read and accept the CCI xG Testbed Acceptable Use Policy. Check the agreement box and click **Continue**.

6. **Open Your Dashboard**

   After successful login, you will be redirected to the **My Applications** dashboard. This dashboard displays the resource tiles assigned to your account. Click any available tile to launch the corresponding environment, tool, or project platform.

.. note::
   For detailed information on navigating the portal and using assigned resources, refer to the
   :doc:`Portal Access Guide <portal_access>`.

.. attention::
   When completing the initial request form, be specific about your project requirements in the
   **Purpose of CCI xG Testbed Usage** field. This helps ensure that the appropriate resource
   tiles are allocated to your dashboard.


CCI Dashboard and Experiment Environment
=======================================

After receiving your access credentials, you'll begin your journey with the CCI Dashboard, which serves as the central hub for accessing various components of the CCI xG Testbed and launching your experiments.

.. figure:: ../user-dashboard/user-flow.jpg
   :alt: User-Flow 
   :align: center
   
   
   Figure: Experimental User - Workflow Diagram

Authentication and Navigation
---------------------------

1. **Initial Access**:

   - Navigate to the CCI Dashboard login page
   - Enter your provided username and password
   - The system will validate your credentials

     * If invalid, an error message will be displayed, prompting you to re-enter your credentials
     * If valid, you'll be redirected to the main navigation page

2. **Main Navigation Options**:
   After successful login, you'll be presented with a clean, intuitive interface offering four main options:

   * **Non-RT Dashboard**: Access to Non-Real-Time RAN Intelligent Controller management
   * **Near-RT Dashboard**: Access to Near-Real-Time RAN Intelligent Controller management
   * **Clear-ML**: Access to the Clear-ML platform for ML model training and management
   * **OpenStack Login**: Button to authenticate and access the OpenStack environment

Dashboard Components
------------------

**Non-RT Dashboard**

If you select the Non-RT Dashboard option, you'll gain access to:

* **Non-RT RIC Management**: Monitor and configure the Non-RT RIC platform
* **rApps Management**: Deploy, configure, and monitor rApps
* **Policy Management**: Create, edit, and distribute policies to Near-RT RICs

**Near-RT Dashboard**

If you select the Near-RT Dashboard option, you'll gain access to:

* **Near-RT RIC Management**: Monitor and configure the Near-RT RIC platform
* **xApps Management**: Deploy, configure, and monitor xApps
* **E2 Node Management**: Monitor and manage E2 Nodes (CU/DU) connected to the Near-RT RIC

Accessing the Experiment Environment via OpenStack
-----------------------------------------------

The primary way to access your experiment environment is through the OpenStack dashboard. This is where you'll create and manage the virtual machines and resources needed for your experiments.

**Accessing OpenStack**:

1. From the CCI Dashboard main navigation page, click the **OpenStack Login** button
2. You'll be redirected to the OpenStack authentication page
3. Enter your OpenStack credentials (provided in your welcome email)
4. After successful authentication, you'll access the OpenStack Dashboard
5. From there, you can create instances, configure networks, manage volumes, and launch your experiment environment

**Setting Up Your Experiment Environment**:

Once logged into the OpenStack Dashboard, you can:

1. Create virtual machines with your required specifications
2. Configure networking for your experiment
3. Allocate storage resources
4. Deploy and run your experiment software
5. Document your experiment setup, track issues, and manage your project timeline using gitlab

For detailed instructions on creating and managing OpenStack instances, please refer to our 
:doc:`OpenStack Instance Launch Guide <openstack>`.

.. note:: For the best experience with the CCI xG Testbed portal and OpenStack dashboard, we recommend 
          using modern web browsers such as Google Chrome, Mozilla Firefox, or Microsoft Edge.
