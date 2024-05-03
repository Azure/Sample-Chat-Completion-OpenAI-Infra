# Azure Infrastructure for Open AI powered Chat

## Overview

This project provides a Chat-Completion experience tailored specifically for Information about Surface Devices. It utilizes an external repository to reference the template. This specialized assistant can provide detailed information on Surface Devices, including specifications, troubleshooting, and warranty details. Additionally, it addresses inquiries related to sales, availability, and current trends.

[For more information on this template](https://github.com/Azure-Samples/openai/tree/main/End_to_end_Solutions/AOAISearchDemo)

**Note**: Please install the [Configure-Azure-Settings](https://github.com/apps/configure-azure-settings) app from the GitHub Marketplace to populate the below inputs as secrets in your repository

## Inputs

- **client-id** (required): Client ID used for Azure login set as AZURE_CLIENT_ID in repo secrets.
- **tenant-id** (required): Tenant ID used for Azure login set as AZURE_TENANT_ID in repo secrets.
- **subscription-id** (required): Azure subscription ID used with the `az login` set as AZURE_SUBSCRIPTION_ID in repo variables.
- **principal-id** (required): FIC Service Principal ID set as AZURE_APP_ID in repo secrets.

Set these values as variables on your repo where the workflow runs

- **location** (required): Location to deploy resources to set as AZURE_LOCATION in repo variables.
- **openai-location** (required): Location to deploy resources to set as AZURE_OPENAI_LOCATION in repo variables.

    For the openai-location, the regions that currently support the OpenAI models used in this sample at the time of writing this are East US(eastus) or South Central US(southcentralus). For an up-to-date list of regions and models, check here.
- **env-name** (required): Name of environment where env values are set set as AZURE_ENV_NAME in repo variables.

Use the name field from this link corresponding to desired location. [Azure Regions List](https://gist.github.com/ausfestivus/04e55c7d80229069bf3bc75870630ec8)

## Usage

Create this workflow in your repo on this path: `.github/workflows/workflow_file.yml`

```yaml
name: Workflow to deploy AI Powered Chat Completion infra to Azure

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
  deploy-resources-to-azure:
    runs-on: ubuntu-latest
    env:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ vars.AZURE_SUBSCRIPTION_ID }}
    steps:
        - name: Checkout master
          uses: actions/checkout@v3
          
        - name: Deploy a Open AI action
          uses: Azure/Sample-Chat-Completion-OpenAI-Infra@v4
          with:
            client-id: ${{ secrets.AZURE_CLIENT_ID }}
            tenant-id: ${{ secrets.AZURE_TENANT_ID }}
            principal-id: ${{ secrets.AZURE_APP_ID }}
            subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
            location: ${{ vars.AZURE_LOCATION }}
            openai-location: ${{ vars.AZURE_OPENAI_LOCATION }}
            env-name: ${{ vars.AZURE_ENV_NAME }}

```

## Output

The action deploys an Open AI application to Azure and prints 2 URLs on the console at the end of the workflow. Click the backend URL to interact with the application in your browser.