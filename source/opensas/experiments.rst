Tutorials
=========

Sample Experiment: Priority Protection of PAL Users through OpenSAS
-------------------------------------------------------------------

Objective
^^^^^^^^^

This tutorial outlines the steps to demonstrate the integration of the OpenSAS Server and CBSD for GAA and PAL operations using srsRAN and Open5GS. We demonstrate priority protection for higher-tier user (PAL) in presence of lower-tier users (GAA) in the CBRS ecosystem through OpenSAS.  

Experimental Setup
^^^^^^^^^^^^^^^^^^

The setup uses one VM/PC running OpenSAS and its dashboard, and two other VMs/PCs running the CBSD client with an srsRAN gNB in ZMQ mode. The GAA CBSD is run first, followed by the PAL CBSD requesting same frequency in a overlapping coverage area. The OpenSAS server will grant the PAL CBSD access to the spectrum, ensuring priority protection for PAL users and GAA users wait for the spectrum to be available. Once spectrum is vacated by the PAL user, the GAA user is automatically granted access to the spectrum.

.. figure:: ../_static/opensas/image24.png
   :align: center
   :alt: Experimental Setup
   :scale: 50%

   **Figure :** Experimental Setup.


Setting Up the Experiment
~~~~~~~~~~~~~~~~~~~~~~~~~


.. - :ref:`OpenSAS Core <start-opensas-server>`
.. - :ref:`OpenSAS Dashboard <launch-opensas-dashboard>`
.. - :ref:`CBSD with srs gNB <running-cbsd-client>`


**Prerequisites**

   - OpenSAS server and dashboard are installed and configured properly. The installation and configuration steps can be found at `server installation and configuration <https://cci-opensas.readthedocs.io/en/latest/installation_configuration.html#building-opensas-from-source>`_ and `dashboard installation and configuration <https://cci-opensas.readthedocs.io/en/latest/installation_configuration.html#building-opensas-dashboard-from-source>`_.
   - Install and configure PAL and GAA CBSDs with srsRAN gNB and Open5GS core. The installation and configuration steps can be found at `CBSD client for OpenSAS <https://cci-opensas.readthedocs.io/en/latest/installation_configuration.html#cbsd-client-for-opensas>`_.

**Running the Experiment**


1. **Start the OpenSAS Server**
   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^

   From the VM hosting OpenSAS, start the OpenSAS Server:

   .. code-block:: bash

      cd OpenSAS/Core
      python server.py

   Ensure that the OpenSAS server is running.

   .. figure:: ../_static/opensas/image4.png
      :align: center
      :alt: OpenSAS Server Running
      :scale: 50%

      **Figure:** OpenSAS Server Running.

.. _launch-opensas-dashboard:

2. **Launching the OpenSAS Dashboard**
   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

   Once OpenSAS server is up and running, launch the OpenSAS dashboard:

   .. code-block:: bash

      cd OpenSAS-dashboard/
      npm run dev

   Verify that it is operational and accessible at `http://localhost:9528/` or `http://<dashboard-machine-ip>:9528/`.

   .. figure:: ../_static/opensas/image7.png
      :align: center
      :alt: OpenSAS Dashboard Running
      :scale: 80%

      **Figure:** OpenSAS Dashboard Running.
      

   .. figure:: ../_static/opensas/image8.png
      :align: center
      :alt: OpenSAS Dashboard Login
      :scale: 40%

      **Figure:** OpenSAS Dashboard Login Page.

   The OpenSAS dashboard presents CBSD and spectrum views of the registered CBSDs, also a map showing the location and transmission radius of the CBSDs.

   .. figure:: ../_static/opensas/image9.png
      :align: center
      :alt: CBSD List
      :scale: 40%

      **Figure:** OpenSAS Dashboard views.

.. _running-cbsd-client:

