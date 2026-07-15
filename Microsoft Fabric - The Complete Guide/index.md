# 1. Getting started with Fabric

## 1.1 Overview

Traditional Challenge

- Multiple tools
  - Data Integration
  - Data Science
  - Data Engineering
- Multiple Vendors

All-in-One analytics solution; SaaS (Software as a Service)

- Data Factory
- Data Science
- Data Warehousing
- Data Engineering
- Power BI
- Real-Time Analytics
- Foundation of this platform: **OneLake**

<!-- login email: hohegi3175@lovadio.com -->

Steps

1. Create a Fabric account
2. Create a Fabric workspace (Trial Capacity)

Here is the summary with the headings removed:

**Organizational Architecture**

To understand the pricing, it helps to understand how the platform is structured:

- **Tenant:** The main account for an organization. Usually, one organization uses a single tenant.
- **Capacity:** A dedicated pool of compute resources within the tenant. **This is the primary component you pay for.**

- **Workspace:** A collaborative environment where items and data lakes are built. Workspaces are assigned to a capacity and draw from its underlying compute power.

---

**Capacity Pricing Models**

Capacity costs vary by region and are determined by your chosen billing model.

| Pricing Model     | Billing | Pausable? | Best Use Case                                                        |
| ----------------- | ------- | --------- | -------------------------------------------------------------------- |
| **Pay-as-you-go** | Hourly  | Yes       | Flexible workloads; can be turned off when not needed to halt costs. |
| **Reservation**   | Advance | No        | Consistent, continuous workloads; offers cost savings over time.     |

---

**SKUs, Compute Power, and Licensing**

The amount of compute power you purchase is defined by its SKU and measured in **Capacity Units (CUs)**.

- **Scaling:** Higher SKUs provide more capacity units (double the units equals double the compute power) at a correspondingly higher price.
- **Licensing Rules:**

- **Creators:** Users creating reports still require a Power BI Pro license.
- **Consumers:** The "Power BI free" license is now called "Fabric Free." Starting at the **F64 SKU capacity**, users with Fabric Free licenses can consume and view content without needing a paid Pro license.

---

**Additional Costs: Storage & Networking**

A key takeaway is that **compute and storage are decoupled** and billed separately.

- **Storage:** OneLake storage is very affordable, generally costing around $0.02 to $0.03 per gigabyte per month, depending on the region.
- **Data Transfer (Bandwidth):** Ingress (bringing data in) is free. However, transferring data between different availability zones or regions may incur additional networking costs.

---

**Performance Management: Bursting and Smoothing**

To help handle sudden spikes in activity (like heavy batch jobs in the morning), the system automatically uses **bursting and smoothing**.

- **Bursting:** Temporarily allocates extra compute resources so peak workloads execute faster than your capacity's strict limits would normally allow.
- **Smoothing:** Balances this temporary spike out in the background during periods of lower workload activity.

This process happens automatically, ensuring better performance during peak hours without requiring you to manually upgrade your capacity.

# 2. OneLake & Lakehouses

## 2.1 OneLake

<!-- OneLake

- One Unique Lake for our account
- Central data storage for all data

Shortcuts to external files

- External
- ex. other cloud storage -->

**OneLake Fundamentals**

- **Central Repository:** It acts as an invisible, managed data lake that stores all of an organization's data.
- **Single Instance:** There is only one unique OneLake created per tenant. Every user in the organization connects to this same unified account.

**External Data Integration (Shortcuts)**

- **Zero Data Movement:** You can create "shortcuts" to connect to external files (such as those in AWS or Azure) directly within OneLake.
- **Simplified Access:** Shortcuts eliminate the need to physically move data or build complex data pipelines. The external data appears and functions exactly as if it were natively stored in your OneLake.

**Workspaces and Lakehouses**

- These are the practical environments and primary items built on top of OneLake, which the lecture notes will be explored in depth next.

## 2.2 Workspaces

<!-- workspace - container
onelake has multiple workspaces (= containers)
workspace contain all items, reports, lakehouses ...
workspace setip users and permissions, object can be shared across workspaces -->

**Lakehouses**

- **Core Function:** They serve as the fundamental items for data tasks, acting as the foundation for future work.
- **Hybrid Approach:** They combine the flexible nature of a data lake with the performance-oriented, analytical power of a data warehouse.
- **Location:** Lakehouses are created as distinct items within a workspace.

**Workspaces**

- **Primary Purpose:** They act as visible containers to hold and manage your items, as the underlying OneLake layer is invisible.
- **Organization:** You can create multiple workspaces to logically separate different projects, departments, customers, or users.
- **Item Creation:** Depending on the platform experience you are using, you can build different items here. For example, Data Engineering allows you to create Lakehouses, while Power BI allows you to create reports, dashboards, and data flows.
- **Access Management:** Workspaces are the primary unit for configuring security, allowing you to grant permissions and assign specific roles to users.

**Cross-Workspace Sharing**

- Although workspaces function as self-contained units for permissions, the objects within them (such as a Lakehouse) can still be shared across multiple different workspaces whenever necessary.

## 2.3 Lakehouses

<!-- Items created in workspaces

one location

- Tables(delta tables); Storing, managing & analyzing data
  - SQL endpoint (read-only)
- Files; Structured & Unstractured data

- Shortcuts: reference to other location

Delta tables:

- Based on open source Delta Lake format for Apache Spark
- Capabilities for batch and streaming data.

Structure of Delta Tables:

- Use Parquet format (immutable)
- Folder as transactional log (update, delete, time travel) → transactional modifications

Analytical data store

- Benefits of a relational database (SQL querying)
- Flexibility of file storage in data lake -->

**Lakehouse Overview**

- **Definition:** An item created within a workspace (via the Data Engineering experience) that acts as a single, centralized location for storing, managing, and analyzing data.
- **Hybrid Storage:** It combines the high-performance nature of a data warehouse with the flexibility of a data lake, allowing you to store two main types of data:
- **Structured Data:** Stores "Delta tables," which are highly optimized, rigid tables.
- **Unstructured/Semi-structured Data:** Stores flexible file formats like CSVs or JSONs.

**Integration and Access**

- **SQL Endpoint:** Every lakehouse automatically generates a read-only SQL endpoint. This allows external systems (such as SQL Server Management Studio) to connect and run queries directly against your Delta tables. Because it is read-only, you cannot use this endpoint to modify the data.
- **Tool Integration:** The lakehouse is designed to easily connect with both internal features and external database tools.
- **Shortcuts:** You can use shortcuts to create references to files stored in other locations. This allows external data to appear and function within the lakehouse without needing to physically copy or move the files.

[Roles in workspaces in Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/fundamentals/roles-workspaces)

<!--
SQL Connection string:
vdv6y6fkkp4u3a3m542uy3qplu-piflacrhlvaejmaebuc5hwfr6u.datawarehouse.fabric.microsoft.com
 -->

# 3. Power BI & Semantic models

## 3.1 Understaning semantic models

