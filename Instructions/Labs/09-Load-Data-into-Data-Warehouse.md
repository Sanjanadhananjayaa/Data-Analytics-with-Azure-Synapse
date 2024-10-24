# Lab 07: Load Data in Relational Data warehouse in Synapse

## Lab Scenario

In this exercise, you're going to load data into a dedicated SQL Pool. Also you will learn about data ingestion solution that loads new data into a relational data warehouse.

### Objectives

After completing this lab, you will be able to:

+ Task 1: Provision an Azure Synapse Analytics workspace
+ Task 2: Prepare to load data.
+ Task 3: Load data warehouse tables.
+ Task 4: Perform post-load optimization.


### Estimated timing: 120 minutes

### Architecture Diagram

   ![Azure portal with a cloud shell pane](./Lab-Scenario-Preview/media/lab9.png)

## Task 1: Provision an Azure Synapse Analytics workspace

You'll need an Azure Synapse Analytics workspace with access to data lake storage and a dedicated SQL pool hosting a data warehouse.

In this task, we will provision an Azure Synapse Analytics workspace using PowerShell and an ARM template, ensuring it includes access to data lake storage and a dedicated SQL pool. Then clone a repository, run a setup script, and verify the completion of provisioning.

1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new Cloud Shell in the **Azure portal**.

    ![Azure portal with a cloud shell pane](./images/DA-image1.png)

1. The first time you open the Cloud Shell, you may be prompted to choose the type of shell you want to use (Bash or PowerShell). If so, select PowerShell.

    ![Azure portal with a cloud shell pane](./images/DA-image2.png)

    > **Note**: If you have previously created a cloud shell that uses a *Bash* environment, use the the drop-down menu at the top left of the cloud shell pane to change it to ***PowerShell***.

1. On Getting started window choose **Mount storage account(1)** then under Storage account subscription select your available **subscription (2)** from the dropdown and click on **Apply (3)**.

   ![Azure portal with a cloud shell pane](./images/DA-image3.png)

1. Within the Mount storage account pane, select **we will create a storage aacount for you (1)** and click **Next (2)**.

    ![Azure portal with a cloud shell pane](./images/DA-image4.png)
    
1. Cloud Shell can be resized by dragging the separator bar at the top of the pane, or by using the —, **&#9723;**, and **X** icons at the top right of the pane to minimize, maximize, and close the pane. For more information about using the Azure Cloud Shell, see the [Azure Cloud Shell documentation](https://docs.microsoft.com/azure/cloud-shell/overview).

1. In the PowerShell pane, enter the following commands to clone this repository:

    ```powershell
    rm -r synapse -f
    git clone https://github.com/CloudLabsAI-Azure/Data-Analytics-with-Azure-Synapse synapse
    ```

1. After the repository has been cloned, enter the following commands to change to the folder for this exercise, and run the **setup.ps1** script it contains:

    ```powershell
    cd synapse/Allfiles/labs/09
    ./setup.ps1
    ```

1. If prompted, choose which subscription you want to use (this option will only happen if you have access to multiple Azure subscriptions).

1. When prompted, enter a suitable password to be set for your Azure Synapse SQL pool.

    > **Note**: Be sure to remember this password!

1. Wait for the script to complete - typically this takes around 10 minutes, but in some cases may take longer. While you're waiting, review the [Data loading strategies for dedicated SQL pool in Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/sql-data-warehouse/design-elt-data-loading) article in the Azure Synapse Analytics documentation.

## Task 2: Prepare to load data

In this task, we will open Synapse Studio, verify the status of the dedicated SQL pool, and inspect the data files in Azure Data Lake Storage. we use SQL scripts to load data from CSV files into staging tables and manage any errors encountered during the loading process.

1. After the script has completed, in the Azure portal search and select **Resource group**, go to the **analytics** resource group that it created, and select your **Synapse workspace**.

    ![Azure portal with a cloud shell pane](./images/DA-image6.png)

    ![Azure portal with a cloud shell pane](./images/DA-image7.png)

   
1. In the **Overview page** for your Synapse Workspace, in the **Open Synapse Studio** card, select **Open** to open Synapse Studio in a new browser tab; signing in if prompted.

   ![Azure portal with a cloud shell pane](./images/DA-image8.png)

