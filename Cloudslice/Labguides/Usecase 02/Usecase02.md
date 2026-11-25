# Use Case 02: Build sales analytics with AdventureWorks dataset using Fabric data agent

**Scenario**

**Adventure Works Cycles** is a global manufacturer and seller of bicycles, components, clothing and accessories. It sells online to individual customers (Internet sales) and through a network of resellers in North America, Europe and the Pacific.

Every week the sales operations team receives dozens of ad-hoc requests from regional managers and finance: “Which product categories grew fastest last year? What were monthly and year-to-date Internet sales? Which territories depend most on resellers? How much more do repeat customers spend?” Each request means writing a new SQL query or building a one-off report, and the backlog keeps growing.

The Head of Sales Analytics, **Maria Lopez**, has asked the data team to deliver a **self-service, conversational analytics assistant** with these requirements:

- Business users ask questions in natural language and receive accurate, explainable answers backed by governed data.

- Answers must come only from approved sales tables in the company lakehouse, and respect each user's data permissions.

- Analysts must be able to teach the assistant company terminology (for example, what a "repeat purchase" means) and proven query patterns.

- Once validated, the assistant must be shared with colleagues and be reusable by other apps and agents.

You are the **data engineer** on this project. Your job is to land the AdventureWorks sales data in a Fabric lakehouse and build, tune, publish and test the **AdventureWorks Sales Agent**.

**Introduction**

Microsoft Fabric data agents let you build conversational question-and-answer experiences over your organization's data. A data agent uses generative AI to understand a question asked in plain English, choose the right data source, generate and run a query (SQL for a lakehouse or warehouse, DAX for a semantic model, or KQL for a KQL database), and return an answer that users can trace back to the query behind it.

In this lab, you help Adventure Works Cycles deliver its conversational analytics assistant. You create a Fabric workspace and a lakehouse, load the AdventureWorks sales dataset with a notebook, and create a data agent connected to the lakehouse. You then ground the agent with instructions and example SQL queries, test and refine its answers with real business questions, publish it, and finally call the published agent from a Fabric notebook through its Model Context Protocol (MCP) endpoint.

By the end of the lab, sales managers and finance users at Adventure Works can ask questions such as *"What were the total internet sales by year?"* or *"Compare internet sales and reseller sales by sales territory group"* and receive accurate, data-driven answers in seconds, without waiting for a new report.

**Objectives**

- Create a **Microsoft Fabric workspace** on a Fabric capacity.

- Create a **lakehouse** and load the AdventureWorks dataset into Delta tables by using a notebook.

- Create a **Fabric data agent** and connect it to the lakehouse tables.

- Improve the agent's responses with **agent instructions** and **example queries**.

- Test and refine the agent with business questions, and then **publish** it.

- Query the published agent programmatically through its **MCP endpoint** from a Fabric notebook.

## Solution architecture

The following diagram shows the components that you build in this lab and how data flows from the source files to the people and applications that use the data agent.

![](./media/image1.png)

## Task 1: Create a Fabric workspace

In this task, you will set up the foundational environment by creating a Fabric workspace that will host the Lakehouse, notebooks, and the Data Agent. This workspace acts as the central container for all assets used throughout the use case.

1.  Open your browser, enter the following URL in the address bar: +++https://app.fabric.microsoft.com/+++, and then press **Enter**.

    ![](./media/image2.png)

2.  In the **Microsoft Fabric** window, enter the following username, and then select **Submit**. You use the password in the next step.

    | | |
    |----|----|
    | Username | +++@lab.CloudPortalCredential(User1).Username+++ |
    | Password | +++@lab.CloudPortalCredential(User1).Password+++ |

    ![](./media/image3.png)

3.  In the **Microsoft** window, enter the password, and then select **Sign in**.

    ![](./media/image4.png)

4.  In the **Stay signed in?** window, select **Yes**.

5.  You are redirected to the **Microsoft Fabric** home page.

    ![](./media/image5.png)

6.  On the Fabric home page, select the **+ New workspace** tile.

    ![](./media/image6.png)