<!-- Lakehouse -> tables / files -> Load to Tables -> New semantic model -> Name -> Relationship -> Measures / Calculations -> Save

Workspasce -> Open semantic model -->

**Semantic Models in Power BI**

- **Core Function:** They serve as the mandatory foundation for any Power BI report, holding the data structure, tables, and views required for building visualizations.
- **Data Visibility:** Within a model, you can choose to make specific tables invisible in the report view. This hides them from the report creator's right-hand data pane, though the underlying data remains active within the model.

**Relationships and Cardinality**

- **Purpose:** Relationships allow you to combine data from multiple tables (like connecting a transactional `Sales` table with a descriptive `Products` dimension table) to display them together in a single visual.
- **Configuration:** They are established by identifying a common column between two tables and dragging one column key onto another (e.g., matching `Product ID` to `Product ID`).
- **One-to-Many ($1:*$) Cardinality:**

- **The "One" Side ($1$):** Typically the dimension table (e.g., `Products`), where each unique item appears exactly once.
- **The "Many" Side ($*$):** Typically the fact/transaction table (e.g., `Sales`), where a single product can be sold multiple times across different transaction rows.

**Creation and Access Workflows**

To build and manage a semantic model from scratch, the process follows these primary structural stages:

- **Step 1: Prep the Tables**

  Ensure all necessary datasets are loaded as Delta tables within your Lakehouse view.

- **Step 2: Generate the Model**

  Click the **New Semantic Model** button (accessible from either the _Lakehouse view_ or the _SQL endpoint_), select your desired tables and views, and assign a model name.

- **Step 3: Define Schema Relationships**

  Once inside the automated Model View layout, map out your table relationships by linking matching keys (such as `Product ID`) to handle analytical workflows.

- **Step 4: Future Modifications**

  To update an existing model later, return to your workspace, select the specific semantic model item, and choose **Open Semantic Model** to re-enter the configuration interface.

## 3.2 Create new semantic model

**The Evolution to Semantic Models**

In Microsoft Fabric, what were previously known as Power BI "datasets" are now called **Semantic Models**. These models serve as the structural foundation for your Power BI reports, defining tables and relationships so the reporting engine can use the data efficiently.

**Direct Lake Connection Mode**

Fabric introduces a new connectivity mode called **Direct Lake**, which fundamentally changes how Power BI interacts with data by combining the best traits of older methods:

- **Import Mode (Legacy):** Offered excellent performance by loading data into memory, but required duplicating the data and scheduling constant refreshes.
- **Direct Query (Legacy):** Kept data completely up-to-date by querying the database directly, but often suffered from slow, sluggish performance.
- **Direct Lake (New):** Achieves the lightning-fast performance of Import Mode while reading directly from the single source of truth in OneLake. It completely eliminates data duplication and the need for refresh schedules.

**Hands-On Practice: Creating a Custom Semantic Model**

Based on the walkthrough in the transcript, here is a step-by-step exercise you can do to build and test your own model:

1. **Navigate to the Endpoint:**

   Open your workspace and enter either the Lakehouse experience or the SQL Analytics endpoint view.

2. **Initiate Model Creation:**

   Locate and click the **New Semantic Model** button (this is found under the Reporting tab if you are using the SQL endpoint).

3. **Select Tables and Name the Model:**

   Choose only the specific tables you need for your specific use case (for example, select your _Products_ table and _Sales_ table). Give your model a name (e.g., "Sales Model") and confirm.

4. **Define Table Relationships:** Check the cardinality direction.
   Inside the automated Model View, map the connections between your tables. Drag the common key (like `Product ID`) from your dimension table to your fact table. Ensure the cardinality is set correctly, such as a **1-to-many** relationship from Products to Sales.

5. **Create a Test Report:**

   Return to your main workspace, select your newly created semantic model item, and click **Create report**. Build a visual like a stacked column chart using fields from both tables (e.g., _Product Name_ and _Total Price_) to verify that your data relationships are functioning correctly.

<!-- Tables is not available to create within free pro membership -->

## 3.3 Creating measures

**Understanding Measures**

- **Virtual Calculations:** Measures are formula-based calculations that do not add physical data or new rows to your semantic model.
- **Reusability:** Once a measure is created, it becomes a permanent part of the semantic model. This means you can reuse the same exact calculation across multiple different Power BI reports that connect to that model.
- **Location Flexibility:** When writing a measure, you can reference columns from any table in your model. The specific table you choose to create the measure _in_ only dictates where it will be organized and displayed in the user's right-hand Data pane. Logically grouping them (e.g., putting a sales discount measure in the Sales table) improves the user experience.

**Hands-On Practice: Creating and Using a Measure**

Based on the lecture, here is how you can practice creating your own virtual calculation:

1. **Open the Semantic Model:**

   Navigate to your workspace, locate your semantic model item, and open it to view the model layout.

2. **Enable Edit Mode:**

   Once the model view loads, ensure you click to enable **Edit mode** so you are authorized to make structural changes.

3. **Create the Measure:**

   In the Data pane, find the table where you want the measure to be organized (e.g., the _Sales_ table). Click the three dots next to the table name and select **New measure**. A calculator icon will appear indicating the new item.

4. **Define the Formula:**

   In the formula bar, give the measure a descriptive name and write your calculation. Remember that measures require an aggregation function (like SUM). For example, type: `Discounted Price = SUM(Sales[Total Price]) * 0.9` and press Enter to save.

5. **Apply to a Report:** Ensure you click 'Edit' if opening an existing report.
   Open a Power BI report that uses this semantic model. Locate your newly created measure in the Data pane, and drag it into a visual (like a table or chart) alongside your original metrics to compare the computed results.

## 3.4 Connect to Fabric from Power BI Desktop

**Connecting to Power BI Desktop**

Working directly within the Fabric web interface isn't always ideal. You can connect Power BI Desktop directly to your Fabric Lakehouse or custom Semantic Models to build reports in a familiar desktop environment.

**Authentication and Live Connection**

To access your Fabric data, you must sign into Power BI Desktop using your organizational Fabric credentials (including any required two-factor authentication). Connecting to a "Power BI semantic model" creates a live connection to the data residing in OneLake.

**Capabilities and Constraints in Desktop**

- **What you can do:** Build visualizations, drag and drop fields, and create new local measures directly within Power BI Desktop.
- **What you cannot do:** Because it is a live connection to a centrally managed model, the data model view is strictly read-only. You cannot alter table relationships, rename or delete tables, use the "Transform Data" (Power Query) button, or easily combine the model with external data sources like a local SQL server. Any structural changes must be made back in the Fabric web interface.

**Switching Sources and Publishing**

If you need to switch your report to a entirely different semantic model, you must do so through the "Data source settings". Once your report is complete, saving the file and clicking "Publish" will push the report directly back to your selected workspace in the Fabric cloud.

**Hands-On Practice: Connect and Publish from Power BI Desktop**

1. **Authenticate Power BI Desktop:** Ensure you use your Fabric account email.
   Open Power BI Desktop locally and sign in to your organizational account, completing any required two-factor authentication.