1. On the left side of Synapse Studio, use the ›› icon to expand the menu - revealing the different pages within Synapse Studio that you’ll use to manage resources and perform data analytics tasks.

     ![Azure portal with a cloud shell pane](./images/DA-image9.png)


1. On the **Manage** page, on the **SQL pools** tab, select the row for the **sql*xxxxxxx*** dedicated SQL pool, which hosts the data warehouse for this exercise, and use its **&#9655;** icon to start it if its not started; confirming that you want to resume it when prompted.

   ![Azure portal with a cloud shell pane](./images/DA-image10.png)

   ![Azure portal with a cloud shell pane](./images/DA-image11.png)


   Resuming the pool can take a few minutes. You can use the **&#8635; Refresh** button to check its status periodically. The status will show as **Online** when it's ready. While you're waiting, proceed with the steps below to view the data files you will load.

    ![Azure portal with a cloud shell pane](./images/DA-image12.png)

1. On the **Data (1)** page, view the **Linked (2)** tab and verify that your workspace includes a link to your **Azure Data Lake Storage Gen2 (3)** storage account, which should have a name similar to **synapsexxxxxxx (Primary - datalakexxxxxxx) (4)**.

      ![Azure portal with a cloud shell pane](./images/DA-image13.png)

1. Expand your storage account and verify that it contains a file system container named **files (primary)**.

    ![Azure portal with a cloud shell pane](./images/DA-image14.png)

1. Select the files container, and note that it contains a folder named **data**. This folder contains the data files you're going to load into the data warehouse.

    ![Azure portal with a cloud shell pane](./images/DA-image15.png)

1. Open the **data** folder and observe that it contains .csv files of **customer** and **product** data.

      ![Azure portal with a cloud shell pane](./images/DA-image16.png)

1. Right-click any of the files and select **Preview** to see the data it contains. Note the files contain a header row, so you can select the option to display column headers and click **OK** to exist from the preview page.

     ![Azure portal with a cloud shell pane](./images/DA-image17.png)

     ![Azure portal with a cloud shell pane](./images/DA-image18.png)
   
1. Return to the **Manage** page and verify that your dedicated SQL pool is online.

## Task 3: Load data warehouse tables

Let's look at some SQL Based approaches to loading data into the Data Warehouse.

1. On the  **Data** page, select the **workspace** tab.
2. Expand **SQL Database** and select your **sql*xxxxxxx*** database. Then in its **...** menu, select **New SQL Script** >**Empty Script**.

   ![Azure portal with a cloud shell pane](./images/DA-image19.png)

### Task 3.1: Load data from a data lake by using the COPY statement
In this task, you now have a blank SQL page, which is connected to the instance. You will use this script to explore several SQL techniques that you can use to load data.

1. In your SQL script, enter the following code into the window.

    ```sql
    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```
1. On the toolbar, use the **&#9655; Run** button to run the SQL code and confirm that there are **0** rows currently in the **StageProduct** table.

   ![Azure portal with a cloud shell pane](./images/DA-image20.png)
   
1. Replace the code with the following COPY statement (changing **datalake*xxxxxx*** to the name of your data lake):

   ```sql
    COPY INTO dbo.StageProduct
    (ProductID, ProductName, ProductCategory, Color, Size, ListPrice, Discontinued)
    FROM 'https://datalakexxxxxx.blob.core.windows.net/files/data/Product.csv'
    WITH
    (
    FILE_TYPE = 'CSV',
    MAXERRORS = 0,
    IDENTITY_INSERT = 'OFF',
    FIRSTROW = 2 --Skip header row
    );
    SELECT COUNT(1) 
    FROM dbo.StageProduct
    ```

    ![Azure portal with a cloud shell pane](./images/DA-image21.png)

    >**Note** : You can find datalakexxxx value by navigating to **Data section > Linked > Azure Data Lake Storage Gen2**, please refer below screenshot.

     ![Azure portal with a cloud shell pane](./images/DA-image22.png)
  
1. Run the script and review the results. 11 rows should have been loaded into the **StageProduct** table.

    Now let's use the same technique to load another table, this time logging any errors that might occur.