7.  In the **Create a workspace** pane that appears on the right, enter the following details, and then select **Apply**.

    | **Property**           | **Value**                                                         |
    |------------------------|-------------------------------------------------------------------|
    | Name                   | +++Fabric Data agent-@lab.LabInstance.Id+++ (must be a unique Id) |
    | Advanced               | Under License mode, select Fabric                                 |
    | Default storage format | Small dataset storage format                                      |
    | Template apps          | Check the Develop template apps                                   |

    ![](./media/image7.png)

    **Note:** To find your lab instance ID, select **Help** in the lab interface and copy the **ID** value.

    ![](./media/image8.png)

    ![](./media/image9.png)

8.  Wait for the workspace to be created. This takes 1–2 minutes. The new, empty workspace opens.

    ![](./media/image10.png)

## Task 2: Create the AdventureWorksLH lakehouse and load data

In this task, you create a lakehouse and populate it with the AdventureWorks tables by using a Fabric notebook. The lakehouse is the governed data foundation that the data agent queries.

1.  On the workspace page, select **+ New item** on the command bar.

    ![](./media/image11.png)

2.  In the **New item** pane, select the **Lakehouse** tile.

    ![](./media/image12.png)

3.  In the **New lakehouse** dialog box, enter +++AdventureWorksLH+++ in the **Name** field, leave **Lakehouse schemas** cleared, and then select **Create**. The new lakehouse opens.

    **Note:** Make sure there is no leading space before **AdventureWorksLH** in the name.

    ![](./media/image13.png)

    ![](./media/image14.png)

4.  When the lakehouse is ready, a **Successfully created SQL endpoint** notification appears.

    ![](./media/image15.png)

5.  On the lakehouse ribbon, select **Analyze data with** \> **Notebook** \> **New notebook**. A new notebook opens with the lakehouse attached.

    ![](./media/image16.png)

6.  Replace the code in the first cell with the following code, and then select **▷ Run cell** to the left of the cell. The code downloads each AdventureWorks Parquet file and saves it as a Delta table in the lakehouse.

    ```python
    import pandas as pd
    from tqdm.auto import tqdm
    base = "https://synapseaisolutionsa.z13.web.core.windows.net/data/AdventureWorks"

    # load list of tables
    df_tables = pd.read_csv(f"{base}/adventureworks.csv", names=["table"])

    for table in (pbar := tqdm(df_tables['table'].values)):
        pbar.set_description(f"Uploading {table} to lakehouse")

        # download
        df = pd.read_parquet(f"{base}/{table}.parquet")

        # save as lakehouse table
        spark.createDataFrame(df).write.mode('overwrite').saveAsTable(table)
    ```

    ![](./media/image17.png)

7.  Wait for the cell to finish running. A progress bar shows each table as it is uploaded; loading all tables takes about 5–7 minutes.

    ![](./media/image18.png)

8.  In the **Explorer** pane, expand **AdventureWorksLH**, select the **…** (ellipsis) next to **Tables**, and then select **Refresh**.

    ![](./media/image19.png)

9.  Expand **Tables** and verify that the AdventureWorks tables, such as **dimcustomer**, **dimdate**, **dimproduct** and **factinternetsales**, appear.

    ![](./media/image20.png)

## Task 3: Create a Fabric data agent

In this task, you create a Fabric data agent and connect it to the lakehouse. You select the dimension and fact tables that the agent needs to answer a wide range of sales analytics questions.

1.  In the left navigation pane, select your **Fabric Data agent-XXXXXX** workspace.

    ![](./media/image21.png)

2.  On the workspace page, select **+ New item**.

    ![](./media/image22.png)

3.  In the **Filter by item type** search box, enter +++data agent+++, and then select the **Data agent** tile.

    ![](./media/image23.png)

4.  In the **Create data agent** dialog box, enter +++AI-agent+++ as the name, and then select **Create**.

    ![](./media/image24.png)

    ![](./media/image25.png)

5.  On the **AI-agent** page, under **Build your data agent**, select **Add a data source**.

    ![](./media/image26.png)

