# Azure Infrastructure for Open AI powered Chat

## Overview

This GitHub Action deploys an AI-powered chat app on Azure using OpenAI. It sets up all necessary resources, including the web app and OpenAI service, deploys the code, and configures the environment. 

The final output is a fully functional chat application that can answer a wide range of questions, from general knowledge to technical topics. It's ideal for trying out conversational AI, building chatbots, or integrating AI-driven features into your projects.

Template: [ChatGPT-like Python app using Azure OpenAI](https://github.com/Azure-Samples/openai-chat-app-quickstart/)

## Prerequisites

1. If you're new to Azure, [get an Azure account for free](https://azure.microsoft.com/free/cognitive-search/) and you'll get some free Azure credits to get started.
2. Your Azure Subscription will need to have access to Azure OpenAI service [OpenAI access request process](https://aka.ms/oai/access)

## Installation

1. Please install the [Configure-Azure-Settings](https://github.com/apps/configure-azure-settings) app from the GitHub Marketplace to populate all of the inputs needed for Azure login and resource deployment. This app will populate the inputs as secrets in your repositories.
   - **client-id** (required): Client ID used for Azure login, set as **AZURE_CLIENT_ID** in repo secrets.
   - **tenant-id** (required): Tenant ID used for Azure login, set as **AZURE_TENANT_ID** in repo secrets.
   - **subscription-id** (required): Azure subscription ID used with the `az login`, set as **AZURE_SUBSCRIPTION_ID** in repo variables.
   - **principal-id** (required): Managed Identity Principal ID, set as **AZURE_MANAGEDIDENTITY_PRINICIPAL_ID** in repo secrets.
   - **location-id** (required):  Location to deploy resources, to set as **AZURE_LOCATION** in repo variables

2. Set these required values as variables on your repo where the workflow runs or pass them as parameters to the action in the workflow directly:

   - **env-name** (required): Name of environment where env values are set, set as **AZURE_ENV_NAME** in repo variables.

   - **openai-location** (required): Location to deploy open ai resources to set as **AZURE_OPENAI_LOCATION** in repo variables.

     For the openai-location, the regions that currently support the OpenAI models used in this sample at the time of writing this are 'eastus', 'swedencentral'.

## Optional Inputs

Further customization of resources like quota, tier, etc... can be done by passing in [variables](https://github.com/Azure-Samples/openai-chat-app-quickstart/blob/main/infra/main.parameters.json) as seen below.  [Template documentation](https://github.com/Azure-Samples/openai-chat-app-quickstart/blob/main/docs/README.md)

````yaml
- name: Deploy Open AI infrastructure action execution
  uses: ./
  with:
    ...
    additional-args: '{
      "AZURE_OPENAI_SKU_NAME":"S0", 
    }'
````

- Note: Setting a parameter as part of additional-args is the same as doing **azd env set <VAR_NAME> <VALUE>** in the original template.

## Usage

In this section, you'll create a workflow file in your GitHub repository. This file will execute a GitHub Action that automatically deploys AI infrastructure and a chat interface on Azure. 

By setting this up, GitHub will handle all the steps needed to deploy the necessary infrastructure whenever you make changes or run the workflow. This makes it easy to keep your chat assistant up and running without manual effort.

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
        uses: Azure/Sample-Quickstart-Chat-OpenAI-Infra@v1
        with:
          location: ${{ vars.AZURE_LOCATION }}
          env-name: ${{ vars.AZURE_ENV_NAME }}
          openai-location: ${{ vars.AZURE_OPENAI_LOCATION }}
          # optional parameters
          additional-args: '{
            "AZURE_OPENAI_SKU_NAME":"S0", 
            }'
```

## Troubleshooting

Below you can find the most common failure scenarios and solutions:

1. The subscription (`AZURE_SUBSCRIPTION_ID`) doesn't have access to the Azure OpenAI service. Please ensure `AZURE_SUBSCRIPTION_ID` matches the ID specified in the [OpenAI access request process](https://aka.ms/oai/access).

1. You're attempting to create resources in regions not enabled for Azure OpenAI (e.g. East US 2 instead of East US), or where the model you're trying to use isn't enabled. See [this matrix of model availability](https://aka.ms/oai/models).

1. You've exceeded a quota, most often number of resources per region. See [this article on quotas and limits](https://aka.ms/oai/quotas).

1. You're getting "same resource name not allowed" conflicts. That's likely because you've run the sample multiple times and deleted the resources you've been creating each time. However, you might be forgetting to purge your cognitive resources such as OpenAI or Document Intelligence. Azure keeps resources for 48 hours unless you purge from soft delete. See [this article on purging resources](https://learn.microsoft.com/azure/cognitive-services/manage-resources?tabs=azure-portal#purge-a-deleted-resource).

1. After running `azd up` and visiting the website, you see a '404 Not Found' in the browser. Wait 10 minutes and try again, as it might be still starting up. Then try re-running the workflow and wait again. If you still encounter errors with the deployed app, consult the [guide on debugging App Service deployments](docs/appservice.md). Please file an issue if the logs don't help you resolve the error.

1. If you encounter 'Application error' or 503 'Service Unavailable' when using your own data, it may be due to the app service plan reaching capacity because of the size of files provided. Use the deployment link provided in the workflow logs to scale up your app service plan using this [guide](https://learn.microsoft.com/en-us/azure/app-service/manage-scale-up)

## Output

The action deploys an Open AI infrastructure to Azure and prints a URL endpoint on the console at the end of the workflow. Click the URL to interact with the chat application in your browser.
