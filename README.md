# Use-an-Apache-Spark-notebook-in-a-pipeline
# Use an Apache Spark Notebook in an Azure Synapse Analytics Pipeline

This document outlines the steps to create an Azure Synapse Analytics pipeline that includes an activity to run an Apache Spark notebook for data transformation.

## Prerequisites

* An Azure subscription with administrative-level access.
* An Azure Synapse Analytics workspace provisioned with access to data lake storage and a Spark pool.

## Steps

### 1. Provision an Azure Synapse Analytics Workspace

If you haven't already, provision an Azure Synapse Analytics workspace using the provided PowerShell script and ARM template.

1.  Sign in to the [Azure portal](https://portal.azure.com).
2.  Open **Cloud Shell** by clicking the **[\>_]** button. Select **PowerShell** if prompted.
3.  Clone the repository:
    ```powershell
    rm -r dp-203 -f
    git clone [https://github.com/MicrosoftLearning/dp-203-azure-data-engineer](https://github.com/MicrosoftLearning/dp-203-azure-data-engineer) dp-203
    ```
4.  Navigate to the lab directory and run the setup script:
    ```powershell
    cd dp-203/Allfiles/labs/11
    ./setup.ps1
    ```
5.  If prompted, choose your subscription and enter a password for the SQL pool.
6.  Wait for the script to complete (approximately 10 minutes).

### 2. Run a Spark Notebook Interactively

1.  After the script completes, go to the `dp203-xxxxxxx` resource group in the Azure portal and select your Synapse workspace.
2.  Open **Synapse Studio**.
3.  Navigate to the **Data** page and verify the linked service to your Azure Data Lake Storage Gen2 account (`synapsexxxxxxx (Primary - datalakexxxxxxx)`) and the **files (primary)** container containing the **data** folder with CSV files. Preview the data to understand its structure.
4.  **Download the Spark Notebook:** Download the `Spark Transform.ipynb` file from `Allfiles/labs/11/notebooks`. You can do this by:
    * Copying the text content and saving it as `Spark Transform.ipynb` with "All Files" type.
    * Downloading it directly from the GitHub repository.
5.  **Import the Notebook:**
    * Go to the **Develop** page in Synapse Studio.
    * Expand **Notebooks** and click the **+ Import** option.
    * Select the `Spark Transform.ipynb` file you downloaded.
6.  **Attach and Run the Notebook:**
    * Open the imported **Spark Transform** notebook.
    * Attach the notebook to your **sparkxxxxxxx** Spark pool.
    * Review the notebook's notes and code cells. Note that it:
        * Sets a variable for a unique folder name.
        * Loads CSV sales order data from the `/data` folder.
        * Transforms the data by splitting the customer name.
        * Saves the transformed data in Parquet format in the unique folder.
    * Click the **▷ Run All** button to execute all cells. The first run might take longer as the Spark pool starts.
7.  **Verify Transformed Data:**
    * After execution, note the generated unique folder name in the `files` container on the **Data** page. Refresh if needed.
    * Navigate to the root of the `files` container.
    * Select the uniquely named folder. In the **New SQL Script** menu, choose **Select TOP 100 rows**.
    * In the **Select TOP 100 rows** pane, set the file type to **Parquet format** and apply.
    * Run the generated SQL script to verify the transformed data.

### 3. Run the Notebook in a Pipeline

1.  **Create a Parameters Cell:**
    * Return to the **Spark Transform** notebook in Synapse Studio.
    * In the toolbar, under the **...** menu, select **Clear output**.
    * Select the first code cell (setting `folderName`).
    * In the cell's **...** menu (top right), select **[@] Toggle parameter cell**. Verify "parameters" appears at the bottom right.
    * Click **Publish** to save changes.
2.  **Create a Pipeline:**
    * Go to the **Integrate** page.
    * Click **+** and select **Pipeline**.
    * In the **Properties** pane, name the pipeline `Transform Sales Data` and hide the properties.
3.  **Add a Notebook Activity:**
    * In the **Activities** pane, expand **Synapse** and drag a **Notebook** activity onto the pipeline canvas.
    * In the **General** tab, name it `Run Spark Transform`.
    * In the **Settings** tab:
        * **Notebook**: Select the **Spark Transform** notebook.
        * **Base parameters**: Expand and define a parameter:
            * **Name**: `folderName`
            * **Type**: `String`
            * **Value**: Click **Add dynamic content** and set it to the **Pipeline Run ID** system variable (`@pipeline().RunId`).
        * **Spark pool**: Select your **sparkxxxxxxx** pool.
        * **Executor size**: Choose **Small (4 vCores, 28GB Memory)**.
    * Your pipeline should now show the Notebook activity configured to run your Spark notebook.

### 4. Publish and Run the Pipeline

1.  Click **Publish all** to save the pipeline and other assets.
2.  In the pipeline designer, click **Add trigger** and select **Trigger now**. Confirm by clicking **OK**.
3.  Navigate to the **Monitor** page and view the **Pipeline runs** tab to see the status of the `Transform Sales Data` pipeline.
4.  Select the pipeline run to view details and note the **Pipeline run ID** in the **Activity runs** pane.
5.  Wait for the pipeline to complete (may take 5 minutes or longer). Refresh the status as needed.
6.  Once the pipeline succeeds, go to the **Data** page, browse the **files** storage container. Verify that a new folder with the name matching the pipeline run ID has been created and contains the Parquet files with the transformed sales data.  

**Important:** After completing this lab, remember to delete the resource group `dp203-xxxxxxx` in the Azure portal to avoid incurring unnecessary costs.