6.  In the **OneLake catalog** window, select the **AdventureWorksLH** lakehouse, and then select **Add**.

    ![](./media/image27.png)

    ![](./media/image28.png)

7.  In the **Explorer** pane, expand **AdventureWorksLH** \> **Schemas** \> **dbo** \> **Tables**, and select the checkboxes for the following tables. The agent can answer questions only from the tables you select.

    - dimcustomer

    - dimdate

    - dimgeography

    - dimproduct

    - dimproductcategory

    - dimpromotion

    - dimreseller

    - dimsalesterritory

    - factinternetsales

    - factresellersales

    ![](./media/image29.png)

    ![](./media/image30.png)

## Task 4: Provide agent instructions

In this task, you add agent instructions that describe the data, define business terms and set formatting rules for answers. Instructions help the agent choose the right tables and interpret questions the way your business does.

1.  On the data agent ribbon, select **Agent instructions**. The **Agent instructions** pane opens.

    ![](./media/image31.png)

2.  Copy the following text and paste it into the **Agent instructions** box:

    ```text
    You are the AdventureWorks Sales Agent for Adventure Works Cycles.
    Answer questions about customers, products, promotions, resellers,
    sales territories and sales using the AdventureWorksLH lakehouse.

    Data source AdventureWorksLH contains:
    - dimcustomer: customer demographics and contact information
    - dimdate: calendar and fiscal date attributes
    - dimgeography: city, state/province, postal code and country/region
    - dimproduct, dimproductcategory: products and product categories
    - dimpromotion: promotions and discounts
    - dimreseller: reseller (business customer) details
    - dimsalesterritory: sales territory region, country and group
    - factinternetsales: online sales to individual customers
    - factresellersales: sales through resellers

    Business rules:
    - "Sales" or "revenue" means SUM(SalesAmount) unless stated otherwise.
    - "Internet sales" uses factinternetsales; "reseller sales" uses
      factresellersales; "total sales" combines both.
    - Use OrderDate for time-based questions unless asked otherwise.
    - A "repeat purchase" is any purchase after a customer's first purchase.
    - Show currency values rounded to 2 decimals.
    - If a question is ambiguous, state the assumption you made.
    ```

    ![](./media/image32.png)

3.  Close the **Agent instructions** pane. The instructions are saved automatically.

## Task 5: Add example queries for the lakehouse

In this task, you add example question-and-SQL pairs for the lakehouse. The agent uses these examples as patterns when it generates SQL for similar questions, which improves accuracy for complex calculations such as repeat-purchase growth and running totals.

1.  In the **Explorer** pane, select the **Setup** tab, and then select **Example queries** under **AdventureWorksLH**.

    ![](./media/image33.png)

2.  In the **Example queries** pane, select **Add example**.

    ![](./media/image34.png)

3.  In **Example query 1**, enter the following text in the **Question** box:

    +++Calculate the average percentage increase in sales amount for repeat purchases for every zipcode. Repeat purchase is a purchase subsequent to the first purchase (the average should always be computed relative to the first purchase)+++

4.  In the **SQL query** box, enter the following query:

    ```sql
    SELECT AVG((s.SalesAmount - first_purchase.SalesAmount) / first_purchase.SalesAmount * 100) AS AvgPercentageIncrease
    FROM factinternetsales s
    INNER JOIN dimcustomer c ON s.CustomerKey = c.CustomerKey
    INNER JOIN dimgeography g ON c.GeographyKey = g.GeographyKey
    INNER JOIN (
        SELECT *
        FROM (
            SELECT
                CustomerKey,
                SalesAmount,
                OrderDate,
                ROW_NUMBER() OVER (PARTITION BY CustomerKey ORDER BY OrderDate) AS RowNumber
            FROM factinternetsales
        ) AS t
        WHERE RowNumber = 1
    ) first_purchase ON s.CustomerKey = first_purchase.CustomerKey
    WHERE s.OrderDate > first_purchase.OrderDate
    GROUP BY g.PostalCode;
    ```

    ![](./media/image35.png)

5.  Select **+ Add** to add a second example, and then enter the following text in the **Question** box:

    +++Show the monthly total and year-to-date total sales. Order by year and month.+++

