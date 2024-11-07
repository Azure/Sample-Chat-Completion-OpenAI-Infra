# Azure Infrastructure for Open AI powered Chat

## Overview

This action deploys a chat-assistant app based on Open AI-powered Large Language Model (LLM) and hosted on Azure. It deploys all the infrastructure the chat assistant needs, using an external repository to reference the template. This chat assistant is intended for demo purposes and can answer questions about a fictitious company called Contoso Electronics, and allows its employees to ask questions about the benefits, internal policies, as well as job descriptions and roles.

Template: [ChatGPT-like app with your data using Azure OpenAI and Azure AI Search (Python)](https://github.com/Azure-Samples/azure-search-openai-demo)

## Installation

1. If you're new to Azure, [get an Azure account for free](https://azure.microsoft.com/free/cognitive-search/) and you'll get some free Azure credits to get started.
2. Your Azure Subscription will need to have access to Azure OpenAI service [OpenAI access request process](https://aka.ms/oai/access)
3. Please install the [Configure-Azure-Settings](https://github.com/marketplace/configure-azure-settings) app from the GitHub Marketplace to populate all of the inputs needed for Azure login and resource deployment. This app will populate the inputs as secrets in your repositories.
   - **client-id** (required): Client ID used for Azure login, set as **AZURE_CLIENT_ID** in repo secrets.
   - **tenant-id** (required): Tenant ID used for Azure login, set as **AZURE_TENANT_ID** in repo secrets.
   - **subscription-id** (required): Azure subscription ID used with the `az login`, set as **AZURE_SUBSCRIPTION_ID** in repo variables.
   - **principal-id** (required): Managed Identity Principal ID, set as **AZURE_MANAGEDIDENTITY_PRINICIPAL_ID** in repo secrets.
   - **location-id** (required):  Location to deploy resources, to set as **AZURE_LOCATION** in repo variables


4. Set these required values as variables on your repo where the workflow runs or pass them as parameters to the action in the workflow directly:

   - **env-name** (required): Name of environment where env values are set, set as **AZURE_ENV_NAME** in repo variables.

   - **openai-location** (required): Location to deploy open ai resources to set as **AZURE_OPENAI_LOCATION** in repo variables.

     For the openai-location, the regions that currently support the OpenAI models used in this sample at the time of writing this are 'canadaeast', 'eastus', 'eastus2', 'francecentral', 'switzerlandnorth', 'uksouth', 'japaneast', 'northcentralus', 'australiaeast', 'swedencentral'.

   - **documentintelligence-location** (required): Location to deploy document intelligence resources to set as **AZURE_DOCUMENTINTELLIGENCE_LOCATION** in repo variables.

     For the documentintelligence-location, the regions that currently support the OpenAI models used in this sample at the time of writing this are 'eastus', 'westus2', 'westeurope' [Learn more](https://learn.microsoft.com/azure/ai-services/document-intelligence/concept-layout)

## Optional Inputs

Optional customizable parameters that enable integration of the deployed AI infrastructure with your specific data

- **bring-your-own-data** (optional): Boolean to indicate if you are bringing your own data, set as **OWN_DATA** in repo variables.
- **data-path** (optional): Path to data to be used to train the OpenAI model, set as **DATA_PATH** in repo variables.

Further customization of resources like quota, tier, etc... can be done by passing in [variables](https://github.com/Azure-Samples/azure-search-openai-demo/blob/main/infra/main.parameters.json) as seen below.  [Template documentation](https://github.com/Azure-Samples/azure-search-openai-demo/blob/main/docs/README.md)

````yaml
- name: Deploy Open AI infrastructure action execution
  uses: ./
  with:
    ...
    additional-args: '{
      "AZURE_APP_SERVICE_SKU":"B2", 
    }'
````

- Note: Setting a parameter as part of additional-args is the same as doing **azd env set <VAR_NAME> <VALUE>** in the original template.

## Usage

In this section, you'll create a workflow file in your GitHub repository that executes this GitHub Action which deploys AI infrastructure along with a chat interface on Azure automatically. By setting this up, every time you make changes or run the workflow, GitHub will handle all the steps needed to deploy the necessary infrastructure, making it easy to keep your chat assistant up and running without manual effort.
Create this workflow in your repo on this path: `.github/workflows/workflow_file.yml`, and customize it with the required/optional inputs.

```yaml
name: Workflow to deploy OpenAI infrastructure to Azure

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Create and start virtual environment
        run: |
          python -m venv venv
          source venv/bin/activate

  deploy-resources-to-azure:
    runs-on: ubuntu-latest
    needs: build
    env:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
      AZURE_PRINCIPAL_ID: ${{ secrets.AZURE_MANAGEDIDENTITY_PRINCIPAL_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ vars.AZURE_SUBSCRIPTION_ID }}
      AZURE_RG: ${{ vars.AZURE_RG }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Login to Azure
        uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
          
      - name: Deploy Open AI infrastructure action execution
        uses: Azure/Sample-Chat-Completion-OpenAI-Infra@v6
        with:
          location: ${{ vars.AZURE_LOCATION }}
          env-name: ${{ vars.AZURE_ENV_NAME }}
          openai-location: ${{ vars.AZURE_OPENAI_LOCATION }}
          documentintelligence-location: ${{ vars.AZURE_DOCUMENTINTELLIGENCE_LOCATION }}
          # optional parameters
          additional-args: '{
            "AZURE_APP_SERVICE_SKU":"B2", 
            }'
          # bring-your-own-data: true
          # data-path: "mydata"
         
```

## Using your own data

The Chat App is designed to work with any PDF documents. The sample data is provided to help you get started quickly, but you can easily replace the sample data with your own data.

1. You will need to upload your data folder to the repo containing the workflow above, and set **data-path** as your data folder path. Folder should only contain files that will be used for training.

1. Please change env name for a new set of resources. If the same env name is used, then new data will get re-deployed to previously created resources.
  
**Note**: The frontend is built using React and Fluent UI components. In order to customize prompts seen on the chat interface, you will have to clone the template and make those changes. [Refer to template for further information](https://github.com/Azure-Samples/azure-search-openai-demo)

## Troubleshooting

Below you can find the most common failure scenarios and solutions:

1. The subscription (`AZURE_SUBSCRIPTION_ID`) doesn't have access to the Azure OpenAI service. Please ensure `AZURE_SUBSCRIPTION_ID` matches the ID specified in the [OpenAI access request process](https://aka.ms/oai/access).

1. You're attempting to create resources in regions not enabled for Azure OpenAI (e.g. East US 2 instead of East US), or where the model you're trying to use isn't enabled. See [this matrix of model availability](https://aka.ms/oai/models).

1. You've exceeded a quota, most often number of resources per region. See [this article on quotas and limits](https://aka.ms/oai/quotas).

1. You're getting "same resource name not allowed" conflicts. That's likely because you've run the sample multiple times and deleted the resources you've been creating each time, but are forgetting to purge your cognitive resources such as OpenAI or Document Intelligence. Azure keeps resources for 48 hours unless you purge from soft delete. See [this article on purging resources](https://learn.microsoft.com/azure/cognitive-services/manage-resources?tabs=azure-portal#purge-a-deleted-resource).

1. After running `azd up` and visiting the website, you see a '404 Not Found' in the browser. Wait 10 minutes and try again, as it might be still starting up. Then try re-running the workflow and wait again. If you still encounter errors with the deployed app, consult the [guide on debugging App Service deployments](docs/appservice.md). Please file an issue if the logs don't help you resolve the error.

1. If you encounter 'Application error' or 503 'Service Unavailable' when using your own data, it may be due to the app service plan reaching capacity because of the size of files provided. Use the deployment link provided in the workflow logs to scale up your app service plan using this [guide](https://learn.microsoft.com/en-us/azure/app-service/manage-scale-up)

## Output

The action deploys an Open AI infrastructure to Azure and prints a URL endpoint on the console at the end of the workflow. Click the URL to interact with the chat application in your browser.