1. Replace the SQL code in the script pane with the following code, changing **datalake*xxxxxx*** to the name of your data lake in both the ```FROM``` and the ```ERRORFILE``` clauses:

    ```sql
    COPY INTO dbo.StageCustomer
    (GeographyKey, CustomerAlternateKey, Title, FirstName, MiddleName, LastName, NameStyle, BirthDate, 
    MaritalStatus, Suffix, Gender, EmailAddress, YearlyIncome, TotalChildren, NumberChildrenAtHome, EnglishEducation, 
    SpanishEducation, FrenchEducation, EnglishOccupation, SpanishOccupation, FrenchOccupation, HouseOwnerFlag, 
    NumberCarsOwned, AddressLine1, AddressLine2, Phone, DateFirstPurchase, CommuteDistance)
    FROM 'https://datalakexxxxxx.dfs.core.windows.net/files/data/Customer.csv'
    WITH
    (
    FILE_TYPE = 'CSV'
    ,MAXERRORS = 5
    ,FIRSTROW = 2 -- skip header row
    ,ERRORFILE = 'https://datalakexxxxxx.dfs.core.windows.net/files/'
    );
    ```

1. Run the script and review the resulting message in **Messages** tab. The source file contains a row with invalid data, so one row is rejected. The code above specifies a maximum of **5** errors, so a single error should not have prevented the valid rows from being loaded. You can view the rows that *have* been loaded by running the following query.

    ```sql
    SELECT *
    FROM dbo.StageCustomer
    ```

