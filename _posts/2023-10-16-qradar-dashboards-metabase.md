---
full_title: "Quick QRadar Dashboards with Metabase BI"
description: "A tutorial on how to create IBM QRadar SIEM dashboards on Metabase BI (Business Intelligence) tool."
date: "2023-10-16"
coverImage: "metabase_42.png"
layout: post
tags:
- Beginner
- QRadar
- QRadar-Reports
- Metabase
- Dashboard
- Visualization
- BI
- Business-Intelligence
- CSV
- SIEM 
- IBM
- Security
- Tutorial
- MySQL
- VM
- VirtualBox
sitemap:
    lastmod: 2023-10-16
---
## Introduction

Have you ever wanted to quickly create an interactive QRadar Dashboard on a modern, open-source, self-service Business Intelligence (BI) tool?

In this step-by-step tutorial, we will learn how to leverage [Metabase](https://www.metabase.com/) and its new [CSV upload feature](https://www.metabase.com/product/csv-uploads) to import data exports from QRadar and create interactive Dashboards to gather valuable insights.

> Note: This tutorial assumes you have *admin* access to a live QRadar deployment. 
> For the purpose of this tutorial, I am using [QRadar Community Edition](https://www.ibm.com/community/101/qradar/ce/). Please follow my step-by-step guide - [How to install IBM QRadar CE V7.3.3 on VirtualBox](https://diaryofarjun.com/blog/install-qradar-ce-on-virtualbox) to get a basic QRadar deployment up and running in your lab environment.

## Pre-requisites

- QRadar with admin access
> I am using QRadar CE V7.3.3 as described above.
- MySQL
> I am using MySQL Ver 8.0.34 on a CentOS 7 Linux VM.
> For more information about installing MySQL 8.0 on your OS, please refer to [MySQL Installation Guide](https://dev.mysql.com/doc/mysql-installation-excerpt/8.0/en/).
- Metabase Open Source Edition
> I am using Metabase v0.47.2 on a CentOS 7 Linux VM.
> For more information about installing Metabase Open Source Edition on your OS, please refer to [Metabase Open Source Edition](https://www.metabase.com/start/oss/).

## Metabase

According to [Metabase documentation](https://www.metabase.com/learn/getting-started/tour-of-metabase):

> Metabase is an open-source business intelligence tool. Metabase lets you ask questions about your data, and displays answers in formats that make sense, whether that’s a bar chart or a detailed table.
> 
> You can save your questions, and group questions into handsome dashboards. Metabase also makes it easy to share questions and dashboards with the rest of your team.

### CSV Uploads on Metabase

According to [Metabase documentation](https://www.metabase.com/docs/latest/databases/uploads):
> You can upload data in CSV format to Metabase and start asking questions about it. This feature is best suited for ad hoc analysis of spreadsheet data. If you have a lot of data, or will need to update or add to that data regularly, we recommend setting up a way to load that data into a database directly, then connecting Metabase to that database.

The above snippet from the documentation aptly summarizes the benefits and drawbacks of the CSV feature.

In the past, the only available option was to connect Metabase to a [supported database](https://www.metabase.com/docs/latest/databases/connecting#connecting-to-supported-databases). From our perspective, this means that we need to setup ETL (Extract-Transform-Load) pipelines to fetch data from QRadar (using REST APIs), perform transformations, and persist the transformed data into database tables. 

Obviously, it is no simple feat to write, test, and maintain production-ready ETL pipelines. While it is still necessary for most reporting use cases, it is overkill for creating ad hoc Dashboards with quickly exported data. Hence, this new feature from Metabase is a blessing. It is similar to the functionality offered by other popular BI tools such as [Power BI](https://learn.microsoft.com/en-us/power-bi/connect-data/service-comma-separated-value-files).

> Note: Please refer to my blog post titled [QRadar REST APIs with Logstash](/blog/qradar-logstash) to learn how to develop ETL pipelines on Logstash to programatically fetch raw data from QRadar REST APIs, apply processing, and output into various formats and destinations.

## Creating Dashboards

In this section, we will delve into the steps required to create our desired Dashboard on Metabase.

First, we will start by exporting the required CSV data from the QRadar Console. The next step involves configuring Metabase to accept CSV uploads. However, prior to enabling CSV uploads on Metabase, we need to create a new MySQL database and connect it to Metabase. Once the MySQL database is connected to Metabase, we can enable CSV uploads and choose the newly created database as the database to be used for uploads. Next, we will upload the exported QRadar CSV to Metabase as a [Model](https://www.metabase.com/glossary/model). This step also involves configuring the appropriate column types. Finally, we will leverage the Model to ask [Questions](https://www.metabase.com/glossary/question) and create a new [Dashboard](https://www.metabase.com/glossary/dashboard) with multiple metrics and visualizations.

### Exporting QRadar Data

The first step involves exporting the necessary data from the QRadar Console. For the purpose of this tutorial, we will export Offenses from QRadar.

Log in to the QRadar Console. Click on **Offenses** to navigate to the Offenses tab.

![QRadar Dashboard](/assets/images/metabase_qradar_1.png)

In the Offenses tab, the latest *active* Offenses are displayed. Click on **Actions**.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_1.png)

Under the Actions menu, select **Export to CSV**.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_2.png)

The export will commence. The duration of the export will be determined by the number of Offenses to be exported. Ensure your [filters](https://www.ibm.com/docs/en/qsip/7.4?topic=searches-offense) are appropriately set *prior* to initiating the export.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_3.png)

Download the compressed ZIP file to a local directory.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_4.png)

Unzip the compressed file to extract the CSV file.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_5.png)

For the sake of clarity, rename the CSV file to `offenses`.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_6.png)

Open the CSV file in Excel (or a text editor of your choice) to view its contents. Validate the columns and rows. The number of Offenses on the CSV file must match the number displayed on the Offenses tab on the QRadar Console.

> Note: It is to be expected that the export will contain ALL the relevant columns pertaining to each Offense.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_7.png)

For the purpose of this tutorial, we will purge a couple of columns and retain only a few relevant ones.

> Note: The retained columns are: `id`, `magnitude`, `description`, `credibility`, `severity`, `relevance`, `eventCount`, `flowCount`, `attacker`, `target`, `formattedStartTime`, `formattedEndTime`.

![QRadar Dashboard](/assets/images/metabase_qradar_offenses_8.png)


### Configuring Metabase

With the Offenses exported from QRadar, the next step involves configuring Metabase to enable the CSV upload feature.

#### Configuring MySQL Database on Metabase

According to [Metabase documentation](https://www.metabase.com/docs/latest/databases/uploads#enabling-uploads):
> There are a few things admins need to do to support CSV uploads:
>
> 1. Connect to a database using a database user account with write access. This way Metabase will be able to store the uploaded data somewhere.
> 2. Select the database and schema you want to store the uploaded data in.
> 3. Add people to a group with unrestricted data access to the upload schema database.
> 4. (Optional) specify a prefix for Metabase to prepend to the uploaded tables.

Essentially, this means that we need a database that will be used to store the uploaded CSV data. As mentioned in the [pre-requisites](#pre-requisites), we have chosen MySQL. However, you can also choose PostgreSQL, which is the [only other database](https://www.metabase.com/docs/latest/databases/uploads#databases-that-support-uploads) that supports CSV uploads on Metabase.

To connect the MySQL database with Metabase, start by connecting to MySQL. I am using the MySQL client (`mysql`).

![MySQL](/assets/images/metabase_mysql_1.png)

Create a new database called `qradar` using the command: `CREATE DATABASE qradar;`

> Note: Use the `SHOW DATABASES` command to view the existing databases on MySQL.

![MySQL](/assets/images/metabase_mysql_2.png)

Now that we have created the database on MySQL, the next step is to configure it on Metabase.

Log in to Metabase. Click on the Settings icon on the top-right to open the Settings menu.

![Metabase](/assets/images/metabase_1.png)

Click on **Admin settings**.

![Metabase](/assets/images/metabase_2.png)

On the Admin settings page, click on the **Databases** tab.

![Metabase](/assets/images/metabase_3_databases.png)

On the **Databases** page, click on **Add database**.

![Metabase](/assets/images/metabase_4.png)

On the **Add databases** page, populate the form with connection details to the MySQL database. Click on **Save**.

> Note: It is pertinent to ensure that the connection details are accurate. We have used `127.0.0.1` since MySQL and Metabase are on the *same* CentOS 7 Linux VM. Depending on your setup, you may need to add/modify firewall rules to ensure connectivity.

![Metabase](/assets/images/metabase_5.png)

If all goes well, a pop-up will appear on the bottom-right indicating that the database was added and synced successfully.

![Metabase](/assets/images/metabase_6.png)

#### Configuring CSV Uploads on Metabase

Navigate back to the Admin settings page. Click on the **Uploads** tab on the left.

![Metabase](/assets/images/metabase_3_uploads.png)

On the Uploads page, click on the **Select a database** dropdown. 

![Metabase](/assets/images/metabase_7.png)

Select **QRadar_MySQL** from the dropdown.

![Metabase](/assets/images/metabase_8.png)

Once selected, an input box titled **Upload Table Prefix (optional)** will appear. Although it is optional, I have appended `qradar` for the sake of this tutorial. The **Enable uploads** button will now be enabled. Click on the button.

![Metabase](/assets/images/metabase_9.png)

If all goes well, the button will turn green and display **Uploads enabled**. Exit the Admin settings page by clicking on **Exit admin** on the top-right.

![Metabase](/assets/images/metabase_10.png)

### Uploading CSVs to Metabase

The next step involves uploading the QRadar Offenses CSV to Metabase.

Navigate to the Metabase home page. Click on the meatballs menu (yes, it's *actually* called [meatballs menu](https://www.computerhope.com/jargon/m/meatballs-menu.htm)) next to **COLLECTIONS**. Click on **+ New collection**.

According to [Metabase documentation](https://www.metabase.com/docs/latest/exploration-and-organization/collections):
> Collections are the main way to organize questions, dashboards, and models. You can think of them like folders or directories. You can nest collections in other collections, and move collections around. One thing to note is that a single item, like a question or dashboard, can only be in one collection at a time (excluding parent collections).

![Metabase](/assets/images/metabase_11.png)

Populate the **New collection** form with a **Name** and an optional **Description**. Click on **Create**. 

![Metabase](/assets/images/metabase_12.png)

The new collection is created. It is empty and is ready to be filled with Questions, Dashboards, Models, etc. 

To upload the Offenses CSV file, click on the **Upload data to QRadar** icon on the top-right.

![Metabase](/assets/images/metabase_13.png)

The file browser pop-up will open. Locate and select the `offenses` CSV file. Click on **Open**.

![Metabase](/assets/images/metabase_14.png)

If all goes well, a pop-up will appear on the bottom-right indicating that the data was added to the **QRadar** collection. 

A new Model, titled **Offenses**, will appear in the Collection. Click on it.

![Metabase](/assets/images/metabase_15.png)

We can see our QRadar Offenses on Metabase. Great!

![Metabase](/assets/images/metabase_16.png)

It is pertinent to validate the Model including the column types and formatting before building Dashboards. To delve into the Model, click on the meatballs menu on the right, and click on **Edit metadata**.

![Metabase](/assets/images/metabase_17.png)

On this page, set the appropriate column type for each column. It is recommended to provide a description for each column to ensure better data governance.

> Note: Set the column type for `ID` as **Entity Key**.

![Metabase](/assets/images/metabase_18.png)

Once completed, click on **Save changes**.

![Metabase](/assets/images/metabase_19.png)

The updated Model will be loaded.

![Metabase](/assets/images/metabase_20.png)

### Questions and Dashboards

The final step involves visualizing Questions and creating a Dashboard on Metabase.

Let us start with a simple metric (Question) - *Number of Offenses*.

To calculate this, we need to essentially perform a *count* operation. Click on **Summarize**.

![Metabase](/assets/images/metabase_21.png)

By default, the *metric* is **Count** indicating the count of rows in the Model. Click on **Done**. Click on **Save**.

![Metabase](/assets/images/metabase_22.png)

Let us save it as a new Question. Click on **Save**.

![Metabase](/assets/images/metabase_23.png)

Populate the **Save new question** form with a **Name** and an optional **Description**. Click on **Save**.

![Metabase](/assets/images/metabase_24.png)

Now, we want to add this newly created Question to a Dashboard. Click on **Yes please!** to proceed.

![Metabase](/assets/images/metabase_25.png)

In the **Add this question to a dashboard** pop-up, select the **QRadar** Collection and click on **+ Create a new dashboard**.

![Metabase](/assets/images/metabase_26.png)

Populate the **New dashboard** form with a **Name** and an optional **Description**. Click on **Create**.

![Metabase](/assets/images/metabase_27.png)

Visualize your data! This is where your creativity can shine.

> Note: Please refer to [this page](https://www.metabase.com/docs/latest/questions/sharing/visualizing-results) from the Metabase documentation which explains in depth about the available visualization types and options.

For this metric (*Number of Offenses*), we have chosen a simple **Number** visualization, which looks like a *scorecard*.

According to [Metabase documentation](https://www.metabase.com/docs/latest/questions/sharing/visualizations/numbers):
> The Numbers option is for displaying a single number, nice and big.

![Metabase](/assets/images/metabase_28.png)

The Dashboard is displayed. Let us add some more visualizations. To do this, you will need to create new Questions. Click on **+ New**.

![Metabase](/assets/images/metabase_29.png)

Click on **Question**.

![Metabase](/assets/images/metabase_30.png)

Click on **Models**.

![Metabase](/assets/images/metabase_31.png)

Select **Offenses** under **QRadar**.

![Metabase](/assets/images/metabase_32.png)

Let us attempt another simple metric (Question) - *Offenses by Magnitude*.

To calculate this, we need to essentially perform a *count* operation followed by a *group-by* operation on the `Magnitude` column. Click on **Visualize**.

The screenshot below illustrates how we leverage the [Metabase Notebook editor](https://www.metabase.com/glossary/notebook_editor) to calculate this metric (Question).

![Metabase](/assets/images/metabase_33.png)

We have a table populated with the result. However, for the Dashboard, we would prefer a visualization. Click on **Visualization** on the bottom-left.

![Metabase](/assets/images/metabase_34.png)

A [bar chart](https://www.metabase.com/learn/visualization/bar-charts) typically works well to represent a simple *distribution*. Again, it's completely your choice on what visualization to pick :) Click on **Done**.

![Metabase](/assets/images/metabase_35.png)

Click on **Save**.

![Metabase](/assets/images/metabase_36.png)

Populate the **Save new question** form with a **Name** and an optional **Description**. Click on **Save**.

![Metabase](/assets/images/metabase_37.png)

Now, we want to add this newly created Question to a Dashboard. Click on **Yes please!** to proceed.

![Metabase](/assets/images/metabase_38.png)

Select our existing **SIEM Offenses Dashboard**.

![Metabase](/assets/images/metabase_39.png)

Add the visualization to the Dashboard. Click on **Save**.

![Metabase](/assets/images/metabase_40.png)

Great! We now have two visualizations on our SIEM Offenses Dashboard. Feel free to come up with your own metrics (Questions) and add them to your Dashboard.

![Metabase](/assets/images/metabase_41.png)

Here's what my final Dashboard looks like!

![Metabase](/assets/images/metabase_42.png)

## Conclusion

In this tutorial, we learnt how to build a simple QRadar Dashboard on Metabase, an open-source BI tool, using its new CSV upload feature.

Metabase is a fantastic BI tool and the CSV upload feature is an absolute game changer. While it is still in its infancy, it seems promising for small SOC/SecOps teams to quickly visualize and create ad hoc Dashboards. That being said, for more resilient and automated reporting, the preferred approach should be to leverage ETL pipelines. With the right *data engineering* and *architecture* in place, Metabase can easily connect to your database/data warehouse and seamlessly refresh Dashboards.

It is to be noted that one of the biggest caveats of the CSV upload feature is to do with the CSV file size limit.

According to [Metabase documentation](https://www.metabase.com/docs/latest/databases/uploads):
> CSV files cannot exceed 50 MB in size.

But, they have offered a workaround:
> If you have a file larger than 200 MB, the workaround here is to:
> 1. Split the data into multiple files.
> 2. Upload those files one by one. Metabase will create a new model for each sheet.
> 3. Consolidate that data by creating a new question or model that joins the data from those constituent models created by each upload.

Using the concepts and steps from this tutorial, you can easily build sophisticated Dashboards with multiple Models representing various QRadar entities such as Offenses, Events, Rules and Networks. If you are limited by the GUI, you can always leverage the [Metabase SQL editor](https://www.metabase.com/docs/latest/questions/native-editor/writing-sql). It is to be noted that Metabase does offer [Pro and Enterprise versions](https://www.metabase.com/pricing/) of their software (cloud and on-prem options available). Depending on your requirements, you may either opt for the open-source version or a premium one.

I hope you enjoyed reading this tutorial. Please reach out if you have any questions or comments.

## Useful Links

- [Metabase Open Source Edition](https://www.metabase.com/start/oss/)
- [Metabase Documentation](https://www.metabase.com/docs/latest/)
- [Learn Metabase](https://www.metabase.com/learn/)
- [Metabase Discussion (discourse)](https://discourse.metabase.com/)
- [QRadar Community Edition](https://www.ibm.com/community/101/qradar/ce/)
- [QRadar Community Edition Quickstart Guide](https://www.ibm.com/security/digital-assets/qradar/community-edition-quickstart-guide/)
- [QRadar Documentation](https://www.ibm.com/docs/en/qsip/7.4)
- [How to install IBM QRadar CE V7.3.3 on VirtualBox](/blog/install-qradar-ce-on-virtualbox)