2. **Connect to the Semantic Model:**

   Navigate to the **Home** ribbon, click on **Data Hub** (or Get Data), and select **Power BI semantic model**. Choose either your default Lakehouse model or a custom model you created previously, then click Connect.

3. **Build Visuals and Local Measures:**

   Drag fields (such as _Product Name_ and _Total Price_) onto the report canvas to create a chart. Right-click on a table in the Data pane to create a new virtual calculation (e.g., a simple sum measure) and add it to your visual.

4. **Observe Model Constraints:** Notice that Transform Data is unavailable.
   Navigate to the Model view on the left sidebar. Try to delete a table or change a relationship to observe that the model structure is locked and read-only.

5. **Publish to Fabric:**

   Save your Power BI file locally (e.g., "Second_Report.pbix"). Click the **Publish** button on the Home ribbon, select your Fabric workspace, and verify the report now appears in your web browser inside Fabric.

## 3.5 Using SQL endpoint

**Connecting via the SQL Endpoint**

While connecting to a Semantic Model provides an optimized but locked-down experience, you can also connect to your Lakehouse directly using its SQL endpoint. This method treats your Fabric data exactly like a traditional SQL Server database.

**Key Differences from Semantic Models**

- **Complete Flexibility:** Because you are pulling raw tables rather than a predefined model, you regain full structural control. You can modify table relationships, rename or delete items, and open the Power Query Editor to apply data transformations.
- **Multiple Data Sources:** You are no longer restricted to just your Fabric data. You can freely blend the Lakehouse tables with external data sources, such as local Excel workbooks.
- **Data Import:** Connecting this way typically uses Import Mode. The data is downloaded directly into your local machine's memory, which means you are bypassing the specific optimizations of the Direct Lake engine.
- **External Tooling:** The exact same SQL connection string used for Power BI can be pasted into any external database management tool, such as SQL Server Management Studio (SSMS), to query the Lakehouse externally.

**Hands-On Practice: Connect via the SQL Endpoint**

Below are the steps to practice bypassing the semantic model and connecting directly to the raw endpoint:

1. **Locate the Connection String:**

   In your Fabric workspace, open your Lakehouse's SQL endpoint view. Click the gear icon to view the settings and copy the SQL connection string.

2. **Initiate a SQL Server Connection:** Use a fresh file, not the one from the previous exercise.
   Open a new, blank Power BI Desktop file. Select **SQL Server** from the data sources menu, paste your copied connection string (you can leave the database field blank), and click OK.

3. **Authenticate your Account:**

   When prompted for credentials on the left menu, select **Microsoft Account**. Click Sign In, provide your Fabric email and password, and approve the two-factor authentication prompt on your mobile device.

4. **Load or Transform the Data:**

   In the Navigator window, expand your Lakehouse folder to view the available tables. Check the boxes next to the tables you need, then click **Load** (to pull them into memory) or **Transform Data** (to open the Power Query Editor).

5. **Modify the Local Model:**

   Navigate to the Model view on the left sidebar. Unlike the Semantic Model exercise, observe that you can now double-click relationships to change them, delete tables, and use the "Get Data" button to bring in external sources like Excel.

## 3.6 Auto Create Reports

**The Foundation:** Semantic models serve as the necessary baseline for any Power BI report, holding your data, table relationships, and row-level security rules. Because the model already understands how your data connects, Fabric can use it to automatically generate a starting report.

**Auto-Create Capabilities:** When you trigger the auto-create feature, Fabric analyzes your semantic model and builds a pre-populated report canvas. The tool is generally smart about choosing the right visual for the data type—for example, automatically placing timestamps into line charts to show trends over time, or using bar charts for categorical data.

**Limitations and Editing:** The auto-generated result is best viewed as a quick starting point rather than a finished product. The automatically generated titles are often vague, and the visuals lack a cohesive, meaningful story.

- **Quick Tweaks:** You can easily swap out data by selecting or deselecting fields in the data pane, and the layout will dynamically adjust the visuals. You can also quickly change the chart type (e.g., swapping a clustered bar chart for a column chart).
- **Full Customization:** For deeper control over layout, sizing, and formatting, clicking the **Edit** button unlocks the full Power BI report authoring experience.

**Saving and Distribution:** Once you are satisfied with the layout, you save the report to your designated workspace. However, it is important to note that end-users rarely consume the raw report file. Instead, finished reports are typically bundled into Power BI **Apps** for scaled, professional distribution.

**Hands-On Practice: Generate and Edit an Auto-Created Report**

1. **Locate Your Semantic Model:**

   Navigate to your workspace (using either the Power BI or Data Engineering experience) and find the semantic model you want to base your report on.

2. **Trigger Auto-Create:**

   Click the three dots (...) next to the semantic model's name and select **Auto-create report**. Wait a few seconds for the platform to generate the initial canvas.

3. **Test Dynamic Updates:**

   In the data pane on the right, check and uncheck different fields (such as _Product Name_ or _Quantity Sold_). Observe how the generated visualizations automatically adjust to incorporate your selections.

4. **Enter Edit Mode:** This unlocks the full authoring canvas.
   Click the **Edit** button at the top of the screen to transition from the quick-view mode into the full Power BI report editor.

5. **Save the Report:**

   Click Save, provide a name for your file (e.g., "Auto-Generated Sales Report"), and select your destination workspace.

