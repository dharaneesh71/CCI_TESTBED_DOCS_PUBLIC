===================
GitLab Access Guide
===================

This guide provides detailed information about accessing and using the CCI xG Testbed GitLab project management and version control system.

Video Tutorial
--------------

.. raw:: html

   <div class="demo-videos">
     <h3>GitLab Complete Walkthrough</h3>
     <video controls preload="metadata" playsinline crossorigin="anonymous" width="800">
       <source src="../_static/Gitlab_new.mp4" type="video/mp4">
       Your browser does not support the video tag.
     </video>
     <p><a href="../_static/Gitlab_new.mp4">Download video</a></p>
   </div>

What is GitLab?
---------------

GitLab is a flexible platform used by the CCI xG Testbed for version control, CI/CD, and project management. It allows you to:

* Track experiment progress and manage codebase versions
* Report, track, and resolve technical issues
* Collaborate with team members and testbed administrators
* Access documentation via repository READMEs and Wikis
* Manage project timelines, merge requests, and milestones

Accessing GitLab via Authentik SSO
----------------------------------

Access to GitLab is securely managed through the CCI xG Testbed's Authentik Single Sign-On (SSO) portal.

To access GitLab:

1. Open your web browser.
2. Navigate to the Authentik portal: `Link <https://authentik.ccixgtestbed.org>`_
3. Authenticate using one of the following method:

   * **CILogon (Institutional Access)**: Click the **Continue with CILogon** button at the bottom of the page. On the following screen, select your academic institution (e.g., Virginia Tech) from the Identity Provider dropdown, click **Log On**, and complete your standard university sign-in process.
4. After successfully authenticating, you will be directed to the **My applications** dashboard.
5. Locate and click on the **Gitlab** tile. You will be automatically authenticated and redirected to your GitLab workspace at https://gitlab.ccixgtestbed.org.

Getting Started with GitLab
---------------------------

Dashboard Overview
^^^^^^^^^^^^^^^^^^

After logging in, you'll see your "Your work" dashboard which provides:

* **Home**: A personalized view highlighting your assigned issues, merge requests, and recent activity
* **Projects**: A list of all repositories you have access to
* **Groups**: The organizational folders containing related subgroups and projects
* **Issues**: A consolidated list of all issues assigned to you or authored by you
* **Merge requests**: Active code integration requests

Navigating Groups and Projects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

GitLab organizes repositories under **Groups**. In the current testbed setup, projects live directly under the main group (GitLab also supports subgroups).

1. Click on the **Groups** tab in the left sidebar to see all available groups.
2. Select the main group (e.g., ``CCI xG Testbed External``).
3. On the **Subgroups and projects** tab, select the project repository you need (e.g., ``OpenStack``, ``Clear-ML``, ``Near-RT-RIC``, ``Non-RT-RIC``, or ``SDR Booking``).
4. Inside a project, use the left sidebar to access **Code**, **Build**, **Deploy**, and more. **Issues** and **Merge requests** are pinned at the top of the project sidebar.

Working with Issues
-------------------

Searching for Existing Issues
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before creating a new issue, it's best practice to search for existing tickets that might address your concern:

1. Navigate to the **Issues** section of your specific project.
2. Use the top search bar to enter keywords related to your issue.
3. Apply filters to narrow down results:

   * Status (Open, Closed)
   * Assignee
   * Author
   * Labels

Creating New Issues
^^^^^^^^^^^^^^^^^^^

If you don't find an existing issue that matches your needs:

1. Navigate to your specific project (e.g., ``CCI xG Testbed External`` > ``OpenStack``).
2. Click **Issues** in the left sidebar, then click the blue **New item** button at the top right and choose **Issue** (the **New issue** form opens).
3. Fill in the required fields:

   * **Type**: Select the ticket type (Issue, Incident, or Task).
   * **Title**: A brief, descriptive title (e.g., "testing").
   * **Description**: Detailed information about the issue. You can use Markdown to format text or attach files.
   * **Assignee**: Search for and select the team member who should work on this issue.
   * **Labels / Milestone**: Apply relevant tags or target completion dates if applicable.
   * **Dates**: Optionally set a **Start** and **Due** date for the issue.

4. Click the blue **Create issue** button at the bottom to submit.

.. note:: Once you have submitted the ticket, your issue will be successfully created and added to the project. The issue will be addressed soon by the respective team member or admin.

Tracking and Updating Issues
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once an issue is created:

1. You'll receive email notifications about updates.
2. You can scroll to the bottom of the issue to add comments or tag team members using ``@username``.
3. Use the right-hand sidebar to update metadata:

   * Modify the **Assignee**
   * Add or remove **Labels** to indicate progress (e.g., *To Do*, *Doing*, *In Review*)
   * Set or adjust **Start** / **Due** dates
   * Link **Child items** or **Linked items**, and log effort under **Time tracking**
   * Close the issue once it is resolved using the **Close issue** button


.. note:: For additional assistance with GitLab, please contact the CCI xG Testbed support team through GitLab issues or via email at the address provided in your welcome message.