6.  In the **SQL query** box, enter the following query:

    ```sql
    SELECT
        Year,
        Month,
        MonthlySales,
        SUM(MonthlySales) OVER (PARTITION BY Year ORDER BY Year, Month ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS CumulativeTotal
    FROM (
        SELECT
           YEAR(OrderDate) AS Year,
           MONTH(OrderDate) AS Month,
           SUM(SalesAmount) AS MonthlySales
        FROM factinternetsales
        GROUP BY YEAR(OrderDate), MONTH(OrderDate)
    ) AS t
    ```

    ![](./media/image36.png)

    ![](./media/image37.png)

7.  Confirm that both examples show no validation errors, and then close the **Example queries** pane.

    ![](./media/image38.png)

    **Note:** Adding example question/query pairs isn't currently supported for Power BI semantic model data sources.

## Task 6: Test and refine the data agent

In this task, you act as a business user from the sales team and validate the agent's answers. You then refine the agent so that it handles an ambiguous question correctly.

Enter each of the following questions in the chat box at the bottom of the **Test the agent's responses** pane, select **Send**, and review the answer.

1.  Enter +++What were the total internet sales by year?+++ and select **Send**.

    ![](./media/image39.png)

    ![](./media/image40.png)

2.  Enter +++Show the monthly total and year-to-date total sales for 2013. Order by month.+++ and select **Send**.

    ![](./media/image41.png)

    ![](./media/image42.png)

3.  Enter +++Which 5 product categories generated the highest internet sales?+++ and select **Send**.

    ![](./media/image43.png)

4.  Enter +++Compare internet sales and reseller sales by sales territory group.+++ and select **Send**.

    ![](./media/image44.png)

    ![](./media/image45.png)

5.  Enter +++How many customers are there in each country/region?+++ and select **Send**.

6.  Enter +++What is the average percentage increase in sales amount for repeat purchases by zipcode? Show the top 10.+++ and select **Send**.

7.  For at least one answer, select **Expand response** to see the **SQL query that the agent generated** and the tables it used.

8.  Compare the SQL generated for the year-to-date question with the example query that you added in Task 5. Notice how the agent reuses the same pattern.

9.  Now test an ambiguous question. Enter +++What were sales last year?+++ and select **Send**. Observe how the agent handles the missing context.

    ![](./media/image46.png)

    ![](./media/image47.png)

10. Select **Agent instructions** and add the following rule at the end of the instructions: +++When no year is specified for "last year", use the latest full year available in dimdate that has sales.+++

    ![](./media/image48.png)

11. Ask +++What were sales last year?+++ again and compare the answer. The agent now identifies the most recent full year with sales and returns its total.

    ![](./media/image49.png)

    ![](./media/image50.png)

12. Optionally, add another example query for a question that your users ask often (for example, top resellers by sales).

    **Tip:** Treat tuning as an iterative loop: test → inspect SQL → improve instructions or examples → test again. Collect real questions from colleagues and make sure the examples reflect them.

## Task 7: Publish the data agent

When the agent's answers are accurate, publish it so that colleagues and other applications can use it.

1.  On the data agent ribbon, select **Publish**.

    ![](./media/image51.png)

2.  In the **Publish data agent** dialog box, enter +++Answers questions about Adventure Works customers, products and sales.+++ in **Description of purpose and capabilities**, and then select **Publish**.

    ![](./media/image52.png)

    ![](./media/image53.png)

3.  When the **Successfully published data agent** notification appears, select **View publishing details**. The data agent settings pane opens.

    ![](./media/image54.png)

4.  Select **Model Context Protocol** and review the **MCP server name**, **MCP server URL** and **MCP server tool**. The published agent can now be used as an MCP server.

    ![](./media/image55.png)

## Task 8: Query the agent programmatically via MCP

Developers can call a published data agent from code. In this task, you call the agent's MCP endpoint from a Fabric notebook.

**Important:** OpenAI retired the Assistants API, so applications should use the **MCP endpoint** for agent interactions. The MCP endpoint doesn't manage conversations for you – callers must keep conversation state and send relevant context across requests.

