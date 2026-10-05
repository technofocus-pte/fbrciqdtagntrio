# Usecase 01: From Semantics to Insights: Leveraging Fabric IQ Ontology with Fabric Data Agents

**Scenario**

**Lakeshore Retail** is a fictional multi-region retailer that sells general merchandise, perishable goods, and frozen products (such as ice cream) through stores that are supplied by regional distribution centers.

Lakeshore Retail's data is spread across several systems:

- **Lakehouse tables** with locations (stores and distribution centers), products, suppliers, inventory positions, shipments, and refrigeration units.

- **Streaming telemetry** from the refrigeration units in each store (temperature, humidity, door status) in an eventhouse.

- A **Power BI semantic model** with sales transactions and sales measures.

The operations team needs to answer cross-domain questions quickly, for example:

*Which high-priority stores in the West have frozen products below safety stock, a recent refrigeration-temperature exception, and on-shelf availability below target? For each store, show the next inbound shipment and its expected arrival.*

Answering this today means joining inventory, telemetry, location, and shipment data by hand. In this lab, you are a Lakeshore Retail data engineer. You build an ontology named **RetailSalesOntology** that connects all these sources in business terms, enrich it with descriptions and business rules, explore it as a graph, and use the **Ontology agent** to answer the scenario question in natural language.

**Introduction**

Modern data platforms need a **business-centric semantic layer**: a shared model that describes the business in its own terms and works the same way across many data sources. The **Ontology (preview)** item in **Microsoft Fabric IQ** provides this layer. In an ontology, you define **entity types** (business concepts such as stores, products, and shipments), their **properties**, the **relationships** between them, and business **rules**, and then bind these definitions to real data in OneLake: lakehouse tables, eventhouse (streaming) tables, and Power BI semantic models.

Once the ontology is in place, people and AI agents can explore it as a **graph** and ask questions in **natural language** without knowing the underlying tables or joins.

