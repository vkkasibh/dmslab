---


---

<h1 id="aws-database-migration-service-lab">AWS Database Migration Service Lab</h1>
<h2 id="version-2.3">Version 2.3</h2>
<p>© 2019 Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited.</p>
<p>Errors or corrections? Email us at <a href="mailto:aws-course-feedback@amazon.com">aws-course-feedback@amazon.com</a>.</p>
<p>Other questions? Contact us at <a href="https://aws.amazon.com/contact-us/aws-training/">https://aws.amazon.com/contact-us/aws-training/</a></p>
<h2 id="overview">Overview</h2>
<p>This lab exercise shows you how to migrate an Oracle database to an Amazon Aurora PostgreSQL database. The database migration is performed in two steps—convert the schema, and migrate the data. You will learn how to:</p>
<ul>
<li>Use the AWS Schema Conversion Tool (SCT) to convert a database schema</li>
<li>Use the AWS Database Migration Service (DMS) to migrate a database and replicate its data</li>
</ul>
<p>Optionally, you will also learn how to use continuous data replication to automatically synchronize changes in the source database to the target database.</p>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/DMS_Lab_Overview_Workflow.png" alt="Lab workflow"></p>
<h2 id="system-requirements">System Requirements</h2>
<p>To complete the lab exercises, your computer setup must include the following:</p>
<ul>
<li>Network connection to the AWS console (<a href="https://console.aws.amazon.com">https://console.aws.amazon.com</a>) and an appropriate browser to use the console</li>
<li>Remote desktop connection utility to view a remote Windows server desktop</li>
<li>Secure shell program, such as PuTTY, to log in to a Linux instance by using ssh (optional)</li>
</ul>
<h2 id="lab-environment">Lab Environment</h2>
<p>The diagram illustrates the lab environment deployed in the Qwiklabs AWS account. The lab runs in the <strong>Virginia</strong> AWS Region <strong>(us-east-1)</strong>.</p>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/Lab-diagram.png" alt="Lab diagram"></p>
<h2 id="starting-the-lab-environment">Starting the Lab Environment</h2>
<p>In this exercise, you will start the lab environment and collect connection details.</p>
<ol>
<li>
<p>On the Qwiklabs console, click <strong>Start Lab</strong> to start your lab. The Qwiklabs console runs an AWS CloudFormation stack to set up the following components in AWS:</p>
<ul>
<li><strong>Server-SCT</strong>—EC2-based server instance, which runs the AWS Schema Conversion Tool, Oracle SQL Developer, and PostgreSQL Administrator interface</li>
<li><strong>Oracle</strong>—EC2-based Oracle instance, which serves as the source database</li>
<li><strong>AWS RDS Aurora PostgreSQL</strong>—Target database instance</li>
</ul>
<blockquote>
<p><strong>Note</strong> Lab setup can take up to 20 minutes. Track the startup progress by watching the bar at the top of the Qwiklabs console. You can perform the next steps in the lab when you see <em>Lab Running</em> in the progress bar.</p>
</blockquote>
</li>
<li>
<p>From the Qwiklabs console, download the PEM and PPK private keys to decrypt the Windows Administrator’s password for the Server-SCT instance. You will use the same key pair to connect to the Oracle EC2 instance for optionally testing source data change capture for data replication.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/download_pem.png" alt="Download keys"></p>
<ol start="3">
<li>When the lab setup completes, copy the connection details so they can be used later.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/Qwiklabs_conn_details.png" alt="Setup complete"></p>
<h2 id="task-1-converting-the-schema">Task 1: Converting the Schema</h2>
<p>In this exercise, you will migrate the Oracle database’s schema to Aurora PostgreSQL. The exercise includes the following procedures:</p>
<ul>
<li>Connect to the EC2 instance that runs the AWS Schema Conversion Tool (SCT)</li>
<li>Use SCT to create the schema in the target database</li>
<li>On the target database, drop foreign keys and disable triggers</li>
<li>Use DMS to create a task to replicate the source database to the target database in full-load mode</li>
<li>When the full-load replication completes, restore the foreign keys on the target database</li>
</ul>
<h3 id="log-on-to-the-ec2-server-instance">Log on to the EC2 server instance</h3>
<p>In this exercise, you will open the AWS EC2 console, obtain the Server-SCT password, and if needed, download and install the Remote Desktop File.</p>
<ol>
<li>
<p>Click <strong>Open Console</strong> in the left sidebar of the Qwiklabs console. Use the connection details at the left to complete the console login page, and click <strong>Sign In</strong>.</p>
</li>
<li>
<p>Type <strong>EC2</strong> into the search bar, and select <strong>EC2</strong>. You should see the EC2 dashboard.</p>
</li>
<li>
<p>Click <strong>Instances</strong> to see the servers running in your AWS account.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/DMS-Hetrogeneous-Diagram3.png" alt="enter image description here"></p>
<ol start="4">
<li>Select the <strong>Server-SCT</strong> instance, and click <strong>Connect</strong>.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/DMS_EC2_Console.png" alt="enter image description here"></p>
<ol start="5">
<li>
<p>In the <strong>Connect To Your Instance</strong> dialog box, click <strong>Get Password</strong>.</p>
</li>
<li>
<p>In the <strong>Connect To Your Instance &gt; Get Password</strong> dialog box, click <strong>Browse</strong>. Choose the PEM key pair file that you downloaded from the Qwiklabs Console to upload. When the key pair is uploaded, click <strong>Decrypt Password</strong> to see the EC2 console-generated administrator password.</p>
</li>
<li>
<p>In the <strong>Connect To Your Instance</strong> dialog box, copy the administrator password, and click <strong>Download Remote Desktop File</strong>. This retrieves the RDP file needed to access the Windows desktop of the Server-SCT EC2 instance.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/DMS_Get_Password_RDPFile.png" alt="enter image description here"></p>
<ol start="8">
<li>To connect to the EC2 instance, use <strong>Remote Desktop Connection</strong> on your computer and the RDP file downloaded in the previous step.</li>
</ol>
<h3 id="create-a-database-migration-project">Create a database migration project</h3>
<p>In the upcoming steps, you will use SCT to create a database migration project.</p>
<ol>
<li>Launch SCT from the application menu or Windows taskbar.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/SCT_Windows_Taskbar.png" alt="enter image description here"></p>
<pre><code>&gt; **Note** Do not upgrade the Schema Conversion Tool. The lab instructions are specific to the version installed.
</code></pre>
<ol start="2">
<li>To create a new project, click <strong>File &gt; New Project…</strong>.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/new_project1.png" alt="enter image description here"></p>
<ol start="3">
<li>
<p>In the <strong>New Project</strong> dialog box, complete the following fields, and click <strong>OK</strong>. This creates a database migration project.</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Project name</td>
<td><strong>AWS Schema Conversion Tool Project1</strong></td>
</tr>
<tr>
<td>Location</td>
<td><strong>C:\Users\Administrator\AWS Schema Conversion Tool\Projects</strong></td>
</tr>
<tr>
<td>Type</td>
<td><strong>Transactional Database (OLTP)</strong></td>
</tr>
<tr>
<td>Source Database Engine</td>
<td><strong>Oracle</strong></td>
</tr>
<tr>
<td>Target Database Engine</td>
<td><strong>Amazon Aurora (PostgreSQL compatible)</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/project_main.png" alt="enter image description here"></p>
<ol start="4">
<li>In the SCT menu bar, click <strong>Connect to Oracle</strong>.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/oracle_connect1.png" alt="enter image description here"></p>
<ol start="5">
<li>
<p>In the <strong>Connect to Oracle</strong> dialog box, complete the following values, and click <strong>Test Connection</strong>.</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Type</td>
<td><strong>SID</strong></td>
</tr>
<tr>
<td>Server Name</td>
<td>Use the value of <em>OracleEC2InstancePrivateIP</em> from the Qwiklabs Connection Details.</td>
</tr>
<tr>
<td>Server Port</td>
<td><strong>1521</strong></td>
</tr>
<tr>
<td>Oracle SID</td>
<td><strong>xe</strong></td>
</tr>
<tr>
<td>User Name</td>
<td><strong>sct_user</strong></td>
</tr>
<tr>
<td>Password</td>
<td><strong>reinvent_Dms_Sct_17</strong></td>
</tr>
<tr>
<td>Use SSL</td>
<td><strong>Cleared</strong></td>
</tr>
<tr>
<td>Store Password</td>
<td><strong>Cleared</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/oracle_connect2.png" alt="enter image description here"></p>
<ol start="6">
<li>
<p>Test the connection. When the test is successful, click <strong>OK</strong>. This saves the connection configuration.</p>
</li>
<li>
<p>In the SCT menu bar, click <strong>Connect to Amazon Aurora (PostgreSQL compatible)</strong>.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/pg_connect.png" alt="enter image description here"></p>
<ol start="8">
<li>
<p>In the <strong>Connect to Amazon Aurora (PostgreSQL compatible)</strong> dialog box, complete the following fields, and click <strong>Test Connection</strong>.</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Server name</td>
<td>Use the value of <em>AuroraPostgreSQLDBEndpoint</em> from the Qwiklabs Connection Details.</td>
</tr>
<tr>
<td>Server port</td>
<td><strong>5432</strong></td>
</tr>
<tr>
<td>Database</td>
<td><strong>AuroraPostgreSQLDB</strong></td>
</tr>
<tr>
<td>User name</td>
<td><strong>sct_user</strong></td>
</tr>
<tr>
<td>Password</td>
<td><strong>reinvent_Dms_Sct_17</strong></td>
</tr>
<tr>
<td>Use SSL</td>
<td><strong>Cleared</strong></td>
</tr>
<tr>
<td>Store Password</td>
<td><strong>Cleared</strong></td>
</tr>
</tbody>
</table></li>
<li>
<p>Test the connection. When the test is successful, click <strong>OK</strong>. This saves the connection configuration.</p>
</li>
<li>
<p>Expand the <strong>HR</strong> schema, as shown in the following figure.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/hr_schema.png" alt="enter image description here"></p>
<h3 id="convert-the-schema">Convert the schema</h3>
<p>To convert the Oracle database schema to an Aurora PostgreSQL database, complete the following steps.</p>
<ol>
<li>In the <strong>Oracle</strong> pane, right-click the <strong>HR</strong> schema, and click <strong>Convert Schema</strong>.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/convert_schema2.png" alt="enter image description here"></p>
<ol start="2">
<li>
<p>In the <strong>Convert</strong> dialog box, for the <strong>Objects may already exist in the target database, Replace?</strong> question, click <strong>Yes</strong>.</p>
</li>
<li>
<p>Review and analyze the schema conversion result. Items marked with a red exclamation point cannot be directly translated from the source to the target. In this case, the <em>functions</em>, <em>views</em>, and <em>procedures</em> can be directly converted.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/sct_convert_result.png" alt="enter image description here"></p>
<ol start="4">
<li>To view more information about the issues, select <strong>View &gt; Assessment Report View</strong>. This displays a report containing conversion information. Review the assessment report, and examine any suggested actions for addressing schema conversion issues.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/post_conversion2.png" alt="enter image description here"></p>
<ol start="5">
<li>To see details about issues SCT finds during a conversion, click <strong>Action Items</strong>. In the <strong>Action Items</strong> tab, you can either resolve the issues in the source database, or skip them and fix issues on the target database.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/action_items.png" alt="enter image description here"></p>
<ol start="6">
<li>
<p>Check each issue listed, and compare the contents on the source <strong>Oracle schema</strong> panel and the target <strong>Amazon Aurora (PostgreSQL compatible) schema</strong> panel. Are the issues resolved? How?</p>
</li>
<li>
<p>Notice SCT proposed resolving the issues by generating equivalent PostgreSQL DDL to convert the objects. Since the resolutions are satisfactory, no issue-related action is required.</p>
</li>
<li>
<p>In the <strong>Amazon Aurora (PostgreSQL compatible)</strong> pane, right-click <strong>hr</strong>, and click <strong>Save as SQL</strong>. This exports the SQL DDLs that SCT generated for the HR target database.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/ddl_save.png" alt="enter image description here"></p>
<pre><code>&gt; **Note** You can safely ignore the message **Windows can't open this type of file (.sql)**. You can use Windows WordPad to open the SQL file.   

&gt; **Note** You can use some of the exported SQL commands in Task 2 of this lab to drop and restore the foreign keys and triggers at the target database. Alternatively, you can use the SQL commands in the next section to drop and restore the database constraints.
</code></pre>
<h3 id="apply-the-converted-schema">Apply the converted schema</h3>
<p>To migrate the converted schema to the target Aurora PostgreSQL-compatible database, complete the steps in this section.</p>
<ol>
<li>In the <strong>Amazon Aurora (PostgreSQL compatible)</strong> pane, right-click <strong>hr</strong>, and click <strong>Apply to database</strong>.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/apply_to_database.png" alt="enter image description here"></p>
<ol start="2">
<li>In the <strong>Amazon Aurora (PostgreSQL compatible) database</strong> dialog box, for the <strong>You chose to apply the schema definition for hr. Are you sure?</strong> question, click <strong>Yes</strong>. This confirms the action and applies the schema.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/schema_confirmation.png" alt="enter image description here"></p>
<ol start="3">
<li>Point to the <strong>hr</strong> schema or expand the tree view to see the tables. You should see <strong>Object was successfully applied to database</strong>.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/schema_verification.png" alt="enter image description here"></p>
<pre><code>&gt; **Note** If you don't see the tables, right-click the **hr** schema, and click **Refresh from Database**.
</code></pre>
<ol start="4">
<li>Leave the Remote Desktop Connection open. You will use it in Task 2.</li>
</ol>
<h3 id="summary">Summary</h3>
<p>In Task 1, you learned how to successfully convert and migrate a schema from an Oracle database into an Amazon Aurora PostgreSQL database. The SCT largely automates the schema conversion. You can follow the same steps to migrate SQL Server and Oracle workloads to other Amazon Relational Database Service (Amazon RDS) engines, including Aurora MySQL and MySQL.</p>
<h2 id="task-2-migrating-data">Task 2: Migrating Data</h2>
<p>The AWS Database Migration Service (AWS DMS) helps you easily and securely migrate databases to AWS. The source database remains fully operational during the migration, minimizing downtime to applications that rely on the database. AWS DMS can migrate your data to and from most widely used commercial and open-source databases. You can also use DMS for continuous data replication with high availability.</p>
<p>Task 2 shows you how to migrate data from an Oracle database hosted on an EC2 instance to Amazon Aurora PostgreSQL. In this part of the lab, you will:</p>
<ul>
<li>Create a DMS replication instance</li>
<li>Create DMS source and target endpoints</li>
<li>Run a DMS replication task for full-load replication, including:
<ul>
<li>Checking the source database content for post-replication validation</li>
<li>Dropping foreign keys and disabling triggers on the target database</li>
<li>Setting up and running a full-load replication task</li>
<li>Validating the data replication result</li>
<li>Restoring foreign keys</li>
</ul>
</li>
</ul>
<h3 id="create-a-dms-replication-instance">Create a DMS replication instance</h3>
<p>In this exercise, you will open the DMS console and create a replication instance. The replication instance initiates a connection between  source and target databases, transfers the data, and caches any changes that occur on the source database during the initial data load. To open the DMS console and create a replication instance, complete the following steps.</p>
<ol>
<li>Switch to the browser window that contains the AWS console, and open the DMS console. You can find the DMS console by typing <strong>DMS</strong> in the search bar at the top of the page.</li>
</ol>
<p><img src="https://apn-training.s3.amazonaws.com/Screen-shot-1.png" alt="enter image description here"><br>
2. In the DMS console, click <strong>Create replication instance</strong>.</p>
<p>![enter image description here](<a href="https://s3.amazonaws.com/migrationtechresources2.3/Screen+Shot+2020-02-05+at+12.04.20+PM.png">https://s3.amazonaws.com/migrationtechresources2.3/Screen+Shot+2020-02-05+at+12.04.20+PM.png</a></p>
<ol start="3">
<li>
<p>In the <strong>Create replication instance</strong> window, configure the replication instance with the following values, and then click <strong>Create replication instance</strong>.</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Name</td>
<td><strong>replication-instance</strong></td>
</tr>
<tr>
<td>Description</td>
<td><strong>Oracle to Aurora DMS replication instance</strong></td>
</tr>
<tr>
<td>Instance class</td>
<td><strong>dms.t2.medium</strong></td>
</tr>
<tr>
<td>Engine version</td>
<td><strong>Pick the latest version</strong></td>
</tr>
<tr>
<td>VPC</td>
<td>Select the VPC labeled with <em>DMS Lab v2</em></td>
</tr>
<tr>
<td>Multi-AZ</td>
<td><strong>No</strong></td>
</tr>
<tr>
<td>Publicly accessible</td>
<td><strong>Cleared</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3.amazonaws.com/migrationtechresources2.3/rep-instance.png" alt="enter image description here"><br>
4. In a few minutes, the replication instance is created and available to use. Notice the <strong>Status</strong> field changes from <strong>creating</strong> to <strong>available</strong>.</p>
<h3 id="create-source-and-target-endpoints">Create source and target endpoints</h3>
<p>In the next steps, you will create source and target endpoints.</p>
<ol>
<li>
<p>After the replication instance is available, go to <strong>Endpoints</strong>, and click <strong>Create endpoint</strong>.</p>
</li>
<li>
<p>Create the source endpoint by completing the following fields:</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Endpoint type</td>
<td><strong>Source</strong></td>
</tr>
<tr>
<td>Select RDS DB Instance</td>
<td><strong>Cleared</strong></td>
</tr>
<tr>
<td>Endpoint identifier</td>
<td><strong>Oracle-source</strong></td>
</tr>
<tr>
<td>Source engine</td>
<td><strong>oracle</strong></td>
</tr>
<tr>
<td>Server name</td>
<td>Use the value of <em>OracleEC2InstancePrivateIP</em> from the Qwiklabs Connection Details.</td>
</tr>
<tr>
<td>Port</td>
<td><strong>1521</strong></td>
</tr>
<tr>
<td>SSL mode</td>
<td><strong>none</strong></td>
</tr>
<tr>
<td>User name</td>
<td><strong>sct_user</strong></td>
</tr>
<tr>
<td>Password</td>
<td><strong>reinvent_Dms_Sct_17</strong></td>
</tr>
<tr>
<td>SID/Service Name</td>
<td><strong>xe</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3.amazonaws.com/migrationtechresources2.3/Screen+Shot+2020-02-05+at+12.23.08+PM.png" alt="enter image description here"></p>
<ol start="3">
<li>
<p>Expand the <strong>Test endpoint connection (optional)</strong> section, and choose the VPC that has a label ending in <em>DMS Lab v2</em>.</p>
</li>
<li>
<p>Click <strong>Run test</strong>, and verify that the endpoint is valid before you use it in the replication task.</p>
</li>
<li>
<p>When the connection tests successfully, click <strong>Create endpoint</strong>. This saves the endpoint.</p>
</li>
<li>
<p>Create the target endpoint by repeating the preceding steps to create an endpoint with the following configuration:</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Endpoint type</td>
<td><strong>Target</strong></td>
</tr>
<tr>
<td>Select RDS DB Instance</td>
<td><strong>Cleared</strong></td>
</tr>
<tr>
<td>Endpoint identifier</td>
<td><strong>Aurora-PostgresSQL-Target</strong></td>
</tr>
<tr>
<td>Target engine</td>
<td><strong>aurora-postgresql</strong></td>
</tr>
<tr>
<td>Server name</td>
<td>Use the value of <em>AuroraPostgreSQLDBEndpoint</em> from the Qwiklabs Connection Details.</td>
</tr>
<tr>
<td></td>
<td><strong>Please ignore the error about the server name value being too long</strong></td>
</tr>
<tr>
<td>Port</td>
<td><strong>5432</strong></td>
</tr>
<tr>
<td>SSL mode</td>
<td><strong>none</strong></td>
</tr>
<tr>
<td>User name</td>
<td><strong>sct_user</strong></td>
</tr>
<tr>
<td>Password</td>
<td><strong>reinvent_Dms_Sct_17</strong></td>
</tr>
<tr>
<td>Database name</td>
<td><strong>AuroraPostgreSQLDB</strong></td>
</tr>
</tbody>
</table></li>
<li>
<p>Expand the <strong>Test endpoint connection (optional)</strong> section, and choose the VPC that has a label ending in <em>DMS Lab v2</em>.</p>
</li>
<li>
<p>Next click <strong>Create endpoint</strong>.</p>
</li>
<li>
<p>Next, on the <strong>Endpoints</strong> section of the DMS console you will see both the Source and the Target Endpoints created.</p>
</li>
<li>
<p>Select the Target Endpoint and click on the <strong>Actions</strong> menu on the right and select <strong>Test connection</strong>.<br>
<img src="https://s3.amazonaws.com/migrationtechresources2.3/Test+connection.png" alt="enter image description here"></p>
</li>
<li>
<p>The Test should be successful. Please do not proceed to the next step if the Test is not successful.</p>
</li>
</ol>
<h3 id="check-database-content-and-run-a-dms-replication-task-for-full-replication">Check database content, and run a DMS replication task for full replication</h3>
<p>In this exercise, you will check the source database by counting the rows in each table so you can compare the results of the data migration when the task completes. After checking the database, you will create a replication task and perform a data replication.</p>
<p>To connect to the Oracle database and check its size, complete the following steps.</p>
<ol>
<li>
<p>Switch to the Remote Desktop Connection to view the <strong>Source-SCT</strong> Windows server desktop. If you disconnected from the Server-SCT environment, follow the steps in Task 1 to connect to your Windows instance.</p>
</li>
<li>
<p>In the Server-SCT environment, run <strong>Oracle SQL Developer</strong> from the taskbar.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/sql_developer.png" alt="enter image description here"></p>
<ol start="3">
<li>
<p>On the <strong>Connections</strong> menu, right-click <strong>DMS Oracle</strong>, and click <strong>Properties</strong>.</p>
</li>
<li>
<p>In the <strong>New / Select Database Connection</strong> window, complete the following fields, and click <strong>Test</strong>.</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Connection Name</td>
<td><strong>DMS Oracle</strong></td>
</tr>
<tr>
<td>Username</td>
<td><strong>sct_user</strong></td>
</tr>
<tr>
<td>Password</td>
<td><strong>reinvent_Dms_Sct_17</strong></td>
</tr>
<tr>
<td>Save Password</td>
<td><strong>Checked</strong></td>
</tr>
<tr>
<td>Hostname</td>
<td>Use the value of <em>OracleEC2InstancePrivateIP</em> from Qwiklabs Connection Details.</td>
</tr>
<tr>
<td>Port</td>
<td><strong>1521</strong></td>
</tr>
<tr>
<td>SID</td>
<td><strong>xe</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/oracle_developer_connect1.png" alt="enter image description here"></p>
<ol start="5">
<li>
<p>When the connection tests successfully, click <strong>Connect</strong>.</p>
</li>
<li>
<p>To check and record the number of rows in each of the HR database tables, copy the following SQL statements into the <strong>Query Builder</strong> worksheet in the SQL Developer tool:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> <span class="token string">'regions'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>REGIONS <span class="token keyword">UNION</span>   
<span class="token keyword">SELECT</span> <span class="token string">'locations'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>LOCATIONS <span class="token keyword">UNION</span>   
<span class="token keyword">SELECT</span> <span class="token string">'departments'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>DEPARTMENTS <span class="token keyword">UNION</span>   
<span class="token keyword">SELECT</span> <span class="token string">'jobs'</span> TABLE_NAME<span class="token punctuation">,</span>  <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>JOBS <span class="token keyword">UNION</span>   
<span class="token keyword">SELECT</span> <span class="token string">'employees'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span>  HR<span class="token punctuation">.</span>EMPLOYEES <span class="token keyword">UNION</span>   
<span class="token keyword">SELECT</span> <span class="token string">'job_history'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span>  HR<span class="token punctuation">.</span>JOB_HISTORY <span class="token keyword">UNION</span>   
<span class="token keyword">SELECT</span> <span class="token string">'countries'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>COUNTRIES <span class="token keyword">Order</span> <span class="token keyword">by</span> TABLE_NAME<span class="token punctuation">;</span>   
</code></pre>
</li>
<li>
<p>Run the query to see how many rows are in each table in the HR database. Note the results, so you can compare them to the migrated database after the replication.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/oracle_source_query1.png" alt="enter image description here"></p>
<ol start="8">
<li>Before running the DMS replication task, you must disable the foreign keys and triggers on the target database. First, on the Server-SCT EC2 instance, run <strong>pgAdmin4</strong> from the application menu or the Windows taskbar.</li>
</ol>
<p><img src="https://apn-training.s3.amazonaws.com/Screen-shot-4.png" alt="enter image description here"></p>
<ol start="9">
<li>To connect to the Aurora PostgreSQL database, click <strong>Servers &gt; Create &gt; Server</strong>.</li>
</ol>
<p><img src="https://apn-training.s3.amazonaws.com/Screen-shot-5.png" alt="enter image description here"></p>
<ol start="10">
<li>
<p>On the <strong>General</strong> tab, complete the following fields:</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Name</td>
<td><strong>AuroraPostgreSQLDBServer</strong></td>
</tr>
<tr>
<td>Server group</td>
<td><strong>Servers</strong></td>
</tr>
<tr>
<td>Connect now?</td>
<td><strong>Checked</strong></td>
</tr>
<tr>
<td>Comments</td>
<td><strong>Migrated Oracle DB content</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://apn-training.s3.amazonaws.com/Screen-shot-6.png" alt="enter image description here"></p>
<ol start="11">
<li>
<p>On the <strong>Connection</strong> tab, complete the following fields, and then click <strong>Save</strong>.</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Host name/address</td>
<td>Use the value of <em>AuroraPostgreSQLDBEndpoint</em> from the Qwiklabs Connection Details</td>
</tr>
<tr>
<td>Port</td>
<td><strong>5432</strong></td>
</tr>
<tr>
<td>Maintenance database</td>
<td><strong>AuroraPostgreSQLDB</strong></td>
</tr>
<tr>
<td>Username</td>
<td><strong>sct_user</strong></td>
</tr>
<tr>
<td>Password</td>
<td><strong>reinvent_Dms_Sct_17</strong></td>
</tr>
<tr>
<td>Save password?</td>
<td><strong>Checked</strong></td>
</tr>
<tr>
<td>Role</td>
<td><strong>Cleared</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://apn-training.s3.amazonaws.com/Screen-shot-7.png" alt="enter image description here"></p>
<ol start="12">
<li>Click the <strong>hr</strong> schema, and click <strong>Tools &gt; Query Tool</strong>. Alternatively, right-click the <strong>hr</strong> schema, and click <strong>Query Tool</strong>. This opens the Query Tool.</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/postgres_admin_connect.png" alt="enter image description here"></p>
<ol start="13">
<li>
<p>Drop foreign keys in the hr table by using the following SQL ALTER TABLE commands. You can also open the text (.sql) file that you exported during the schema conversion to copy the SQL commands. Paste the ALTER TABLE commands into the Query Tool editor, and click <strong>Run</strong>.</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>countries <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> countr_reg_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>departments <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> dept_loc_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>departments <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> dept_mgr_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>employees <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> emp_dept_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>employees <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> emp_job_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>employees <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> emp_manager_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>job_history <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> jhist_dept_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>job_history <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> jhist_emp_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>job_history <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> jhist_job_fk<span class="token punctuation">;</span>   
<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>locations <span class="token keyword">DROP</span> <span class="token keyword">CONSTRAINT</span> loc_c_id_fk<span class="token punctuation">;</span>   
</code></pre>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/disable_foriegn_key.png" alt="enter image description here"></p>
<ol start="14">
<li>
<p>Disable database triggers in the hr table by using the following SQL DROP TRIGGER commands. You can also open the text (.sql) file that you exported during the schema conversion to copy the SQL statements. Paste the DROP TRIGGER commands into the Query Tool editor, and click <strong>Run</strong>.</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">DROP</span> <span class="token keyword">TRIGGER</span> <span class="token keyword">IF</span> <span class="token keyword">EXISTS</span> secure_employees <span class="token keyword">ON</span> hr<span class="token punctuation">.</span>employees<span class="token punctuation">;</span>   
<span class="token keyword">DROP</span> <span class="token keyword">TRIGGER</span> <span class="token keyword">IF</span> <span class="token keyword">EXISTS</span> update_job_history <span class="token keyword">ON</span> hr<span class="token punctuation">.</span>employees<span class="token punctuation">;</span>   
</code></pre>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/disable_trigger.png" alt="enter image description here"></p>
<h3 id="run-a-dms-replication-task">Run a DMS replication task</h3>
<p>In this exercise, you will create, configure, and run a DMS replication task to load the data from the Oracle database into the Aurora PostgreSQL database.</p>
<ol>
<li>Switch to the browser window that runs the AWS DMS console. In the DMS console, click <strong>Database migration tasks</strong> and Select <strong>Create task</strong>.</li>
</ol>
<p><img src="https://s3.amazonaws.com/migrationtechresources2.3/Screen+Shot+2020-02-05+at+12.41.23+PM.png" alt="enter image description here"></p>
<ol start="2">
<li>
<p>In the <strong>Create Task</strong> window, complete the following fields:</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Task name</td>
<td><strong>replication-task-1</strong></td>
</tr>
<tr>
<td>Replication instance</td>
<td>Choose the replication instance you created in the previous exercise.</td>
</tr>
<tr>
<td>Source endpoint</td>
<td><strong>oracle-source</strong></td>
</tr>
<tr>
<td>Target endpoint</td>
<td><strong>aurora-postgresql-target</strong></td>
</tr>
<tr>
<td>Migration type</td>
<td><strong>Migrate existing data</strong></td>
</tr>
<tr>
<td>Start task on create</td>
<td><strong>Checked</strong></td>
</tr>
<tr>
<td><em>Task Settings</em></td>
<td></td>
</tr>
<tr>
<td>Target table preparation mode</td>
<td><strong>Truncate</strong></td>
</tr>
<tr>
<td>Include LOB columns in replication</td>
<td><strong>Limited LOB mode</strong></td>
</tr>
<tr>
<td>Max LOB size (kb)</td>
<td><strong>32</strong></td>
</tr>
<tr>
<td>Enable validation</td>
<td><strong>Checked</strong></td>
</tr>
<tr>
<td>Enable logging</td>
<td><strong>Checked</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3.amazonaws.com/migrationtechresources2.3/Screen+Shot+2020-02-05+at+12.44.30+PM.png" alt="enter image description here"></p>
<pre><code>&gt; **Note** Enabling logging helps you debug issues that DMS encounters while migrating data.
</code></pre>
<ol start="3">
<li>
<p>The replication task requires selection rules that configure which schema and table data to migrate. The task also requires transformation rules to convert uppercase strings used in the Oracle schema to lowercase strings for PostgreSQL. The transformations must be applied to <em>schemas</em>, <em>tables</em>, and <em>columns</em>. In the <strong>Table mappings</strong> section, for <strong>Selection rules</strong>, complete the following fields, and click <strong>Add new selection rule</strong>.</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Schema name is</td>
<td><strong>HR</strong></td>
</tr>
<tr>
<td>Table name is like</td>
<td><strong>%</strong></td>
</tr>
<tr>
<td>Action</td>
<td><strong>Include</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3.amazonaws.com/migrationtechresources2.3/1.png" alt="enter image description here"></p>
<ol start="4">
<li>
<p>To create three transformation rules that convert uppercase schema, column, and table names to lowercase, complete the following fields, by expanding <strong>Transformation rules</strong> and clicking on <strong>Add new transformation rule</strong> for each rule.<br>
<img src="https://s3.amazonaws.com/migrationtechresources2.3/Screen+Shot+2020-02-05+at+12.53.01+PM.png" alt="enter image description here"><br>
4.a Click on **Add new transformation<br>
Transform schema names</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Target</td>
<td><strong>Schema</strong></td>
</tr>
<tr>
<td>Schema name is</td>
<td><strong>HR</strong></td>
</tr>
<tr>
<td>Action</td>
<td><strong>Make lowercase</strong></td>
</tr>
</tbody>
</table><p><img src="https://s3.amazonaws.com/migrationtechresources2.3/2.png" alt="enter image description here"><br>
4.b Click on <strong>Add new transformation rule</strong><br>
Transform table names</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Target</td>
<td><strong>Table</strong></td>
</tr>
<tr>
<td>Schema name is</td>
<td><strong>HR</strong></td>
</tr>
<tr>
<td>Table name is like</td>
<td><strong>%</strong></td>
</tr>
<tr>
<td>Action</td>
<td><strong>Make lowercase</strong></td>
</tr>
</tbody>
</table></li>
</ol>
<p><img src="https://s3.amazonaws.com/migrationtechresources2.3/3.png" alt="enter image description here"></p>
<p>4.c Click on <strong>Add new transformation rule</strong></p>
<p>Transform column names<br>
| Field | Value |<br>
| — | — |<br>
| Target | <strong>Column</strong> |<br>
| Schema name is | <strong>HR</strong> |<br>
| Table name is like | <strong>%</strong> |<br>
| Column name is like | <strong>%</strong> |<br>
| Action | <strong>Make lowercase</strong> |</p>
<p><img src="https://s3.amazonaws.com/migrationtechresources2.3/4.png" alt="enter image description here"></p>
<ol start="5">
<li>
<p>To save the replication task configuration and start running the task, click <strong>Create Task</strong> at the bottom of the page.</p>
</li>
<li>
<p>Go to the <strong>Tasks</strong> page, and check the task’s status. When the task completes, your data should be replicated to the target database.</p>
</li>
<li>
<p>To check the migrated data’s statistics, click the <strong>Table statistics</strong> tab. You can see how many rows are loaded into each table.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/table_stats.png" alt="enter image description here"></p>
<ol start="8">
<li>
<p>Switch to the Remote Desktop Connection that shows  Server-SCT.</p>
</li>
<li>
<p>Check the replicated tables and their row counts by using the PostgreSQL <strong>pgAdmin 4</strong> tool.</p>
</li>
<li>
<p>Paste the following SQL into the Query Tool, and click <strong>Run</strong>.</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> <span class="token string">'regions'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>REGIONS <span class="token keyword">UNION</span>
<span class="token keyword">SELECT</span> <span class="token string">'locations'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>LOCATIONS <span class="token keyword">UNION</span>
<span class="token keyword">SELECT</span> <span class="token string">'departments'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>DEPARTMENTS <span class="token keyword">UNION</span>
<span class="token keyword">SELECT</span> <span class="token string">'jobs'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>JOBS <span class="token keyword">UNION</span>
<span class="token keyword">SELECT</span> <span class="token string">'employees'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>EMPLOYEES <span class="token keyword">UNION</span>
<span class="token keyword">SELECT</span> <span class="token string">'job_history'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>JOB_HISTORY <span class="token keyword">UNION</span>
<span class="token keyword">SELECT</span> <span class="token string">'countries'</span> TABLE_NAME<span class="token punctuation">,</span> <span class="token function">COUNT</span><span class="token punctuation">(</span><span class="token operator">*</span><span class="token punctuation">)</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>COUNTRIES <span class="token keyword">Order</span> <span class="token keyword">by</span> TABLE_NAME<span class="token punctuation">;</span>
</code></pre>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/oracle_dev_stats1.png" alt="enter image description here"></p>
<pre><code>&gt; **Note** The tables and table counts in the pgAdmin4 tool should match the statistics from the DMS task table statistics and the previous Oracle SQL Developer query.
</code></pre>
<h3 id="restore-the-foreign-keys">Restore the foreign keys</h3>
<p>After completing the full-load replication, you must re-create the foreign keys on the target database. In this exercise, you will use pgAdmin4 to add the foreign keys.</p>
<ol>
<li>
<p>Paste the following SQL into the Query Tool window of the pgAdmin4 tool, and click <strong>Run</strong>. Alternatively, you can open the text (.sql) file  you exported during the schema conversion and find the SQL statements that add constraints.</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>locations   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> loc_c_id_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>country_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>countries <span class="token punctuation">(</span>country_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>countries   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> countr_reg_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>region_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>regions <span class="token punctuation">(</span>region_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>employees   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> emp_dept_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>department_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>departments <span class="token punctuation">(</span>department_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>job_history   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> jhist_dept_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>department_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>departments <span class="token punctuation">(</span>department_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>departments   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> dept_loc_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>location_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>locations <span class="token punctuation">(</span>location_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>departments   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> dept_mgr_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>manager_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>employees <span class="token punctuation">(</span>employee_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>employees   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> emp_manager_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>manager_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>employees <span class="token punctuation">(</span>employee_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>job_history   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> jhist_emp_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>employee_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>employees <span class="token punctuation">(</span>employee_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>employees   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> emp_job_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>job_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>jobs <span class="token punctuation">(</span>job_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   

<span class="token keyword">ALTER</span> <span class="token keyword">TABLE</span> hr<span class="token punctuation">.</span>job_history   
<span class="token keyword">ADD</span> <span class="token keyword">CONSTRAINT</span> jhist_job_fk <span class="token keyword">FOREIGN</span> <span class="token keyword">KEY</span> <span class="token punctuation">(</span>job_id<span class="token punctuation">)</span>   
<span class="token keyword">REFERENCES</span> hr<span class="token punctuation">.</span>jobs <span class="token punctuation">(</span>job_id<span class="token punctuation">)</span>   
<span class="token keyword">ON</span> <span class="token keyword">DELETE</span> <span class="token keyword">NO</span> <span class="token keyword">ACTION</span><span class="token punctuation">;</span>   
</code></pre>
</li>
</ol>
<h3 id="summary-1">Summary</h3>
<p>In Task 2, you learned how to perform a heterogeneous database migration, from Oracle to Aurora PostgreSQL, by using the AWS Database Migration Service. AWS DMS provides full-load database data replication and data change capture in real time.</p>
<h2 id="using-continuous-data-replication-optional">Using Continuous Data Replication (Optional)</h2>
<p>In this optional exercise, you will learn how to use continuous data replication to automatically synchronize changes in the source database to the target database. You will create a DMS replication task for change data capture (CDC) from the source Oracle to the target Aurora database. After the change data capture task completes, you will restore the triggers on the target database.</p>
<p>In general, you will complete the following tasks in this exercise:</p>
<ul>
<li>Enable CDC on the source database</li>
<li>Set up and run a replication task</li>
<li>Introduce changes at the source database</li>
<li>Validate the CDC result in the target database</li>
<li>Enable triggers after the CDC completes</li>
</ul>
<p>You must set up supplemental logging on the Oracle database to capture data, as described in the following steps.</p>
<ol>
<li>
<p>Log in to the Oracle EC2 instance at the <em>Oracle EC2 instance public address</em> by using a standalone SSH client, such as PuTTY, or the terminal utility on Mac or Linux. To locate the Oracle EC2 instance’s public IP address and connection instructions, complete the following steps:</p>
<ul>
<li>In the AWS EC2 console, click <strong>Instances</strong>.</li>
<li>Click the instance with the name <strong>Oracle</strong>.</li>
<li>Click <strong>Connect</strong>. The <strong>Connect To Your Instance</strong> window displays, which contains connection information that you can use to connect and log in. Notice  the <strong>Connect To Your Instance</strong> window shows the log in command for Linux or Mac terminal users. Windows users can use PuTTY with the <strong>EC2 Key Pair Private Key</strong> in PPK format provided in the <em>Qwiklabs Connection Details</em>. Windows users can refer to <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html">this page</a> for detailed instructions.</li>
</ul>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">ssh</span> -i <span class="token operator">&lt;</span>path/to/keypair<span class="token operator">&gt;</span> ubuntu@xxx.xxx.xxx.xxx
</code></pre>
</li>
<li>
<p>Once logged in, switch to the <strong>root</strong> user that runs the Oracle database container with password <strong>admin</strong>, as follows:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">ssh</span> root@localhost -p 49160
</code></pre>
</li>
<li>
<p>To switch to the <strong>oracle</strong> user, create a log archive directory, and run the sqlplus utility, use the following commands:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">su</span> - oracle
<span class="token function">mkdir</span> /u01/app/oracle/log_archive
<span class="token function">chmod</span> 777 /u01/app/oracle/log_archive/
sqlplus <span class="token string">"/ as sysdba"</span>
</code></pre>
</li>
<li>
<p>To check the database for compatibility, run the following command at the <strong>SQL&gt;</strong> prompt:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> name<span class="token punctuation">,</span><span class="token keyword">value</span><span class="token punctuation">,</span>description <span class="token keyword">FROM</span> v$parameter <span class="token keyword">WHERE</span> name<span class="token operator">=</span><span class="token string">'compatible'</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p>To check whether supplemental logging is enabled, use the following command:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> supplemental_log_data_min <span class="token keyword">FROM</span> v$<span class="token keyword">database</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p>If supplemental logging is not enabled, enable it by running the following SQL command:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">ALTER</span> <span class="token keyword">DATABASE</span> <span class="token keyword">ADD</span> SUPPLEMENTAL LOG <span class="token keyword">DATA</span><span class="token punctuation">;</span>
<span class="token keyword">ALTER</span> <span class="token keyword">DATABASE</span> <span class="token keyword">ADD</span> SUPPLEMENTAL LOG <span class="token keyword">DATA</span> <span class="token punctuation">(</span><span class="token keyword">PRIMARY</span> <span class="token keyword">KEY</span><span class="token punctuation">)</span> <span class="token keyword">COLUMNS</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p>To set up the database redo log with the log archive directory that you created previously, run the following command:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">ALTER</span> SYSTEM <span class="token keyword">SET</span> LOG_ARCHIVE_DEST_1 <span class="token operator">=</span> <span class="token string">'LOCATION=/u01/app/oracle/log_archive'</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p>To set the database logging mode to archive mode, use the following command:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SHUTDOWN</span> IMMEDIATE<span class="token punctuation">;</span>
STARTUP MOUNT<span class="token punctuation">;</span>
<span class="token keyword">ALTER</span> <span class="token keyword">DATABASE</span> ARCHIVELOG<span class="token punctuation">;</span>
<span class="token keyword">ALTER</span> <span class="token keyword">DATABASE</span> <span class="token keyword">OPEN</span><span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p>To verify the archive mode and archive destination setup, run the following <strong>SQL</strong> command:</p>
<pre class=" language-sql"><code class="prism  language-sql">ARCHIVE LOG LIST<span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p>Review the text:</p>
<pre><code>Database log mode	          Archive Mode
Automatic archival	          Enabled
Archive destination	          /u01/app/oracle/log_archive
Oldest online log sequence    5
Next log sequence to archive  6
Current log sequence	      6
</code></pre>
</li>
<li>
<p>To switch the log file, use the following command at the <strong>SQL&gt;</strong> prompt:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">ALTER</span> SYSTEM SWITCH LOGFILE<span class="token punctuation">;</span>
<span class="token keyword">ALTER</span> SYSTEM ARCHIVE LOG <span class="token keyword">CURRENT</span><span class="token punctuation">;</span>
</code></pre>
</li>
</ol>
<h3 id="run-a-dms-replication-task-for-cdc">Run a DMS replication task for CDC</h3>
<p>To create and run a DMS replication task, perform the following steps, which create a new task to replicate ongoing data changes.</p>
<ol>
<li>
<p>Switch to the AWS DMS console, and go to the <strong>Tasks</strong> page.</p>
</li>
<li>
<p>Click <strong>Create task</strong>, and create a replication task with the following parameter values:</p>

<table>
<thead>
<tr>
<th>Field</th>
<th>Value</th>
</tr>
</thead>
<tbody>
<tr>
<td>Task name</td>
<td><strong>replication-cdc</strong></td>
</tr>
<tr>
<td>Replication instance</td>
<td>Choose the replication instance you created earlier.</td>
</tr>
<tr>
<td>Source endpoint</td>
<td><strong>oracle-source</strong></td>
</tr>
<tr>
<td>Target endpoint</td>
<td><strong>aurora-postgres-target</strong></td>
</tr>
<tr>
<td>Migration type</td>
<td><strong>Replicate data changes only</strong></td>
</tr>
<tr>
<td>Start task on create</td>
<td><strong>Checked</strong></td>
</tr>
<tr>
<td><em>Task Settings</em></td>
<td></td>
</tr>
<tr>
<td>CDC start mode</td>
<td><strong>Don’t use custom CDC start mode</strong></td>
</tr>
<tr>
<td>CDC stop mode</td>
<td><strong>Don’t use custom CDC stop mode</strong></td>
</tr>
<tr>
<td>Create recovery table on target DB</td>
<td><strong>Cleared</strong></td>
</tr>
<tr>
<td>Target table preparation mode</td>
<td><strong>Do nothing</strong></td>
</tr>
<tr>
<td>Include LOB columns in replication</td>
<td><strong>Limited LOB mode</strong></td>
</tr>
<tr>
<td>Max LOB size (kb)</td>
<td><strong>32</strong></td>
</tr>
<tr>
<td>Enable validation</td>
<td><strong>Cleared</strong></td>
</tr>
<tr>
<td>Enable Logging</td>
<td><strong>Checked</strong></td>
</tr>
</tbody>
</table></li>
<li>
<p>In the <strong>Table mappings</strong> section, enter the same selection and transformation rules you created earlier in the full-load replication task. The rules are summarized here:</p>
<p><strong>Selection rules</strong></p>
<ul>
<li>where <strong>schema name</strong> is like <strong>‘HR’</strong> and <strong>table name</strong> is like <strong>‘%’</strong>, <strong>include</strong></li>
</ul>
<p><strong>Transformation rules</strong></p>
<ul>
<li>For <strong>schema</strong> where <strong>schema name</strong> is like <strong>‘HR’</strong>, <strong>make lowercase</strong></li>
<li>For <strong>table</strong> where <strong>schema name</strong> is like <strong>‘HR’</strong> and <strong>table name</strong> is like <strong>‘%’</strong>, <strong>make lowercase</strong></li>
<li>For <strong>column</strong> where <strong>schema name</strong> is like <strong>'HR</strong>’ and <strong>table name</strong> and <strong>column name</strong> is like <strong>‘%’</strong>, <strong>make lowercase</strong></li>
</ul>
</li>
<li>
<p>To save and start running the replication task, click <strong>Create task</strong>. In a few minutes, the task’s status changes to <strong>Replication ongoing</strong>. The task automatically replicates changes you make to the Oracle database into the Aurora PostgreSQL database.</p>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/repl_task.png" alt="enter image description here"></p>
<h3 id="introduce-changes-to-the-source-database-and-validate-replication">Introduce changes to the source database, and validate replication</h3>
<p>To introduce changes into the Oracle database and watch them appear in the PostgreSQL database, complete the steps in this section.</p>
<ol>
<li>
<p>Switch to the Remote Desktop Connection that shows the Windows EC2 instance, and log in to the Oracle SQL Developer that connects to the source Oracle database.</p>
</li>
<li>
<p>To see the Regions entered in the HR REGIONS table, paste the following SQL command into the <strong>Query Builder</strong>:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> <span class="token operator">*</span> <span class="token keyword">FROM</span> HR<span class="token punctuation">.</span>REGIONS<span class="token punctuation">;</span>
</code></pre>
</li>
<li>
<p>Review the results:</p>
<pre><code>REGION_ID   REGION_NAME
    1         Europe
    2         Americas
    3         Asia
    4         Middle East and Africa
4 rows selected.
</code></pre>
<p><img src="../../images/053-sct-sql-select-hr-regions.png" alt="Figure 53: Selecting HR Regions" title="Figure 53: Selecting HR Regions"></p>
</li>
<li>
<p>To add two data sets to the table, run the following commands:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">INSERT</span> <span class="token keyword">INTO</span> HR<span class="token punctuation">.</span>REGIONS <span class="token keyword">VALUES</span> <span class="token punctuation">(</span><span class="token number">5</span><span class="token punctuation">,</span><span class="token string">'APAC'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">INSERT</span> <span class="token keyword">INTO</span> HR<span class="token punctuation">.</span>REGIONS <span class="token keyword">VALUES</span> <span class="token punctuation">(</span><span class="token number">6</span><span class="token punctuation">,</span><span class="token string">'LATIN AMERICA'</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">COMMIT</span> <span class="token keyword">WORK</span><span class="token punctuation">;</span>
</code></pre>
<blockquote>
<p><strong>Note</strong> You can confirm the data insertion by repeating the SELECT command in the previous step.</p>
</blockquote>
</li>
<li>
<p>To validate the CDC replication result at the target database, complete the following steps:</p>
<ul>
<li>
<p>Log in (or switch) to pgAdmin4, which is connected to the target Aurora PostgreSQL database.</p>
</li>
<li>
<p>Run the following SQL command:</p>
<pre class=" language-sql"><code class="prism  language-sql"><span class="token keyword">SELECT</span> <span class="token operator">*</span> <span class="token keyword">FROM</span> hr<span class="token punctuation">.</span>regions<span class="token punctuation">;</span>
</code></pre>
</li>
</ul>
</li>
</ol>
<p><img src="https://s3-us-west-2.amazonaws.com/migration-training-resources/migration-training-labs/latest/dms_lab_v2/pg_final.png" alt="enter image description here"></p>
<h3 id="restore-triggers">Restore triggers</h3>
<p>After completing the CDC, you must restore the triggers in the target database. To restore the triggers, run the following SQL commands in the pgAdmin 4 utility:</p>
<pre><code>```sql
CREATE TRIGGER secure_employees
BEFORE INSERT OR UPDATE OR DELETE
ON hr.employees
FOR EACH STATEMENT
EXECUTE PROCEDURE hr.secure_employees$employees();

CREATE TRIGGER update_job_history
AFTER UPDATE OF JOB_ID, DEPARTMENT_ID
ON hr.employees
FOR EACH ROW
EXECUTE PROCEDURE hr.update_job_history$employees();
```   
</code></pre>
<blockquote>
<p><strong>Note</strong> Alternatively, you can open the text (.sql) file you previously saved during the schema migration and use the CREATE TRIGGER commands stored there.</p>
</blockquote>
<h2 id="summary-2">Summary</h2>
<p>In this exercise, you learned how to use continuous data replication to automatically synchronize changes in the source database to the target database. You created a DMS replication task for change data capture from the source Oracle to the target Aurora database. After the CDC task completed, you restored the triggers on the target database.</p>