1. On the **files** tab, view the root folder of your data lake and verify that a new folder named **_rejectedrows** has been created (if you don't see this folder, in the **More** menu, select **Refresh** to refresh the view).

   ![Azure portal with a cloud shell pane](./images/DA-image23.png)

1. Open the **_rejectedrows** folder and the date and time specific subfolder it contains, and note that files with names similar to ***QID147_1_1*.Error.Txt** and ***QID147_1_1*.Row.Txt** have been created. You can right-click each of these files and select **Preview** to see details of the error and the row that was rejected.

    ![Azure portal with a cloud shell pane](./images/DA-image24.png)

    ![Azure portal with a cloud shell pane](./images/DA-image25.png)

    >**Note**: The use of staging tables enables you to validate or transform data before moving or using it to append to or upsert into any existing dimension tables. The COPY statement provides a simple but high-performance technique that you can use to easily load data from files in a data lake into staging tables, and as you've seen, identify and redirect invalid rows.

### Task 3.2: Use a CREATE TABLE AS (CTAS) statement

1. Return to the script pane, and replace the code it contains with the following code:

    ```sql
    CREATE TABLE dbo.DimProduct
    WITH
    (
        DISTRIBUTION = HASH(ProductAltKey),
        CLUSTERED COLUMNSTORE INDEX
    )
    AS
    SELECT ROW_NUMBER() OVER(ORDER BY ProductID) AS ProductKey,
        ProductID AS ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.StageProduct;
    ```

2. Run the script, which creates a new table named **DimProduct**  from the staged product data that uses **ProductAltKey** as its hash distribution key and has a clustered columnstore index.

3. Use the following query to view the contents of the new **DimProduct** table:

    ```sql
    SELECT ProductKey,
        ProductAltKey,
        ProductName,
        ProductCategory,
        Color,
        Size,
        ListPrice,
        Discontinued
    FROM dbo.DimProduct;
    ```

    The CREATE TABLE AS SELECT (CTAS) expression has various uses, which include:

    - Redistributing the hash key of a table to align with other tables for better query performance.
    - Assigning a surrogate key to a staging table based upon existing values after performing a delta analysis.
    - Creating aggregate tables quickly for report purposes.

### Task 3.3: Combine INSERT and UPDATE statements to load a slowly changing dimension table

The **DimCustomer** table supports type 1 and type 2 slowly changing dimensions (SCDs), where type 1 changes result in an in-place update to an existing row, and type 2 changes result in a new row to indicate the latest version of a particular dimension entity instance. Loading this table requires a combination of INSERT statements (to load new customers) and UPDATE statements (to apply type 1 or type 2 changes).

1. In the query pane, replace the existing SQL code with the following code:

    ```sql
    INSERT INTO dbo.DimCustomer ([GeographyKey],[CustomerAlternateKey],[Title],[FirstName],[MiddleName],[LastName],[NameStyle],[BirthDate],[MaritalStatus],
    [Suffix],[Gender],[EmailAddress],[YearlyIncome],[TotalChildren],[NumberChildrenAtHome],[EnglishEducation],[SpanishEducation],[FrenchEducation],
    [EnglishOccupation],[SpanishOccupation],[FrenchOccupation],[HouseOwnerFlag],[NumberCarsOwned],[AddressLine1],[AddressLine2],[Phone],
    [DateFirstPurchase],[CommuteDistance])
    SELECT *
    FROM dbo.StageCustomer AS stg
    WHERE NOT EXISTS
        (SELECT * FROM dbo.DimCustomer AS dim
        WHERE dim.CustomerAlternateKey = stg.CustomerAlternateKey);

    -- Type 1 updates (change name, email, or phone in place)
    UPDATE dbo.DimCustomer
    SET LastName = stg.LastName,
        EmailAddress = stg.EmailAddress,
        Phone = stg.Phone
    FROM DimCustomer dim inner join StageCustomer stg
    ON dim.CustomerAlternateKey = stg.CustomerAlternateKey
    WHERE dim.LastName <> stg.LastName OR dim.EmailAddress <> stg.EmailAddress OR dim.Phone <> stg.Phone

    -- Type 2 updates (address changes triggers new entry)
    INSERT INTO dbo.DimCustomer
    SELECT stg.GeographyKey,stg.CustomerAlternateKey,stg.Title,stg.FirstName,stg.MiddleName,stg.LastName,stg.NameStyle,stg.BirthDate,stg.MaritalStatus,
    stg.Suffix,stg.Gender,stg.EmailAddress,stg.YearlyIncome,stg.TotalChildren,stg.NumberChildrenAtHome,stg.EnglishEducation,stg.SpanishEducation,stg.FrenchEducation,
    stg.EnglishOccupation,stg.SpanishOccupation,stg.FrenchOccupation,stg.HouseOwnerFlag,stg.NumberCarsOwned,stg.AddressLine1,stg.AddressLine2,stg.Phone,
    stg.DateFirstPurchase,stg.CommuteDistance
    FROM dbo.StageCustomer AS stg
    JOIN dbo.DimCustomer AS dim
    ON stg.CustomerAlternateKey = dim.CustomerAlternateKey
    AND stg.AddressLine1 <> dim.AddressLine1;
    ```

2. Run the script to load new data into the data warehouse.

## Task 4: Perform post-load optimization

After loading new data into the data warehouse, it's recommended to rebuild the table indexes and update statistics on commonly queried columns.

1. Replace the code in the script pane with the following code:

    ```sql
    ALTER INDEX ALL ON dbo.DimProduct REBUILD;
    ```

2. Run the script to rebuild the indexes on the **DimProduct** table.
3. Replace the code in the script pane with the following code:

    ```sql
    CREATE STATISTICS customergeo_stats
    ON dbo.DimCustomer (GeographyKey);
    ```

4. Run the script to create or update statistics on the **GeographyKey** column of the **DimCustomer** table.

   <validation step="948a1e4f-5d5d-465c-9ed4-e477c27d9c0f" />

  **Congratulations** on completing the lab! Now, it's time to validate it. Here are the steps:

  > - Navigate to the Lab Validation tab, from the upper right corner in the lab guide section.
  > - Hit the Validate button for the corresponding task. If you receive a success message, you have successfully validated the lab. 
  > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
  > - If you need any assistance, please contact us at labs-support@spektrasystems.com.

## Summary

In this lab, you provisioned an Azure Synapse Analytics workspace and set up a dedicated SQL pool. You learned to load data into staging tables using SQL scripts and manage errors during the process. You explored data ingestion techniques, including the COPY statement and CTAS operations, and handled slowly changing dimensions. Finally, you optimized the data warehouse by rebuilding indexes and updating statistics.

## Review

In this lab, you have accomplished the following:
- Provision an Azure Synapse Analytics workspace
- Prepare to load data
- Load data warehouse tables
- Perform post-load optimization

## You have successfully completed the lab.
