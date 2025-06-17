# Clone and deploy a .NET 9 Blazor chat app connected to Azure OpenAI and Cosmos DB

## Prerequisites
- Azure OpenAI resource with deployed embeddings and chat models
- Cosmos Database resource with vector embeddings

### 1. Clone the sample app
```
git clone https://github.com/Azure-Samples/blazor-cosmos-db-vector-search.git
```

### 2. Add Azure OpenAI and Cosmos DB credentials
Go to the `appsettings.json` file and insert the following values from your Azure OpenAI and Cosmos DB resources.

**Azure OpenAI**
- `DEPLOYMENT_NAME = <deployment-name>`
- `ENDPOINT = https://<your-endpoint-name>.openai.azure.com/`
- `API_KEY = <api-key>`
- `MODEL_ID = <model-id>`

**Azure SQL**
- `ENDPOINT_DB = https://<your-cosmos-endpoint-name>.documents.azure.com:443/`

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "COSMOS_DB": {
    "ENDPOINT_DB": "https://<your-cosmos-endpoint-name>.documents.azure.com:443/"
  },
    "OpenAI": {
      "DEPLOYMENT_NAME": "<deployment-name>",
      "ENDPOINT": "https://<your-endpoint-name>.openai.azure.com/",
      "API_KEY": "<api-key>",
      "MODEL_ID": "<model-id>"
    },
    "AllowedHosts": "*"
  }
```

### 3. Update your database and container names
In the `Chat.razor` file, update the database and container names to yours so the correct container can be retrieved.  

```
        Database database = client.GetDatabase("<your-database-name>");
        Container container = database.GetContainer("<your-container-name>");
```

Note, the Cosmos DB client is using a Default Azure Credential with Managed Identity that's required to connect to Cosmos DB.

```
        CosmosClient client = new(
            accountEndpoint: _config["COSMOS_DB:ENDPOINT_DB"],
            tokenCredential: new DefaultAzureCredential()
        );
```

### 3. Run the application
Once the values are saved, run the application using F5 and navigate to the Chat blade. You can then ask a question in the input field to start the chat and receive a response from your Azure OpenAI service grounded in your Azure Cosmos DB data.

### 4. Deploy to App Service
You can deploy the web app as you normally would. Although your secrets in app settings are encrypted at rest, we highly recommend setting up managed identity to secure your resources. Please see the documentation for this sample to secure your resources with [managed identity](https://learn.microsoft.com/azure/app-service/deploy-intelligent-apps-dotnet-to-azure-sql#secure-your-data-with-managed-identity)

The app includes the code for Managed Identity. Uncomment the code for the Azure OpenAI chat client and the embedding client to use managed identity with the `DefaultAzureCredential`. This removes the need to use API Keys when deploying to App Service.

```
        // // Chat completion service - Managed Identity
        // builder.Services.AddAzureOpenAIChatCompletion(
        //     deploymentName: deploymentName,
        //     endpoint: endpoint,
        //     credentials: new DefaultAzureCredential()
        // );

        // // Azure OpenAI client - Managed Identity
        // AzureOpenAIClient azureClient = new(
        //      new Uri("https://<your-endpoint-name>.openai.azure.com/"),
        //      new DefaultAzureCredential()
        // );
```