This lab is based on the Microsoft Learn tutorial series [Ontology (preview) tutorial](https://learn.microsoft.com/fabric/iq/ontology/tutorial-0-introduction).

**Objectives**

After completing this lab, you will be able to:

- Prepare a Microsoft Fabric workspace with a lakehouse, an eventhouse, and a semantic model as data sources

- Create an **Ontology (preview)** item and define entity types, including inherited entity types

- Bind entity types to static data in OneLake tables, time series data in an eventhouse, and a Power BI semantic model

- Create relationship types that represent real business processes (for example, Store *operates* Refrigeration Unit)

- Enrich the ontology with entity, property, and relationship metadata and business rules

- Explore and validate the ontology with canvas views, entity instances, the materialized graph, and path queries

- Ask questions in natural language with the **Ontology agent**

**Architecture**

| **Component**                                 | **Role in the lab**                                                                              |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------|
| Fabric workspace Fabric IQ Ontology\<number\> | Contains all the items for the lab                                                               |
| Lakehouse IQ_Lakehouse                        | Static data: locations, products, suppliers, inventory positions, shipments, refrigeration units |
| Eventhouse TelemetryDataEH (KQL database)     | Streaming data: RefrigerationTelemetry table                                                     |
| Semantic model SalesReport                    | Sales transactions and measures (Gross Margin %, Net Sales)                                      |
| Ontology RetailSalesOntology                  | Business entity types, relationships, metadata, and rules bound to the data                      |
| Graph model                                   | Materialized graph of the ontology for exploration and path queries                              |
| Ontology agent (preview)                      | Copilot that answers natural-language questions over the ontology                                |

**Target ontology**

| **Entity type**         | **Inherits from** | **Data source**                                                       |
|-------------------------|-------------------|-----------------------------------------------------------------------|
| Location                |                   | IQ_Lakehouse \> dimlocations                                          |
| Store                   | Location          | IQ_Lakehouse \> dimlocations                                          |
| Distribution Center     | Location          | IQ_Lakehouse \> dimlocations                                          |
| Product                 |                   | IQ_Lakehouse \> dimproducts                                           |
| Frozen Product          | Product           | IQ_Lakehouse \> dimproducts                                           |
| Perishable Product      | Product           | IQ_Lakehouse \> dimproducts                                           |
| Inventory               |                   | IQ_Lakehouse \> fact_inventory_positions                              |
| Supplier                |                   | IQ_Lakehouse \> dimsuppliers                                          |
| Shipment                |                   | IQ_Lakehouse \> factshipments                                         |
| Refrigeration Unit      |                   | IQ_Lakehouse \> dim_refrigeration_units                               |
| Refrigeration Telemetry |                   | TelemetryDataEH \> RefrigerationTelemetry                             |
| Sale                    |                   | SalesReport semantic model \> Sales (created with the Ontology agent) |

**Prerequisites**

- A workspace on a **Microsoft Fabric-enabled capacity**.

- A Fabric administrator has enabled the tenant settings for **Ontology (preview)** items, Fabric items, and Copilot / Azure OpenAI features (required for the Ontology agent).

- The lab files in **C:\LabFiles\Lab1** on the lab VM: six CSV files with the static data, **refrigeration_telemetry.csv**, and **SalesReport.pbix**.

**Note:** Ontology, the Ontology agent, and graph in Microsoft Fabric are in **preview**. Screens and labels may change slightly. AI-generated answers may be incorrect; always check important answers against the source data.

## Exercise 1: Set up the environment

In this exercise, you prepare the three data sources for the ontology: a lakehouse with static data, an eventhouse with streaming data, and a Power BI semantic model with sales data.

### Task 1: Create a Fabric workspace

1.  Open your browser, go to +++https://app.fabric.microsoft.com/+++, and sign in with your credentials.

| **Username** | **+++@lab.CloudPortalCredential(User1).Username+++** |
| **Password** | **+++@lab.CloudPortalCredential(User1).Password+++** |

2.  On the Fabric home page, select **+ New workspace**.

![](./media/image1.png)

3.  In the **Create a workspace** pane, enter the following details:

    - **Name**: +++Fabric IQ Ontology@lab.LabInstance.Id+++ (the name must be unique)

    - Expand **Advanced**.

![](./media/image2.png)

4.  Under **License mode**, select **Fabric**, and then select **Apply**.

![](./media/image3.png)

5.  The new workspace opens.

![](./media/image4.png)

### Task 2: Create a lakehouse

1.  In the workspace, select **+ New item**.

![](./media/image5.png)

2.  Filter by and select **Lakehouse**.

![](./media/image6.png)

3.  In the **New Lakehouse** dialog, enter +++IQ_Lakehouse+++ in the **Name** box, clear the **Lakehouse schemas** checkbox, and then select **Create**.

![](./media/image7.png)

**Note:** The Microsoft Learn tutorial keeps **Lakehouse schemas** selected, in which case the tables are created in the **dbo** schema. The rest of this lab works either way.

4.  The lakehouse opens. Wait for the notification **Successfully created SQL analytics endpoint**.

![](./media/image8.png)

![](./media/image9.png)

### Task 3: Load the static data into lakehouse tables

1.  On the **IQ_Lakehouse** page, under **Get data in your lakehouse**, select **Upload files**.

![](./media/image10.png)

2.  In the **Upload files** pane, select the folder icon.

![](./media/image11.png)

3.  Browse to **C:\LabFiles\Lab1**, select the six CSV files **dim_refrigeration_units**, **dimlocations**, **dimproducts**, **dimsuppliers**, **fact_inventory_positions**, and **factshipments**, and then select **Open**.

![](./media/image12.png)

**Important:** Don't upload **refrigeration_telemetry.csv** or **SalesReport.pbix**. You load them into the eventhouse and the workspace later in this exercise.

4.  Select **Upload**.

![](./media/image13.png)

5.  Verify that all six files are uploaded, and then close the **Upload files** pane by selecting **X**.

![](./media/image14.png)

6.  In the **Explorer**, select **Files** and verify that the six files are listed. Select **Refresh** if needed.

![](./media/image15.png)

7.  Hover over **dim_refrigeration_units.csv**, select **...**, and then select **Load to Tables \> New table**.

![](./media/image16.png)

![](./media/image17.png)

8.  In the **Load file to new table** dialog, keep the default table name and settings, and then select **Load**.

![](./media/image18.png)

9.  Verify that the **dim_refrigeration_units** table appears under **Tables**.

![](./media/image19.png)

**Note:** You may need to select **Refresh** more than once to see the table and preview its data.

10. Repeat steps 7-9 for each of the remaining five files. For example, for **dimlocations.csv**, select **... \> Load to Tables \> New table**, and then select **Load**.

![](./media/image20.png)

![](./media/image21.png)

11. When you're done, the lakehouse has six tables: **dim_refrigeration_units**, **dimlocations**, **dimproducts**, **dimsuppliers**, **fact_inventory_positions**, and **factshipments**. The default table names are the file names in lowercase.

![](./media/image22.png)

12. In the left navigation bar, select your workspace **Fabric IQ Ontology\<number\>**.

![](./media/image23.png)

### Task 4: Load the streaming data into an eventhouse

1.  In the workspace, select **+ New item**, and then select **Eventhouse**.

![](./media/image24.png)

2.  Enter +++TelemetryDataEH+++ as the **Eventhouse name**, and then select **Create**.

![](./media/image25.png)

3.  The eventhouse opens when it's ready. A KQL database with the same name is created automatically.

![](./media/image26.png)

4.  Under **KQL databases**, select **TelemetryDataEH** to open the database.

![](./media/image27.png)

![](./media/image28.png)

5.  On the ribbon, select **Get data \> Local file**.

![](./media/image29.png)

6.  Under **Select or create a destination table**, select **+ New table**.

![](./media/image30.png)

7.  Enter +++RefrigerationTelemetry+++ as the table name, select the check mark, and then select **Browse for files**.

![](./media/image31.png)

![](./media/image32.png)

8.  Browse to **C:\LabFiles\Lab1**, select **refrigeration_telemetry.csv**, and then select **Open**.

![](./media/image33.png)

9.  Select **Next**.

![](./media/image34.png)

10. On the **Inspect the data** page, review the columns (**TelemetryId**, **UnitId**, **Timestamp**, **TemperatureC**, and more), and then select **Finish**.

![](./media/image35.png)

11. Wait until the ingestion shows **1 blobs: 1 succeeded, 0 failed**, and then select **Close**.

![](./media/image36.png)

12. Verify that the KQL database shows the **RefrigerationTelemetry** table.

![](./media/image37.png)

13. In the left navigation bar, select your workspace.

![](./media/image38.png)

### Task 5: Upload the semantic model

1.  In the workspace, select **Import \> Report, Paginated Report or Workbook \> From this computer**.

![](./media/image39.png)

2.  Browse to **C:\LabFiles\Lab1**, select **SalesReport.pbix**, and then select **Open**.

![](./media/image40.png)

3.  Verify that the **SalesReport** report and a **SalesReport** semantic model with the same name appear in the workspace.

![](./media/image41.png)

## Exercise 2: Build the ontology

In this exercise, you create the ontology item and its **entity types**. An entity type represents a type of business object. You create its **properties** by binding them to columns in a data source. You also use **inheritance**: a child entity type (for example, *Store*) inherits all the properties of its parent (*Location*).

### Task 1: Create the ontology item

1.  In the workspace, select **+ New item**. Search for and select **Ontology (preview)**.

![](./media/image42.png)

2.  In the **New Ontology** dialog, enter +++RetailSalesOntology+++ as the **Name**, keep your workspace as the **Location**, and then select **Create**.

![](./media/image43.png)

**Tip:** Ontology names must begin with a letter and can contain only letters, numbers, and underscores. Don't use spaces or dashes.

**Important:** If Fabric can't create the ontology item, ask your administrator to verify that the required tenant settings are enabled.

3.  The ontology opens on the configuration canvas.

![](./media/image44.png)

### Task 2: Create the Location entity type

1.  On the ribbon, select **+ Add entity type**.

![](./media/image45.png)

2.  Enter +++Location+++ as the **Entity type name**, and then select **Add Entity Type**.

![](./media/image46.png)

3.  The **Location** entity type is added to the canvas and to the **Explorer**.

![](./media/image47.png)

4.  On the canvas, select **...** next to **Location**, and then select **Bind data**.

![](./media/image48.png)

5.  Under **Add a data source**, select **Add**.

![](./media/image49.png)

6.  In the **OneLake catalog**, expand **IQ_Lakehouse**, select the **dimlocations** table, and then select **Select table**.

![](./media/image50.png)

7.  Review the source. **dimlocations** is set as the **Primary source**.

![](./media/image51.png)

8.  Select **Entity type properties**.

![](./media/image52.png)

9.  The columns of the **dimlocations** table are populated as proposed properties (**LocationId**, **LocationType**, **Name**, **Region**, **City**, and more). Without making any changes, select **Create**.

![](./media/image53.png)

10. When **Entity type updated successfully** appears, select **Cancel** to close the binding page.

![](./media/image54.png)

11. The **Configure** page of the entity type opens, showing its properties and their data source.

![](./media/image55.png)

### Task 3: Create the Store entity type (inherits from Location)

1.  Select **Home** to return to the configuration canvas.

![](./media/image56.png)

2.  On the ribbon, select **+ Add entity type**.

![](./media/image57.png)

3.  Enter +++Store+++ as the **Entity type name**. Expand **Additional configuration**, set **Choose entity to inherit from (optional)** to **Location**, and then select **Add Entity Type**.

![](./media/image58.png)

4.  The **Store** entity type appears on the canvas.

![](./media/image59.png)

5.  With **Store** selected in the **Explorer**, select **View Entity Type details**.

![](./media/image60.png)

6.  On the **Configure** tab, notice that **Store** already has properties. They are inherited from **Location**, but their **Data source** is **Unbound**.

![](./media/image61.png)

7.  Select **Manage property bindings \> Add binding and properties**.

![](./media/image62.png)

8.  Select **Add**.

![](./media/image63.png)

9.  Expand **IQ_Lakehouse**, select **dimlocations**, and then select **Select table**.

![](./media/image64.png)

10. Select **Entity type properties**, verify that each property has a source column, and then select **Create**.

![](./media/image65.png)

11. When **Entity type updated successfully** appears, select **Cancel**.

![](./media/image66.png)

**Note:** **Store** and **Distribution Center** both use the **dimlocations** table. The **LocationType** column (STORE or DC) tells them apart. You document this in the entity descriptions in Exercise 4.

### Task 4: Create the remaining entity types

Create the following entity types with the same steps. For each one, bind the listed table and keep all the source columns as properties.

| **Entity type name**    | **Inherits from** | **Data source table**                     |
|-------------------------|-------------------|-------------------------------------------|
| Distribution Center     | Location          | IQ_Lakehouse \> dimlocations              |
| Product                 |                   | IQ_Lakehouse \> dimproducts               |
| Frozen Product          | Product           | IQ_Lakehouse \> dimproducts               |
| Perishable Product      | Product           | IQ_Lakehouse \> dimproducts               |
| Inventory               |                   | IQ_Lakehouse \> fact_inventory_positions  |
| Supplier                |                   | IQ_Lakehouse \> dimsuppliers              |
| Shipment                |                   | IQ_Lakehouse \> factshipments             |
| Refrigeration Unit      |                   | IQ_Lakehouse \> dim_refrigeration_units   |
| Refrigeration Telemetry |                   | TelemetryDataEH \> RefrigerationTelemetry |

The following steps show the first few as examples.

1.  **Distribution Center:** Select **Home \> + Add entity type**. Enter +++Distribution Center+++, expand **Additional configuration**, select **Location** as the parent, and select **Add Entity Type**.

![](./media/image67.png)

![](./media/image68.png)

2.  Select **...** next to **Distribution Center \> Bind data**, select **Add**, select **IQ_Lakehouse \> dimlocations**, and select **Select table**.

![](./media/image69.png)

![](./media/image70.png)

![](./media/image71.png)

3.  Select **Entity type properties \> Create**, and when **Entity type updated successfully** appears, select **Cancel**. The **Configure** page shows the bound properties.

![](./media/image72.png)

![](./media/image73.png)

![](./media/image74.png)

4.  **Product:** Select **Home \> + Add entity type**, enter +++Product+++, and select **Add Entity Type**.

![](./media/image75.png)

**Note:** Use **Product** (or a plural, *Products*) as written here. Avoid entity type names that are reserved words in GQL, the graph query language, such as **ORDER**.

5.  Select **... \> Bind data \> Add**, select **IQ_Lakehouse \> dimproducts**, select **Select table**, then **Entity type properties \> Create**, and then **Cancel**.

![](./media/image76.png)

![](./media/image77.png)

![](./media/image78.png)

6.  **Frozen Product:** Select **Home \> + Add entity type**, enter +++Frozen Product+++, set **Product** as the parent, and select **Add Entity Type**. Bind it to **IQ_Lakehouse \> dimproducts** in the same way.

![](./media/image79.png)

![](./media/image80.png)

7.  Create **Perishable Product** (parent **Product**), **Inventory**, **Supplier**, **Shipment**, and **Refrigeration Unit** in the same way, using the tables in the table above.

8.  **Refrigeration Telemetry:** Select **Home \> + Add entity type**, enter +++Refrigeration Telemetry+++, and select **Add Entity Type**.

![](./media/image81.png)

9.  Select **... \> Bind data \> Add**. In the **OneLake catalog**, expand **TelemetryDataEH \> TelemetryDataEH** (KQL database), select the **RefrigerationTelemetry** table, and select **Select table**.

![](./media/image82.png)

**Note:** This entity type uses the **eventhouse** table, not a lakehouse table, as its data source.

10. Select **Entity type properties**, review the properties (**TelemetryId**, **UnitId**, **Timestamp**, **TemperatureC**, **HumidityPct**, **DoorOpen**, and more), select **Create**, and then **Cancel**.

![](./media/image83.png)

11. **Checkpoint:** Select **Home**. The **Explorer** lists 11 entity types: **Location**, **Store**, **Distribution Center**, **Product**, **Frozen Product**, **Perishable Product**, **Inventory**, **Supplier**, **Shipment**, **Refrigeration Unit**, and **Refrigeration Telemetry**.

![](./media/image84.png)

### Task 5: Add the Sale entity type with the Ontology agent

The **Ontology agent** can also *build* the ontology. In this task, you use it to create a **Sale** entity type bound to the **SalesReport** semantic model.

1.  On the ribbon, select **Ontology agent**. The agent opens in a pane on the right, in **Plan** mode.

![](./media/image85.png)

**Note:** In **Plan** mode, the agent can query and explore the ontology and propose changes, but it doesn't change anything. In **Act** mode, it applies changes.

2.  In the **Say something** box, enter the following prompt and select **Send**:

    +++Create a new entity type called Sale, bound to data in the Sales table from the SalesReport semantic model inside this workspace.+++

![](./media/image86.png)

3.  The agent creates a plan for the new entity type. Review the **Enrichment added** summary.

![](./media/image87.png)

4.  Switch the toggle to **Act**, enter +++Apply the entity type plan+++, and then select **Send**.

![](./media/image88.png)

5.  Wait while the agent generates the ontology changes.

![](./media/image89.png)

6.  The agent adds the **Sale** entity type to the canvas and reports the result.

![](./media/image90.png)

**Note:** The agent may report that the apply was *partial* because of conflicts with other draft definitions. As long as **Sale exists now**, you can continue.

7.  Select **Sale** in the **Explorer**, and then select **View Entity Type details**.

![](./media/image91.png)

8.  Verify that the properties (**Channel**, **CostAmount**, **NetSalesAmount**, **ProductId**, **SaleId**, and more) are bound to the **Sales** table of the semantic model.

![](./media/image92.png)

9.  Scroll down to **Metrics** and verify that two metrics, **Gross Margin %** and **Net Sales**, were added from the DAX measures of the semantic model.

![](./media/image93.png)

## Exercise 3: Create relationship types

A **relationship type** connects an origin entity type to a target entity type. In this lab, you link them by properties: the **origin property** and the **target property** must hold the same values.

### Task 1: Create Store operates Refrigeration Unit

1.  On the canvas, select **...** next to an entity type and select **Add relationship type** (or select **Add relationship** on the ribbon).

![](./media/image94.png)

2.  Enter the following details, and then select **Create**:

    - **Relationship type name**: +++operates+++

    - **Origin entity type**: **Store**

    - **Target entity type**: **Refrigeration Unit**

![](./media/image95.png)

3.  The relationship appears on the canvas between **Store** and **Refrigeration Unit**.

![](./media/image96.png)

4.  Select the **operates** relationship to open its configuration.

![](./media/image97.png)

5.  The page has three sections: the **Origin entity type** (Store), the **Relationship** (operates), and the **Target entity type** (Refrigeration Unit). Leave **Use mapping table?** set to **Off**.

![](./media/image98.png)

6.  Under the origin entity type, in **Property**, select **LocationId**. Under the target entity type, in **Property**, select **StoreId**. Select **Save**.

![](./media/image99.png)

**Note:** These settings mean that **LocationId** on a Store matches **StoreId** on a Refrigeration Unit.

7.  When **Successfully updated the relationship type** appears, select **Cancel**.

![](./media/image100.png)

8.  The **Configure** page of **Store** shows the new relationship in the **Relationships** section.

![](./media/image101.png)

### Task 2: Create the remaining relationship types

1.  Select **Home**, and then select **Add relationship**.

![](./media/image102.png)

2.  Create **deliversTo**: origin **Shipment**, target **Store**, and select **Create**.

![](./media/image103.png)

![](./media/image104.png)

3.  Select the **deliversTo** relationship, set the origin **Property** to **ToStoreId** and the target **Property** to **LocationId**, select **Save**, and then **Cancel**.

![](./media/image105.png)

4.  Create the remaining relationship types in the same way. The table lists all the relationship types, including **operates** and **deliversTo**, which you already created:

| **Relationship type name** | **Origin entity type (Property)** | **Target entity type (Property)** |
|----------------------------|-----------------------------------|-----------------------------------|
| operates                   | Store (LocationId)                | Refrigeration Unit (StoreId)      |
| deliversTo                 | Shipment (ToStoreId)              | Store (LocationId)                |
| occursAt                   | Sale (StoreId)                    | Store (LocationId)                |
| stockedAt                  | Inventory (StoreId)               | Store (LocationId)                |
| originatesAt               | Shipment (FromLocationId)         | Distribution Center (LocationId)  |
| forProduct                 | Sale (ProductId)                  | Product (ProductId)               |
| stockedAt                  | Product (ProductId)               | Inventory (ProductId)             |
| suppliedBy                 | Product (SupplierId)              | Supplier (SupplierId)             |
| contains                   | Shipment (ProductId)              | Product (ProductId)               |
| hasTelemetryReading        | Refrigeration Unit (UnitId)       | Refrigeration Telemetry (UnitId)  |

5.  **Checkpoint:** Select **Home**. The canvas shows the relationships connecting the entity types.

![](./media/image106.png)

**Important:** Select the correct properties. If the origin and target properties hold different values, the relationship is created but finds no matches.

## Exercise 4: Enrich the ontology

Metadata and rules give AI agents the business context they need to interpret the data correctly. In this exercise, you add descriptions to entity types, properties, and relationship types, and you define business rules.

### Task 1: Add entity type metadata

1.  Select **Store** in the **Explorer**, and then select **View Entity Type details**.

![](./media/image107.png)

2.  Scroll to **Entity metadata**. In **Description**, enter the following text, and then select **Update**:

    +++Filtered locations binding where LocationType = STORE; priorityDefinition=Tier 1 stores require same-day response.+++

![](./media/image108.png)

3.  Verify the description, and then select **Home**.

![](./media/image109.png)

4.  Add descriptions to the following entity types in the same way (select the entity type, select **View Entity Type details**, enter the **Description**, and select **Update**):

| **Entity type**     | **Description**                                               |
|---------------------|---------------------------------------------------------------|
| Distribution Center | +++Filtered locations binding where LocationType = DC.+++     |
| Frozen Product      | +++Filtered items binding where StorageClass = FROZEN.+++     |
| Perishable Product  | +++Filtered items binding where StorageClass = PERISHABLE.+++ |

![](./media/image110.png)

![](./media/image111.png)

![](./media/image112.png)

![](./media/image113.png)

![](./media/image114.png)

5.  Open **Inventory**. Under **Additional metadata**, select **+** and add the following key-value pairs, and then select **Update**:

    - **On-Shelf Availability %**: +++On-shelf availability percentage is calculated by dividing the total ShelfAvailableUnits by the total ShelfCapacityUnits. If total ShelfCapacityUnits is zero, the result is blank to avoid division by zero.+++

    - **Low On-Shelf Availability**: +++On-Shelf Availability % less than 95+++

![](./media/image115.png)

6.  Open **Sale**. Review the **Description** and **Synonyms** that were added from the semantic model. No changes are needed.

![](./media/image116.png)

### Task 2: Add property metadata

1.  In the **Explorer**, select **...** next to **Product**, and then select **Bind data**.

![](./media/image117.png)

2.  Select **Entity type properties**.

![](./media/image118.png)

3.  In the **ProductId** row, select **Metadata**.

![](./media/image119.png)

4.  In **Description**, enter +++Enterprise product identifier; not a supplier SKU or UPC.+++ and select **Update**.

![](./media/image120.png)

5.  Select **Save**, and when **Entity type updated successfully** appears, select **Cancel**.

![](./media/image121.png)

![](./media/image122.png)

6.  Repeat for **Refrigeration Telemetry**: in the **TemperatureC** row, select **Metadata**, enter +++Temperature in Celsius+++, select **Update**, and then **Save**.

![](./media/image123.png)

![](./media/image124.png)

![](./media/image125.png)

7.  Repeat for **Inventory**: in the **InventoryStatus** row, select **Metadata**, enter +++0 is AT_RISK and 1 is HEALTHY.+++, select **Update**, and then **Save**.

![](./media/image126.png)

![](./media/image127.png)

### Task 3: Add relationship metadata

1.  Select **Home**, select **Store**, and on the canvas select the **operates** relationship.

![](./media/image128.png)

![](./media/image129.png)

2.  In the **Metadata** section, select **Edit**.

![](./media/image130.png)

3.  In **Description**, enter +++Identifies the refrigeration equipment operating in a store.+++ and select **Update**.

![](./media/image131.png)

4.  Select **Save**, and when **Successfully updated the relationship type** appears, select **Cancel**.

![](./media/image132.png)

![](./media/image133.png)

5.  Add descriptions to the other relationship types in the same way:

| **Relationship**                                                    | **Description**                                                                                         |
|---------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| deliversTo (Shipment \> Store)                                      | Identifies the store receiving the shipment.                                                            |
| occursAt (Sale \> Store)                                            | Identifies where the sale occurred.                                                                     |
| stockedAt (Inventory \> Store)                                      | Represents an active store assortment and its current inventory position, not merely a historical sale. |
| originatesAt (Shipment \> Distribution Center)                      | Identifies the distribution center sending the shipment.                                                |
| forProduct (Sale \> Product)                                        | Identifies the item sold.                                                                               |
| stockedAt (Product \> Inventory)                                    | The item is part of the store's active assortment and has a current inventory position there.           |
| suppliedBy (Product \> Supplier)                                    | Identifies the supplier responsible for the items.                                                      |
| contains (Shipment \> Product)                                      | Identifies the item being replenished.                                                                  |
| hasTelemetryReading (Refrigeration Unit \> Refrigeration Telemetry) | Identifies the sensor telemetry readings for refrigeration units.                                           |

### Task 4: Add business rules

**Rules** describe in natural language what must, must not, or should be true in the business. Words that match entity types are linked to the ontology.

1.  Select **Home**, and in the **Explorer** select **Rules**.

![](./media/image134.png)

2.  Select **+ Create rule**.

![](./media/image135.png)

3.  Enter +++Cold-chain exception+++ as the **Rule name**, and select **Create**.

![](./media/image136.png)

4.  In **Rule definition**, enter the following text. Under **Linked ontology concepts**, select **Add concept** and add **Frozen Product** and **Refrigeration Unit**. Select **Save**.

    +++A frozen product has a cold-chain exception when the temperature of the refrigeration unit storing it remains above the product's maximum storage temperature for more than 20 minutes.+++

![](./media/image137.png)

5.  When **Rule saved** appears, select **Cancel**.

![](./media/image138.png)

6.  Select **New rule**, enter +++Inventory at risk+++, and select **Create**.

![](./media/image139.png)

![](./media/image140.png)

7.  Enter the definition +++A store inventory position is at risk when projected on-hand inventory falls below safety stock before the next scheduled delivery+++, link **Store** and **Inventory**, select **Save**, and then **Cancel**.

![](./media/image141.png)

8.  Select **New rule**, enter +++Late replenishment+++, and select **Create**.

![](./media/image142.png)

9.  Enter the definition +++A replenishment shipment is late when its estimated arrival is more than four hours after its scheduled arrival.+++, link **Shipment**, select **Save**, and then **Cancel**.

![](./media/image143.png)

10. Verify that the **Business rules** list shows the three rules.

![](./media/image144.png)

## Exercise 5: Explore the ontology

### Task 1: Explore the canvas views and entity instances

1.  Select **Home** and select **Product**. On the top right of the canvas, select **Full ontology** to see all entity types and relationships.

![](./media/image145.png)

2.  Select **Lineage** to see the inheritance structure: **Product** has two derived types, **Frozen Product** and **Perishable Product**.

![](./media/image146.png)

3.  Select **Relationship** to see only the relationships of the selected entity type.

![](./media/image147.png)

4.  With **Product** selected, select **View Entity Type details**.

![](./media/image148.png)

5.  Select the **Instances** tab. The list shows the product records populated from the **dimproducts** table.

![](./media/image149.png)

### Task 2: Define entity type keys

An **entity type key** uniquely identifies each record of an entity type. All the entity types in the graph need a key.

1.  Select **Location** and select **View Entity Type details**.

![](./media/image150.png)

2.  Next to **Entity type key**, select **Define entity type key**.

![](./media/image151.png)

3.  Select **LocationId**, and then select **Save**.

![](./media/image152.png)

4.  Verify that the **Entity type key** shows **LocationId**.

![](./media/image153.png)

5.  Repeat for each of the other entity types. For example, for **Store**, select **LocationId**.

![](./media/image154.png)

| **Entity type**                             | **Key**                          |
|---------------------------------------------|----------------------------------|
| Location, Store, Distribution Center        | LocationId                       |
| Product, Frozen Product, Perishable Product | ProductId                        |
| Inventory                                   | the unique ID column of **fact_inventory_positions** |
| Supplier                                    | SupplierId                       |
| Shipment                                    | the unique ID column of **factshipments** |
| Refrigeration Unit                          | UnitId                           |

**Note:** Skip **Sale** and **Refrigeration Telemetry**; they aren't included in the graph in this lab.

### Task 3: Materialize the graph

1.  Select **Home**, and then select **Manage graph** on the ribbon.

![](./media/image155.png)

2.  The **Configure Graph** page opens with **Use the entire Ontology** set to **Yes**. All entity types are selected except **Sale** and **Refrigeration_Telemetry** (shown in gray in the preview).

![](./media/image156.png)

![](./media/image157.png)

![](./media/image158.png)

**Important:** Materializing the **entire** ontology can take **more than one hour**. To finish the lab in time, build the graph from a smaller set of entity types, as described in the next step. A smaller graph is created in a few minutes and is enough for the exploration in this lab.

3.  Set **Use the entire Ontology** to **No**. In the **Entities** list, select only **Store**, **Product**, **Shipment**, and **Supplier**, and clear all the others. Verify that only these entity types are highlighted in blue in the **Preview**, and then select **Continue**.

![](./media/image162.png)

**Note:** The screenshot shows the selection being made. Make sure that **Shipment** and **Supplier** are also selected, because the next task uses the relationships between **Shipment**, **Store**, and **Product** and runs a path query from **Store** to **Supplier**. Relationships are included only when both of their entity types are selected.

**Tip:** If you have more time, you can keep **Use the entire Ontology** set to **Yes** to materialize all entity types except **Sale** and **Refrigeration_Telemetry**. Expect this to take more than an hour.

4.  On the **Projection summary** page, review the selected entities, and then select **Materialize**.

![](./media/image159.png)

5.  A notification **Creating graph model** appears. Wait until the graph model is created.

![](./media/image160.png)

![](./media/image161.png)

### Task 4: Explore the graph

1.  On the ribbon, select **Explore graph**. This button is available after the graph is materialized.

![](./media/image163.png)

2.  The graph queryset opens. On the right side, select the puzzle piece icon to expand the **Components** pane.

![](./media/image164.png)

![](./media/image165.png)

3.  The **Components** pane lists the **Nodes** and **Edges** available in the graph.

![](./media/image166.png)

4.  Under **Nodes**, select **Store**, **Product**, and **Shipment**. Under **Edges**, select **Shipment_Store** and **Shipment_Product**. The nodes and edges are added to the canvas.

![](./media/image167.png)

5.  On the ribbon, select **Path query** and confirm **Switch** when prompted. Enter the following details to find the paths within two hops between the Seattle store and the supplier Cascade Fresh Products, and then select **Run**:

    - **Start node**: **Store**

    - **End node**: **Supplier**

    - **Filter start node**: **City** = +++Seattle+++

    - **Filter end node**: **SupplierName** = +++Cascade Fresh Products+++

    - **Max hops**: **2**

![](./media/image168.png)

6.  Review the result in the query canvas and the **Results** pane.

![](./media/image169.png)

## Exercise 6: Ask questions with the Ontology agent

The **Ontology agent (preview)** is an AI-powered Copilot that helps you build and use ontologies in natural language. It reasons over the ontology, including its metadata and rules, and queries the bound data sources to answer questions.

### Task 1: Ask questions in natural language

1.  Select **Home**, and on the ribbon select **Ontology agent**.

![](./media/image170.png)

2.  The agent opens in **Plan** mode in a pane on the right.

![](./media/image171.png)

3.  Enter +++Which frozen products are stocked at Tier 1 stores in the West?+++ and select **Send**.

![](./media/image172.png)

4.  The agent reasons for a short time and then answers from your ontology. Expand **Reasoning** to see how it reached the answer.

![](./media/image173.png)

![](./media/image174.png)

5.  Enter +++Which of those inventory positions are below safety stock?+++ and select **Send**.

![](./media/image175.png)

6.  Review the answer, which lists the inventory positions with their on-hand quantity, safety stock, and shortfall.

![](./media/image176.png)

7.  (Optional) Explore the ontology further with these questions, or ask your own:

    - +++Which affected stores had a qualifying refrigeration-temperature exception?+++

    - +++What inbound shipments are expected for the affected products and stores?+++

    - +++What is the on-shelf availability of frozen products for each store?+++

    - +++Which of these stores have low on-shelf availability?+++

    - +++Which of these stores have declining gross margin?+++

### Task 2: Answer the scenario question

1.  Enter the main question of the Lakeshore Retail scenario and select **Send**:

    +++Which high-priority stores in the West have frozen products below safety stock, a recent refrigeration-temperature exception, and on-shelf availability below target? For each store, show the next inbound shipment and its expected arrival.+++

![](./media/image177.png)

2.  Review the answer. The agent combines inventory, refrigeration telemetry, store, and shipment data through the ontology, applies the business rules and metadata (for example, the on-shelf availability target), and lists the qualifying stores with their next inbound shipment.

![](./media/image178.png)

![](./media/image179.png)

**Note:** Answers can differ between runs, because the agent decides how to interpret terms such as *recent* and *target*. The agent states its assumptions in the answer. Check important answers against the source data.

## Exercise 7: Clean up resources

1.  In the left navigation bar, select your workspace **Fabric IQ Ontology\<number\>**, and then select **Workspace settings**.

![](./media/image180.png)

2.  On the **General** tab, scroll to **Delete workspace**, and then select **Remove this workspace**. Confirm the deletion.

![](./media/image181.png)

3.  Verify the notification **Workspace deleted**.

![](./media/image182.png)

**Summary**

In this lab, you used **Microsoft Fabric IQ Ontology (preview)** to build a connected, business-friendly model of Lakeshore Retail's operations. You prepared a lakehouse with static data, an eventhouse with refrigeration telemetry, and a Power BI semantic model with sales data. You created the **RetailSalesOntology** with entity types and inheritance (Location \> Store and Distribution Center, Product \> Frozen and Perishable Product), bound them to all three data sources, and used the Ontology agent to add a **Sale** entity type from the semantic model. You connected the entity types with relationship types, enriched the model with entity, property, and relationship metadata and business rules, explored it through canvas views, instances, a materialized graph, and path queries, and finally used the **Ontology agent** to answer the cross-domain scenario question in natural language, without writing a single join. This shows how an ontology bridges operational and analytical data and gives people and AI agents a shared understanding of the business.