1.  Open the **AI-agent** data agent and look at the browser address bar. Copy the GUID after **/groups/** and save it in Notepad. This is your **workspace ID**.

2.  Copy the GUID after **/aiskills/** in the same URL and save it in Notepad. This is your **data agent ID**. You can also find both IDs in the **MCP server URL** that you reviewed in Task 7.

    ![](./media/image56.png)

3.  In the left navigation pane, open **Notebook 1**, the notebook that you created in Task 2.

    ![](./media/image57.png)

4.  Select **+ Add cell** to add a new code cell. Enter the following command to install the MCP Python SDK, and then run the cell:

    +++%pip install "mcp>=1.9,<2"+++

    ![](./media/image58.png)

    ![](./media/image59.png)

    ![](./media/image60.png)

5.  Select **+ Add cell**, paste the following code, replace the placeholder workspace and data agent IDs with the values that you copied, and then run the cell:

    ```python
    from mcp import ClientSession
    from mcp.client.streamable_http import streamablehttp_client

    workspace_id = "<your-workspace-id>"
    data_agent_id = "<your-data-agent-id>"
    question = "What were the total internet sales by year?"

    mcp_url = (
        f"https://api.fabric.microsoft.com/v1/mcp/workspaces/{workspace_id}"
        f"/dataagents/{data_agent_id}/agent"
    )

    def get_auth_headers():
        # Inside a Fabric notebook, get a Microsoft Entra token for the signed-in user
        token = notebookutils.credentials.getToken("pbi")
        return {"Authorization": f"Bearer {token}"}

    async def query_data_agent(question):
        headers = get_auth_headers()
        async with streamablehttp_client(mcp_url, headers=headers) as (read, write, _):
            async with ClientSession(read, write) as session:
                await session.initialize()
                # The data agent exposes a single tool. Discover it, then call it.
                tools = await session.list_tools()
                tool = tools.tools[0]
                question_arg = next(iter(tool.inputSchema["properties"]))
                result = await session.call_tool(tool.name, {question_arg: question})
                answers = [b.text for b in result.content if b.type == "text"]
                return "\n".join(answers)

    # Notebooks already run an event loop, so use top-level await
    answer = await query_data_agent(question)
    print(answer)
    ```

    ![](./media/image61.png)

    ![](./media/image62.png)

6.  Review the answer printed below the cell. To ask another question, change the **question** variable and run the cell again.

7.  When you are done, select **Stop session** on the notebook ribbon to release capacity.

    ![](./media/image63.png)

    **Note:** The Microsoft Learn sample uses an Azure Identity credential object (for example, AzureCliCredential) when running outside Fabric. Inside a Fabric notebook, authentication is handled for you; notebookutils.credentials.getToken is used here as a convenience. If you receive a 401 error, request a token for the audience https://api.fabric.microsoft.com instead.

    **Important:** When calling a data agent programmatically, add a timeout to any polling logic, keep polling frequency low (2–5 seconds), clean up resources you create, and stop the notebook session when finished to release capacity.

**Summary**

In this lab, you built a conversational sales analytics assistant for Adventure Works Cycles by using a Microsoft Fabric data agent. You:

- Created a Fabric workspace and a lakehouse, and loaded the AdventureWorks dataset into Delta tables with a PySpark notebook.

- Created the **AI-agent** data agent and limited it to the ten dimension and fact tables needed for sales analysis.

- Grounded the agent with instructions that define business terms such as "sales", "internet sales" and "repeat purchase", and with example SQL queries for complex calculations.

- Tested the agent with business questions, inspected the generated SQL, and refined the instructions so that the agent handles ambiguous questions such as "last year".

- Published the agent and reviewed its MCP server details.

- Called the published agent from a Fabric notebook through its MCP endpoint.

With this solution, Adventure Works moves from a backlog of ad-hoc report requests to governed, self-service analytics: business users get trustworthy answers in natural language, analysts control how the agent interprets the data, and developers can reuse the same agent in other apps and agents.