[Create Report and Auto Create in Power BI](https://www.youtube.com/watch?v=E0ZxetVqGIY) provides a direct visual walkthrough of the interface used to generate these automatic visuals from your dataset.

## 3.7 Creating Apps

**Power BI Apps Overview**

Publishing a Power BI App is the official, best-practice method for distributing reports and insights to end-users. Instead of sharing individual report files directly, an App provides a polished, easily navigable environment for consumption.

**Key Constraints and Features**

- **One-to-One Ratio:** You can only create **one** App per workspace.
- **Bundled Content:** A single App can contain multiple reports, pages, and external links, all organized into custom sections and navigation menus.
- **Experience Limitation:** The "Apps" menu is only visible when you are working within the **Power BI experience** in Fabric (you won't find it if you are in the Data Engineering experience).

**Audience Management**

The true power of Apps lies in audience targeting. You do not have to create separate apps for different departments.

- **Custom Audiences:** You can define specific user groups (e.g., "Power Users" vs. "General Staff") within a single App.
- **Granular Visibility:** You can toggle the visibility of specific reports or sections on and off for each audience. For example, the entire organization can see the summary report, but only the finance team can see the detailed transactional report.

---

**Hands-On Practice: Create and Publish an App**

1. **Initiate App Creation:**

   Navigate to your workspace (ensure you are in the Power BI experience) and click the **Create App** button.

2. **Configure Setup Details:**

   Provide a name and description for your app. You can also upload a custom logo and select a theme color to customize the end-user interface. Click Next.

3. **Add and Organize Content:**

   Click **Add content** to select the reports from your workspace that you want to include (e.g., your "Auto-Generated Report"). Create "Sections" to organize these reports into a structured navigation menu. Click Next.

4. **Define Your Audiences:**

   In the Audience tab, specify who can access the app (Specific users/groups or the Entire Organization). Create an additional audience group and use the eye icon next to your reports to hide specific content from them.

5. **Publish and Consume:**

   Click **Publish**. Copy the generated link to share directly with users, or navigate to the main **Apps** menu on the left sidebar and click **Get App** to search for your newly published application in the organizational directory.

## 3.8 Updating & Deleting Apps

**Updating Power BI Apps**

When you edit and save changes to a Power BI report, those updates are _not_ automatically visible to the end-users consuming the published app. To push your latest report changes—or to modify app settings like themes, logos, or audience access—you must manually update the app. Since there is only one app per workspace, you manage this from the main workspace view rather than selecting a specific app item.

**Unpublishing (Deleting) an App**

Because a workspace only holds a single app, the app itself does not appear as an individual file or item in your workspace's content list. If you need to completely remove the app from circulation so that no one in the organization can search for or access it, you must use the workspace's top-level menu to unpublish it.

**Hands-On Practice: Update and Unpublish an App**

1. **Edit the Underlying Report:**

   Open a report in your workspace, enter Edit mode, make a visible change (like adding a new visual or text field), and save the report.

2. **Verify the App Remains Unchanged:**

   Navigate to your published app via the left sidebar and observe that your recent report changes are not yet visible to end-users.

3. **Update the App:**

   Return to your workspace. Click the **Update app** button (located at the top of the screen where the "Create app" button used to be). Click through the setup tabs, optionally change a setting like the theme color, and click **Update** to publish the new version.

4. **Unpublish the App:**

   Back in your workspace view, click the three dots (...) at the top of the screen and select **Unpublish app**.

5. **Verify Removal:**

   Navigate to the Apps menu on the left sidebar and click **Get App** to browse the organizational directory, confirming the app is no longer searchable or accessible.

# 4. Data Factory

## 4.1 Understanding Pipelines and Data Flow Gen2

**Understanding Data Flows**

Data Flows (Gen2) act as a low-code or no-code ETL (Extract, Transform, Load) tool powered by the Power Query editor. They are specifically focused on shaping and transforming your data before loading it into a destination like a Lakehouse or Data Warehouse. In short, data flows define your _transformation logic_.

**Understanding Data Pipelines**

While data flows handle the data shaping, Data Pipelines are designed for broader data orchestration and automation. Pipelines control the overarching execution flow—dictating when processes run, the order they run in, and how to handle dependencies or retries if a step fails. A data flow can actually be inserted as just one single activity within a larger data pipeline.

**Hands-On Practice: Initiate a Data Pipeline**

Based on the lecture introduction, here is how you can begin setting up a new data pipeline:

1. **Navigate to Your Workspace:**

   Open your Fabric environment and navigate to your designated workspace (e.g., your training workspace).

2. **Open the Item Menu:**

   Click the **New item** button to view the list of resources you can build.

3. **Create the Pipeline:** Notice the Dataflow options nearby.
   Locate and select **Data pipeline** from the creation menu to launch the orchestration canvas.

## 4.2 Copy Data Activity

<!-- Pipeline - ETL - Tranditional approach (Extract, Transform, Load)

Use Copy Assistant - Data source / Data destination / Connect -->

**Using Data Pipelines for ELT**

In Fabric, data pipelines are the primary tool for orchestrating data movement and automation. A very common pattern is to use a pipeline to perform the "EL" part of an **ELT (Extract, Load, Transform)** workflow.

- **Extract & Load:** You start by using a "Copy Data" activity to extract raw data from a source and load it directly into your Fabric destination (like a Lakehouse) without altering it.
- **Transform:** Once the raw data sits in the Lakehouse, you can use Fabric's powerful compute engines (like Spark notebooks or Data Flow Gen2) to transform the data as a separate downstream activity.

**Using the Copy Data Assistant**

When building your first pipeline, the **Copy Data Assistant** provides a guided, step-by-step wizard to quickly set up your data extraction and loading parameters.

- **Source Configuration:** You can connect to a wide variety of sources, from external cloud providers (AWS S3, Azure) to local Fabric workspaces or even provided sample datasets.
- **Destination Configuration:** You select where the data should land, such as an existing Lakehouse within your workspace.
- **Mapping:** You decide if the data should be stored as raw files or structured Delta tables. When loading into a table, you map the source columns to the destination columns and decide whether to append data to an existing table, overwrite it, or create a brand-new table.

**Hands-On Practice: Build a Copy Data Pipeline**

1. **Create a New Pipeline:**

   In your workspace, click **New item**, select **Data pipeline**, give it a descriptive name (e.g., "Data Pipeline 1"), and click Create.

2. **Launch the Copy Assistant:**

   On the empty pipeline canvas, click the **Copy Data** button to launch the guided setup wizard.

3. **Select Your Source Data:**

   In the assistant, choose your data source. For practice, select the "Sample data" option and choose the "Retail data model" (specifically the `fact_sale` table). Click Next.

4. **Set Your Destination:**

   Choose your destination type (e.g., "Workspace" -> "Lakehouse") and select the specific Lakehouse where you want the data to land.

5. **Configure the Table Mapping:**

   Choose to load the data into a **Table**. Give the destination table a new name, such as `fact_sales_raw` (indicating this is your staging table before transformations). Review the column mappings and click Next.

6. **Save and Run:**

   On the summary screen, ensure the "Start data transfer immediately" box is checked, then click **Save + Run**.

7. **Verify the Data:**

   Monitor the pipeline's status on the canvas until it says "Succeeded." Navigate to your Lakehouse view, expand the Tables folder, and verify that the `fact_sales_raw` table has been created and populated with data.

## 4.3 Adding DataFlow Gen 2 to Pipeline

**Transforming Data with Data Flow Gen2**

After completing the "Extract and Load" (EL) phase of your ELT process using a Copy Data activity, the next logical step is the "Transform" (T) phase. While you can build data flows independently, integrating one directly into your pipeline canvas allows you to orchestrate the entire workflow.

When you add a Data Flow Gen2 activity to your pipeline, you are brought into the familiar **Power Query Editor**. This low-code environment lets you apply robust data shaping techniques before loading the clean data into a final destination.

**Core Transformation Workflows**

- **Connecting to the Source:** You typically connect your data flow to the staging tables you created in the previous extraction step. In this example, the data flow connects to the raw table within your Fabric Lakehouse.
- **Applied Steps:** Every transformation you make (renaming columns, changing data types, dropping rows) is recorded as a sequential "Applied Step." These steps are visible on the right-hand panel and are executed in order every time the flow runs.
- **Adding Custom Columns:** Using the Power Query ribbon, you can compute new columns based on existing data. For example, creating a `total_amount` column by multiplying a `quantity` column by a `unit_price` column.
- **Defining the Destination:** Once the data is shaped, you define a target destination directly within the query settings. You can load this clean data into a new structured Delta table (e.g., `fact_sales_transformed`) within your Lakehouse, choosing whether future runs will **replace** or **append** the data in that table.

**Hands-On Practice: Build a Transformation Data Flow**

1. **Add the Data Flow Activity:**

   Open your existing data pipeline. From the top ribbon, click to add a **Data flow** activity to the canvas.

2. **Launch the Power Query Editor:**

   With the new activity selected, navigate to the Settings tab at the bottom. Since we haven't built one yet, click **New** to launch a fresh Data Flow Gen2 interface.

3. **Connect to Your Staging Data:** Use the staging table created in the previous exercise.
   Click **Get Data** -> **More** and search for "Lakehouse". Select your workspace and the Lakehouse containing your raw staging table. Check the box next to your raw table to load the preview.

4. **Apply a Transformation Step:**

   In the top ribbon, click **Add Column** -> **Custom Column**. Name the column "Total Amount" and use the formula builder to multiply your quantity column by your unit price column. Select the "Currency" data type and click OK.

5. **Set the Final Destination:**

   On the bottom right of the screen (under Query settings), click the **+** icon next to "Data destination". Select "Lakehouse".

6. **Configure the Destination Table:**

   Choose to create a **New table** and name it something descriptive like `fact_sales_transformed`. Choose the **Replace** option so the table updates entirely on each run, then click Save settings.

7. **Publish and Link:**

   Click **Publish** in the bottom right corner. Return to your data pipeline canvas, select the Data Flow activity, and ensure your newly published data flow is selected in the Settings dropdown.

## 4.4 Disable Staging for better performance

**Understanding Data Flow Performance**

Data Flow Gen2 can sometimes experience slow execution times, especially when processing large datasets with millions of rows and multiple transformation steps. For instance, processing 50 million rows might take several hours. If pure speed is your primary goal, using a Copy Data activity is much faster for simply moving data. For high performance transformations, using Spark notebooks within the Data Engineering experience is a highly effective alternative.

**The Staging Toggle**

If you prefer to stick with Data Flows, you can significantly improve execution speed by disabling the default staging feature. By default, Data Flows stage your data in a temporary lakehouse before performing operations. While this helps in some specific scenarios, turning it off often drastically reduces run times. In the lecture example, disabling staging cut the processing time from over three hours down to just one hour.

**Hands On Practice: Disable Staging and Verify Data**

1. **Open the Data Flow:**

   Wait until your data flow has completely finished running, then open it to access the Power Query Editor.

2. **Disable Staging:**

   In the left panel under Queries, right click your query name. Click **Enable staging** to toggle it off. You will know it is disabled when the query name changes to an italic font.

3. **Save or Publish:**

   Click **Publish later** if you do not want to run it immediately, or **Publish** to execute the flow with the new performance setting.

4. **Refresh the Lakehouse:**

   Navigate back to your Lakehouse. If you see an unidentified folder, simply refresh the Tables view to reveal your newly transformed table.

5. **Verify the Row Count:**

   Switch to the SQL analytics endpoint. Open a new query window and run a count query to confirm all rows successfully transferred.

## 4.5 Schedule Data Pipeline

**Scheduling and On Demand Execution**

Once your data pipeline is fully constructed, you do not have to rely solely on manual triggers. Fabric allows you to run pipelines on demand at any time, or automate their execution using a highly customizable scheduling tool.

**Configuration Options**

- **Run History:** You can track the status of current and past executions using the "View Run history" option.
- **Frequencies:** Schedules can be set to run by the minute, hourly, daily, or weekly.
- **Customization:** You can select specific days of the week, define multiple exact run times throughout the day, and set firm start and end dates for the schedule to remain active.

**Hands On Practice: Run and Schedule a Pipeline**

1. **Execute On Demand:**

   Open your data pipeline from the workspace. Click the **Run** button to manually trigger the pipeline immediately. You can click **View Run history** to monitor its progress.

2. **Open the Schedule Menu:**

   Click the **Schedule** button from the top ribbon (or access it by clicking the three dots next to your pipeline in the main workspace view).

3. **Configure the Frequency:**

   Toggle the schedule from "Off" to "On". Select a frequency, such as "Weekly", and check the boxes for specific days, like Monday.

4. **Set the Time and Dates:**

   Define the exact time the pipeline should execute. Ensure you specify a valid start date and end date for the schedule. Remove any blank, incomplete time slots, as they will prevent you from saving the configuration.

5. **Apply the Schedule:**

   Click **Apply** to save your settings and close the window.

## 4.6 Troubleshoot & Monitor Data Pipeline

**Troubleshooting and Monitoring Pipelines**

Fabric provides robust tools for tracking the performance, troubleshooting errors, and reviewing the history of your data pipelines. You can monitor executions at a high level across your workspace, or dive deep into the specific metrics of individual pipeline activities.

**Monitoring Tools and Views**

- **The Monitoring Hub:** Accessible via the workspace menu, this hub provides a centralized view of all active and completed runs. From here, you can also manually cancel any pipeline that is currently in progress.
- **Run History Details:** Within the pipeline itself, the Run History breaks down every single activity that occurred during execution.
- **Activity Metrics:** Clicking on a specific activity (like a Copy Data task) reveals detailed metrics, including the exact volume of data read/written, file counts, and the raw JSON input/output logs.
- **Duration Breakdown:** This provides a visual timeline of where the activity spent its execution time (e.g., time queued vs. time actively transferring data).
- **Gantt View:** If your pipeline contains multiple sequential activities, switching from the standard List View to the Gantt View provides a clear visual timeline. This helps you quickly identify which specific activity took the longest and might be causing performance bottlenecks.

**Hands On Practice: Review Run Metrics**

1. **Open the Monitoring Hub:**

   From your workspace, click the three dots next to your pipeline and select **Recent Runs**. Click into the **Monitoring Hub** to view the overarching status of your executions.

2. **Access the Run History:**

   Open your pipeline. Click on **View Run history** to see a list of your previous executions. Select a specific run that has already "Succeeded".

3. **Inspect the Activity Details:**

   In the run details, click on the **Copy Data** activity. Review the "Input" and "Output" JSON logs using the provided icons.

4. **Review the Duration Breakdown:**

   Click on the activity to open the details pane. Review the throughput statistics and hover over the duration breakdown visual to see exactly how many seconds were spent queued versus actively transferring.

5. **Switch to the Gantt View:**

   Toggle the display from List View to **Gantt View** to see a visual timeline of the activity's duration.

# 5. Data Warehouses

# 7. Real-Time Analytics

## 7.1 Basics of Real-Time Analytics in Fabric

**Understanding Real Time Analytics**

Real time analytics is crucial when insights are required the exact moment data is generated. It is essential for time sensitive decisions where even minor delays are unacceptable. Common use cases include banking fraud detection, instant user recommendations, and monitoring systems that handle high velocity data like IoT sensors.

**The Core Data Workflow**

Working with high velocity real time data generally follows a consistent process:

1. Capture the incoming data from the source devices.
2. Process the data immediately within the ecosystem.
3. Visualize and analyze the processed information.
4. Trigger automated actions based on the insights (optional).

**Fabric Components: Real Time Hub vs. Workspaces**<br/>In Microsoft Fabric, real time capabilities are organized into two distinct areas:

- **The Real Time Hub:** This is a tenant wide, centralized location designed to surface streams and tables. It acts as a discovery and exploration layer, allowing you to easily find and view active data streams. However, it does not physically store the data or the items themselves.
- **Workspaces:** This is the actual storage and management environment. Real time items (such as KQL databases, tables, and event streams) are created, managed, and edited exclusively within a workspace. If you discover a stream in the Hub and need to modify its structure, you must navigate back to the Workspace to make those changes.

**Hands On Practice: Explore the Real Time Landscape**

Since this lecture focuses on orientation, your practice involves locating these foundational areas within the interface:

1. **Locate the Real Time Hub:**

   In the Fabric navigation menu, click the Real Time icon to open the Real Time Hub. Observe that this is your tenant wide discovery center for surfacing existing streams and tables.

2. **Navigate to Your Workspace:**

   Switch to your designated Workspace. Recognize that this is the required environment where you will actually create, store, and edit your KQL databases and event streams in future steps.

## 7.2 Main Building Blocks of Real-Time Analtics in Fabric

**Eventhouses**

An Eventhouse is the real-time counterpart to a Lakehouse or Data Warehouse. While Lakehouses are optimized for batch processing and analytical data, an Eventhouse is purpose-built for rapidly loading and querying structured or unstructured streaming data. It does not store data directly; instead, it acts as an organizational container for your real-time assets.

**KQL Databases and Tables**

Inside an Eventhouse, you create **KQL (Kusto Query Language) databases**. These are column-based databases optimized specifically for high-velocity streaming, log, and time-series data (data that is constantly generated with timestamps).

- They are built purely for lightning-fast analytics and high scalability through data sharding, not for traditional transactional row modifications.
- Within these databases, **KQL tables** hold the actual data. These tables are often created automatically when event-driven data starts flowing in.

**Event Streams**

Event Streams serve as the ingestion and processing layer for your real-time data. They allow you to capture data from high-velocity sources (like Azure Event Hubs, Kafka, or IoT sensors), apply instant transformations visually without writing code, and seamlessly route that data to a destination, such as your KQL database.

**Real-Time Hub vs. Workspace**

As a reminder, the **Real-Time Hub** is a tenant-wide discovery center used to explore and preview your streams. To actually create, manage, or edit the underlying infrastructure (Eventhouses, KQL Databases, and Event Streams), you must always navigate back to your **Workspace**.

**Hands-On Practice: Create an Eventhouse and Explore Event Streams**

1. **Create an Eventhouse:**

   In your workspace, click the **New item** button and search for **Eventhouse** (found under the Store data category). Give it a descriptive name like "First_Eventhouse" and click Create.

2. **Observe the Structure:**

   Once created, notice that the Eventhouse acts as a container. Within it, you can view where your KQL databases and underlying KQL tables will be stored.

3. **Locate Event Streams:**

   Return to your workspace, click **New item**, and search for **Eventstream**. While you do not need to configure it fully yet, observe that this is the visual, no-code tool you will use to capture, transform, and route your incoming data to your Eventhouse.

## 7.3 Creating a KQL database

**Creating a KQL Database**

To begin working with real-time analytics in Fabric, you must first create a database to hold your data. Since Kusto Query Language (KQL) databases are purpose-built for time-series and log data, they must be created inside an Eventhouse (which itself lives inside your workspace).

**Data Ingestion and Storage Characteristics**

Once your KQL database is established, you can ingest data from various sources, such as local files, OneLake, Event Hubs, Eventstreams, or Data Pipelines.

- **Columnar Storage:** The KQL database uses a columnar storage model. This format provides excellent data compression (often reducing required storage to one-third of the original size) and ensures lightning-fast execution for analytical queries.

**Query Sets and Basic Navigation**

To interact with the data in your tables, you use a **Query Set**. This acts as your code editor where you can write, save, and execute KQL scripts.

- **Basic Syntax:** KQL syntax generally starts by declaring the table name, followed by a pipe character (`|`), and then the command. For example, the query `stocks | take 100` retrieves the first 100 records from the "stocks" table.
- **Execution:** You can highlight specific lines of code within your Query Set and hit "Run" to execute only that selected portion, displaying the results in the pane below.

**Hands-On Practice: Create a Database and Run a KQL Query**

1. **Navigate to Your Eventhouse:**

   In your workspace, locate and open the Eventhouse you created previously (e.g., "First_Eventhouse").

2. **Create the KQL Database:**

   Click the **+** (New database) button. Give your database a name, such as "StocksData", and confirm.

3. **Load Sample Data:**

   With your new, empty database open, click the **Get Data** button. Choose **Sample** from the options and select the "Stocks Analytics" dataset to auto-populate a table for practice.

4. **Generate a Starter Query:**

   In the left-hand explorer pane, find your newly populated `stocks` table. Click the three dots (...) next to it, select **Query table**, and choose **Show any 100 records**. This automatically opens a new Query Set.

5. **Execute the Query:**

   In the Query Set editor, highlight the code block (e.g., `stocks | take 100`) and click the **Run** button. Review the 100 returned records in the results pane at the bottom of the screen.

## 7.4 Basics of KQL

**Core KQL Concepts**

Kusto Query Language operates on a sequential flow where data is passed from one operation to the next. You always begin your query by stating the data source, which is typically your table name.

- **The Pipe Operator:** The pipe character `|` acts as a funnel. It takes the output from the previous step and passes it directly into the next operation.
- **Selecting Columns:** The `project` command allows you to specify exactly which columns you want to view. It is important to note that column names in KQL are case sensitive.
- **Limiting Results:** The `take` command followed by a number returns that specific number of rows. These rows are returned randomly and are not in any guaranteed order.

**Execution and Performance**

To run a specific query in your Query Set, highlight the lines of code and press `Shift` and `Enter` on your keyboard. While KQL is flexible and allows operations to be placed in almost any order, establishing a logical sequence is crucial for performance. Because KQL uses columnar storage, starting your query by projecting only the necessary columns or filtering the data early limits the volume of data passed to subsequent steps, resulting in much faster execution times.

**Practical Exercise: Write and Execute Basic KQL Queries**

1. **Start with the Source:**

   Open your Query Set and type the name of your table, for example `stocks`, followed by a space and a pipe character.

2. **Select Specific Columns:**

   On the next line, type `project` followed by the exact, case sensitive names of the columns you want to see separated by commas. For example type `project date, close, volume`.

3. **Limit the Output:**

   Add another pipe character on the next line and type `take 100` to limit the results to a small sample size.

4. **Execute the Query:**

   Highlight the entire block of code you just wrote and press `Shift` and `Enter` to run it and view the results in the bottom pane.

5. **Optimize the Order:**

   Try swapping the order of your operations. Place the `project` command before the `take` command. Highlight and execute again to see that the query still works, demonstrating how filtering early can be used to improve performance on larger datasets.

## 7.5 Sorting & Filtering

**Managing Query Limits**

Running a raw table query without any restrictions can trigger a limit exceeded error if the dataset is massive. It is a best practice to restrict the amount of data processed early in your sequence using commands like take or by filtering the rows.

**Filtering Data**

To filter records, use the where operator followed by a condition that evaluates to true or false. For example, writing where ticker == "LLY" will strictly return rows matching that exact ticker symbol. Placing this filter at the very beginning of your sequence greatly reduces the data load for any subsequent operations.

**Sorting Data**

You can sort your final result set using the order by operator. You specify the column you want to sort by and the direction, such as order by date desc to arrange the dates in descending order.

**Optimizing Performance**

The logical sequence of your operators directly impacts execution speed. The most efficient approach is to first filter rows with where, then select your necessary columns with project, and finally sort the remaining data with order by. Sorting at the very end ensures the engine only uses resources to organize the exact data you intend to display.

**Practice Exercise**

1. **Start the Query:**

   Open your Query Set and type your table name. Add a pipe character on the next line to begin your sequence.

2. **Filter the Rows:**

   Type where ticker == "LLY" to limit the dataset to only include records for that specific company. Add a pipe character below it.

3. **Select the Columns:**

   Type project ticker, date, close, volume to narrow down the visible columns. Add another pipe character.

4. **Sort the Results:**

   Type order by date desc to sort the filtered data chronologically from newest to oldest.

5. **Execute the Code:**

   Highlight the entire query block and press Shift and Enter to view your optimized results in the data pane below.

## 7.6 Aggregating & Grouping

**Aggregating Data**

In KQL, aggregations are handled using the summarize keyword. This command evaluates the input data and produces a calculated summary table. You define the aggregation by providing a custom alias followed by an equals sign and the function you want to use. You can also chain multiple aggregations together separated by commas.

- Count all rows: `summarize Total Records = count()`
- Calculate specific metrics: `summarize Max Close = max(close), Min Close = min(close)`

**Grouping Data with By**

To group those aggregated results into categories, you simply append the keyword by at the end of your summarize statement, followed by the specific column you want to group.

- Grouped metric: `summarize Average Close = avg(close) by ticker`

**Handling Query Limits and Time Bins**

You can group by multiple columns simultaneously by separating them with commas. However, if you group by a highly granular column like an exact date, the massive combination of rows might trigger a query limit exceeded error. To solve this, you can wrap your date column in a binning function to group the data into larger chunks, such as using week_of_year(date) to group by weeks instead of individual days. You can also assign an alias to this grouped function to keep your final table organized.

**Practice Building a Summarized Query**

1. **Begin the Query:**

   Start your query with your table name and a pipe operator.

2. **Define the Aggregation:**

   Type summarize followed by your custom alias and function, such as average close = avg(close).

3. **Add the Grouping:**

   Add the keyword by and specify the primary category, such as ticker.

4. **Add a Time Bin:**

   Add a comma after ticker and include a binned time function, such as week of year = week_of_year(date).

5. **Execute:**

   Highlight the code block and press Shift and Enter to view your cleanly grouped metrics.

## 7.7 Visulizing Data

**Visualizing Data in KQL**

A unique and powerful feature of KQL is the ability to generate visualizations directly within the query editor using the `render` operator. Instead of exporting data to an external reporting tool, you can instantly see trends and distributions on the fly.

**Chart Types and Data Shapes**

You can render various formats like a `barchart`, `columnchart`, or `piechart` simply by appending the command to the end of your aggregated query.

- **Time Charts:** Because KQL is highly optimized for time-series data, the `timechart` is particularly useful. To use it successfully, the data passed to the render command must specifically include a valid date or timestamp column alongside a numeric value to plot.

**Performance and Interface**

The engine is incredibly fast, capable of rendering tens of thousands of records in milliseconds. Once a chart is rendered, the results pane allows you to seamlessly toggle back and forth between the visual graph, the raw data table, and the query execution statistics.

**Hands-On Practice: Render a Time Chart**

1. **Filter the Data:**

   Start your query by specifying the table and filtering down to a single stock to make the chart readable. Type `stocks`, add a pipe, and type `where ticker == "AAPL"`.

2. **Select the Columns:**

   Add a pipe character on the next line and project only the date and the metric you want to track over time. Type `project date, close`.

3. **Render the Chart:**

   Add one final pipe character and type the visualization command: `render timechart`.

4. **Execute the Query:**

   Highlight the code block and press Shift and Enter. The results pane will immediately display a line graph of the closing prices over time.

5. **Toggle the Views:**

   In the results pane below the query, click the **Table** tab to view the raw numbers, then click the **Stats** tab to see exactly how quickly the visualization was generated.

## 7.8 Getting data

**Bringing Local Data into KQL**

Sometimes you need to enrich your real time data streams with static reference information, like mapping a stock ticker symbol to a full company name. In Fabric, you can quickly achieve this by uploading a local file directly into your KQL database.

**One Time Loading and Schema Setup**

When you use the Get Data feature within your database, you can choose between continuous streaming sources and one time loads. Uploading a local CSV file falls under one time loading. The import wizard allows you to create a new destination table on the fly. During this setup process, the preview window gives you the opportunity to adjust column names, modify data types, and specify that the first row is a header so it does not get imported as a standard data row.

**Hands On Practice: Ingest a Local CSV File**

1. **Open the Database and Get Data:**

   Navigate to your KQL database in your workspace. Click the **Get Data** button in the top menu to view your ingestion options.

2. **Select Local File:**

   Under the one time loading category, select **Local file**.

3. **Define the Destination:**

   Choose to create a **New table** and give it a descriptive name, such as `company`.

4. **Upload the File:**

   Drag and drop your `company_info.csv` file into the upload area and click Next to generate a data preview.

5. **Configure the Schema:**

   In the preview window, review your column names and data types. Click **Advanced** and check the box for **First row is column header** to ensure your headers are not imported as data records.

6. **Finish and Verify:**

   Click **Finish** to complete the ingestion. Open a Query Set and run `company | take 10` to verify your reference data is successfully loaded and ready to be joined with your streaming data.

## 7.9 Joining data

**Joining Data in KQL**

In Kusto Query Language, combining your high-velocity real-time data with a static reference table (like mapping a stock ticker to a company name) is straightforward using the `join` operator.

**KQL Join Syntax**
You start with your primary table and pipe it into the `join` command. You must specify the `kind` of join, wrap the secondary table in parentheses, and use the `on` keyword to specify the matching column.

```kusto
Table1
| join kind=inner (Table2) on CommonColumn

```

**Common Join Flavors**

- `kind=inner`: Returns only the rows where the join key matches in both tables. This is highly efficient but will exclude data if a match isn't found (e.g., if a stock ticker exists in your stream but not in your company reference table).
- `kind=fullouter`: Returns all rows from both tables. If a match isn't found, the missing values are filled with nulls. _Note: Using full outer joins on massive datasets without filtering can quickly trigger query limits._

**Hands-On Practice: Join and Filter Tables**

1. **Write the Inner Join:**

   Start with your active dataset, the `stocks` table. Pipe it into the join command and specify the `company` reference table you uploaded earlier: `stocks | join kind=inner (company) on ticker`.

2. **Project Selected Columns:**

   Add a pipe and project a subset of columns from both tables to keep your output clean: `project ticker, company_name, date, open, high`. Highlight and execute to see only matching records.

3. **Test an Outer Join:**

   Change your join type to `kind=fullouter`.

4. **Limit the Output:**

   Because a full outer join on this dataset is massive, add a pipe at the end of your query and type `take 20` to prevent query limit errors. Execute the query to observe how unmatched tickers display null values in the company name column.

## 7.10 Creating Eventstream

**Recapping the Real-Time Workflow**

Before moving to ingestion, it's helpful to review the core components of real-time analytics in Fabric:

- **Real-Time Hub:** The tenant-wide center for discovering and previewing your data streams.
- **Eventhouse:** The container built in your workspace that organizes your real-time items.
- **KQL Database & Tables:** The high-performance, columnar storage layer where streaming data is actually held and queried.
- **Query Sets:** The editor where you write KQL to explore and visualize your tables.

**Introduction to Eventstreams**
Up until now, we have focused on where data is stored and how to query it. But how does high-velocity, real-time data actually get into the system? This is handled by an **Eventstream**.

An Eventstream acts as the ingestion and processing engine. It allows you to:

1. **Capture** data continuously from live sources (like IoT sensors or market feeds).
2. **Transform** that data instantly as it arrives, all without writing code.
3. **Route** the processed data directly into a destination, such as your KQL database.

**Hands-On Practice: Set Up an Eventstream with Sample Data**
To understand the visual editor, you can connect an Eventstream to a live sample feed.

1. **Create the Eventstream:**
   Navigate to your workspace, click **New item**, and select **Eventstream**. Give it a clear name like "First_Eventstream" and click Create to open the visual canvas.

2. **Add a Sample Data Source:**
   On the visual canvas, you will see a prompt to connect a data source. Click **Use sample data**.

3. **Select the Stock Market Feed:**
   In the sample data menu, choose the **Stock Market** option (which provides a high data rate feed) and click **Add**.

4. **Configure the Source Container:**
   Click on your newly created data source node on the canvas. Rename it "Stock Market Data" and click Save.

5. **Preview the Live Data:**
   With the source node selected, look at the data preview pane at the bottom of the screen. You should see a live view of the time-series data flowing into your stream, timestamped within the last hour.

The data is now successfully being captured. Next, we will cover how to transform this live data and route it into your KQL database destination.

## 7.11 Routing streaming data to into KQL database

**Setting Up the Eventstream Destination**

Once your Eventstream is actively capturing data, you need to route it to a storage location. In Fabric, you can route this live data directly into the KQL database you created earlier.

During setup, you can either select an existing table or create a brand new one directly from the Eventstream visual editor. Creating a new table is often the safest approach when handling a new stream, as it automatically builds a schema that perfectly matches the incoming data format, which is typically JSON for these types of feeds.

**Publishing and Verifying Real Time Ingestion**

After configuring your destination, you must publish the Eventstream to activate the flow. Once active, the data begins streaming immediately.

While we call it real time analytics, the data technically arrives in near real time, processed in very rapid, discrete intervals. You can verify this by opening your KQL database and manually refreshing the view. You will see the row count continuously increase as new timestamped records hit the table.

**Hands On Practice: Route Data to Your Database**

1. **Add the Destination:**
   On your Eventstream canvas, click the option to add a destination and select Eventhouse.

2. **Configure the Target:**
   Select the workspace, the specific Eventhouse, and the KQL database you created previously.

3. **Create a New Table:**
   Under the table selection, choose to create a new table. Name it something distinct, like StockDataRealTime, to differentiate it from your static sample data.

4. **Publish the Stream:**
   Click Publish on the visual canvas. Wait a few moments for the success confirmation and for the stream status to turn active.

5. **Verify the Data Flow:**
   Navigate back to your Eventhouse and open your KQL database. Locate your new table, preview the data, and click the refresh button a few times to watch the live row count increase.

## 7.12 Event Processing

**Eventstream Transformations**

In your Eventstream, data does not just have to flow straight from the source to the destination; you can intercept and transform it in real-time. By editing the stream and adding a **Manage Fields** event node between your source and destination, you can perform several on-the-fly modifications without writing code.

**Key Transformation Actions**

- **Changing Data Types:** If an incoming column is formatted incorrectly (e.g., a decimal price is coming in as an integer), you can select that field and modify its data type.
- **Extracting Data:** You can use built-in functions to extract specific elements from existing fields. A common use case is extracting just the `Year` or `Month` from a complex timestamp column to create entirely new, separate columns.
- **Adding Fields:** When using the Manage Fields tool, ensure you select "Add all fields" if you want to retain your original data columns alongside your newly created ones. If you don't, the stream will only output the specific columns you actively modified.

**Handling Schema Changes**

When you add new columns (like Year and Month) to your stream, the structure (schema) of the data changes. If you route this new structure into an existing table that does not have those columns, the database will either drop the new data or result in null values. To avoid this, it is best practice to either manually update your destination table's schema in the KQL database, or simply configure the Eventstream to create a brand new destination table.

**Hands-On Practice: Add a Transformation Node**

1. **Edit the Active Stream:**
   Open your active Eventstream in the workspace. Click the **Edit** button to access the visual canvas.

2. **Break the Connection:**
   Delete the existing line connecting your Source node directly to your Destination node.

3. **Add a Manage Fields Node:**
   Add a new **Manage Fields** transform event. Connect your Source node to this new Transform node.

4. **Extract the Year:**
   Click into the Manage Fields settings. Add a new field, select your timestamp column, and use the built-in Date/Time function to extract the Year. Name the new field `Year`.

5. **Retain Original Data:**
   Crucially, ensure you also select the option to "Add all fields" so you do not lose the rest of your stock data. Save the settings.

6. **Update the Destination:**
   Connect the Transform node to your Destination node. Because the schema has changed, update the destination settings to create a new table (e.g., `StockData_Transformed`).

7. **Publish and Verify:**
   Publish the changes. Navigate back to your KQL database and view the new table to confirm the `Year` column is populating correctly in real-time.

Once you have verified the data is flowing correctly with your new transformations, you can deactivate the stream to conserve resources. Let me know if you're ready to connect this real-time database to Power BI.

## 7.13 Connecting KQL database to Power BI

## 7.14 Ingesting stream into lakehouse

## 7.15 Real-time data in notebooks

## 7.16 Retention policies

## 7.17 Deleting Eventstream
