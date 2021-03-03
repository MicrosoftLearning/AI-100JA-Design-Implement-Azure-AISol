## How To Run Finished Project.

1. Before you start you must complete [Lab 1: Meeting the Technical Requirements](../Lab1-Technical_Requirements/02-Technical_Requirements.md). As result you should have Cosmos DB, Storage Account and Cognitive Service (General or Vision) deployed in your Azure subscription.

1. Modify `settings.json` in the `TestCLI` folder and provide following settings: 

    ```JSON
    {
        "CognitiveServicesKeys": {
            "Url": "https://eastus.api.cognitive.microsoft.com/vision/v1.0",
            "Key": "01234567890"
        },
        "AzureStorage": {
            "ConnectionString": "DefaultEndpointsProtocol=https;AccountName=yourdemoaccount;AccountKey=yourdemokey;EndpointSuffix=core.windows.net",
            "BlobContainer": "images"
        },
        "CosmosDB": {
            "EndpointURI": "https://yourdemodb.documents.azure.com:443/",
            "Key": "0123456789",
            "DatabaseName": "images",
            "CollectionName": "metadata"
        }
    }
    ```
    >NOTE The settings must be retrieved from existed resources. 

    Following links should help you retrieve settings.

    - [Get connections string from storage account](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-keys-manage?tabs=azure-portal#view-account-access-keys)
    - [Get connection key for Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/create-sql-api-python#update-your-connection-string)
    - [Get Key and endpoint URL for Cognitive Service](https://docs.microsoft.com/en-us/azure/search/search-create-service-portal#get-a-key-and-url-endpoint)


### VS Code

1. Install C# extension

1. Open folder "Finished"

1. In the cmd console run command **dotnet build** in each of the folders: `ImageStorageLibrary`, `ProcessingLibrary`, `TestCLI`

1. From `TestCLI` folder run following commands:

    ```cmd	
    dotnet run -process "<%GitHubDir%>\AI-100-Design-Implement-Azure-AISol\Lab2-Implement_Computer_Vision\sample_images"	
    ```	
    > **Note** Replace the <%GitHubDir%> value with the folder where you cloned the repository.	
    Once it's done processing, you can query against your Cosmos DB directly using _TestCLI_ as follows:	
    
    ```cmd	
    dotnet run -query "select * from images"	
    ```	

1. Take some time to look through the sample images (you can find them in /sample_images) and compare the images to the results in your application.	

    > **Note** You can also browse the results in the CosmosDb resource in Azure.  Open the resource, then select **Data Explorer**.  Expand the **metadata** database, then select the **items** node.  You will see several json documents that contains your results.

### VS 2019

1. Open `ImageProcessing.sln` file in the root folder of projects.
1. Set `TestCLI` project as `Start up Project`.
1. Open `TestCLI` project properties. Select `Debug` tab and provide following string in `Application Arguments`: 

    ```
    -process "<%GitHubDir%>\AI-100-Design-Implement-Azure-AISol\Lab2-Implement_Computer_Vision\sample_images"
    ```
    > **Note** Replace the <%GitHubDir%> value with the folder where you cloned the repository.	

1. Run project by F5.
1. Take some time to look through the sample images (you can find them in /sample_images) and compare the images to the results in your application.	

    > **Note** You can also browse the results in the CosmosDb resource in Azure.  Open the resource, then select **Data Explorer**.  Expand the **metadata** database, then select the **items** node.  You will see several json documents that contains your results.