3. On the VM hosting GAA CBSD client (with Open5GS core, srsRAN gNB, and RF frontend connected), run the GAA first: 

   a. **Open a New TMUX Session**

      This allows you to manage multiple terminal sessions.

      .. code-block:: bash

         cd CBSD/
         tmux

   b. **Start the GAA CBSD Client**

      In the `tmux` terminal, run the following command:

      .. code-block:: bash

         python3 run.py --lat 38.88086127246137 --lon -77.11558699967016 --fcc GAA-Arlington --low 3610e6 --high 3620e6 -eirp 20 -react 0

      - After running this, you should see a registration request from the CBSD client sent to OpenSAS, a spectrum inquiry, and then OpenSAS grants.
      - The corresponding logs will appear in the OpenSAS dashboard and OpenSAS console.
      - After a successful grant, another terminal will open, starting the srsRAN 5G gNB.
      - You can now view this CBSD information in the CBSD list, Spectrum list, and map in the OpenSAS dashboard.
   
   c. **Output for GAA Operation**

      - **OpenSAS server logs indicating the CBSD registration:**

      .. figure:: ../_static/opensas/image15.png
         :align: center
         :alt: OpenSAS Log
         :scale: 50%

         **Figure 5:** OpenSAS log indicating the CBSD registration.

      - **CBSD console logs showing registration and spectrum inquiries:**

      .. figure:: ../_static/opensas/image16.png
         :align: center
         :alt: CBSD Console Logs
         :scale: 50%

         **Figure 6:** CBSD console logs indicating registration and spectrum inquiries.

      - **OpenSAS Dashboard displaying GAA CBSD location on the map:**

      .. figure:: ../_static/opensas/image17.png
         :align: center
         :alt: OpenSAS Dashboard Map
         :scale: 50%

         **Figure 7:** OpenSAS Dashboard displaying GAA CBSD location on the map.

      - **Authorized band for the CBSD after grant response:**

      .. figure:: ../_static/opensas/image18.png
         :align: center
         :alt: Authorized Band
         :scale: 50%

         **Figure 8:** Authorized band for the CBSD after grant response.

      - **Registered CBSD and its corresponding ID:**

      .. figure:: ../_static/opensas/image19.png
         :align: center
         :alt: Registered CBSD
         :scale: 50%

         **Figure 9:** Registered CBSD and its corresponding ID.



4. **Running the PAL CBSD Client**
   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

   Now, start the PAL CBSD client on a different VM:

   a. **Open a New TMUX Session**

      This allows you to manage multiple terminal sessions.

      .. code-block:: bash

         cd CBSD/
         tmux

   b. **Start the PAL CBSD Client**

      In the `tmux` terminal, run the following command:

      .. code-block:: bash

         python3 run.py --lat 38.8818743855486 --lon -77.11132435717902 --fcc CCI-CBRS-PAL --low 3610e6 --high 3620e6 -eirp 20 -react 0

      - After running this, you should see a similar registration request from the CBSD client sent to OpenSAS, a spectrum inquiry, and then OpenSAS grants.
      - The corresponding logs will appear in the OpenSAS dashboard and OpenSAS console.
      - After a successful grant, another terminal will open, starting the srsRAN 5G gNB.
      - You can now view this CBSD information in the CBSD list, Spectrum list, and map in the OpenSAS dashboard.
   
   c. **Output for PAL Operation**

      Since the PAL user has priority over the GAA user, the PAL user will be granted access to the spectrum first. The GAA user will have to stop its operation and will be granted access only after the PAL user relinquishes the spectrum. In the meantime, the GAA user is in registered state and keep sending spectrum Inquiry to the SAS.

      - **OpenSAS Dashboard displaying CBSD location on the map:**

      Since PAL user is granted same frequency as GAA user, the GAA user stops transmission and corresponding coverage area is removed from the map and the PAL user's coverage area is shown.

      .. figure:: ../_static/opensas/image23.png
         :align: center
         :alt: OpenSAS Dashboard Map
         :scale: 50%

         **Figure 10:** OpenSAS Dashboard displaying PAL and GAA CBSD location on the map.

      - **Registered PAL CBSD user on the dashboard:**

      .. figure:: ../_static/opensas/image21.png
         :align: center
         :alt: Registered PAL CBSD
         :scale: 50%

         **Figure 11:** Registered PAL CBSD user on the dashboard.

   Once PAL operation is terminated, the GAA user will be granted access to the spectrum and the corresponding coverage area will be shown on the map.

   .. figure:: ../_static/opensas/image17.png
         :align: center
         :alt: OpenSAS Dashboard Map
         :scale: 50%

         **Figure 12:** OpenSAS Dashboard displaying GAA CBSD location on the map.


