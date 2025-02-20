---
title: Deploying to Azure App Service
intro: You can deploy to Azure App Service as part of your continuous deployment (CD) workflows.
product: '{% data reusables.gated-features.actions %}'
redirect_from:
  - /actions/guides/deploying-to-azure-app-service
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
type: tutorial
topics:
  - CD
  - Containers
  - Azure App Service
shortTitle: Deploy to Azure App Service
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}
{% data reusables.actions.ae-beta %}

## Introduction

This guide explains how to use {% data variables.product.prodname_actions %} to build, test, and deploy an application to [Azure App Service](https://azure.microsoft.com/en-us/services/app-service/).

Azure App Service can run web apps in several languages, but this guide demonstrates deploying an existing Node.js project.

## Prerequisites

Before creating your {% data variables.product.prodname_actions %} workflow, you will first need to complete the following setup steps:

1. Create an Azure App Service plan.

   For example, you can use the Azure CLI to create a new App Service plan:

   ```bash{:copy}
   az appservice plan create \
       --resource-group MY_RESOURCE_GROUP \
       --name MY_APP_SERVICE_PLAN \
       --is-linux
   ```

   In the command above, replace `MY_RESOURCE_GROUP` with your pre-existing Azure Resource Group, and `MY_APP_SERVICE_PLAN` with a new name for the App Service plan.

   See the Azure documentation for more information on using the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/):

   * For authentication, see "[Sign in with Azure CLI](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli)".
   * If you need to create a new resource group, see "[az group](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az_group_create)."

2. Create a web app.

   For example, you can use the Azure CLI to create an Azure App Service web app with a node runtime:

   ```bash{:copy}
   az webapp create \
       --name MY_WEBAPP_NAME \
       --plan MY_APP_SERVICE_PLAN \
       --resource-group MY_RESOURCE_GROUP \
       --runtime "node|10.14"
   ```

   In the command above, replace the parameters with your own values, where `MY_WEBAPP_NAME` is a new name for the web app.

3. Configure an Azure publish profile and create an `AZURE_WEBAPP_PUBLISH_PROFILE` secret.

   Generate your Azure deployment credentials using a publish profile. For more information, see "[Generate deployment credentials](https://docs.microsoft.com/en-us/azure/app-service/deploy-github-actions?tabs=applevel#generate-deployment-credentials)" in the Azure documentation.

   In your {% data variables.product.prodname_dotcom %} repository, create a secret named `AZURE_WEBAPP_PUBLISH_PROFILE` that contains the contents of the publish profile. For more information on creating secrets, see "[Encrypted secrets](/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository)."

4. For Linux apps, add an app setting called `WEBSITE_WEBDEPLOY_USE_SCM` and set it to true in your app. For more information, see "[Configure apps in the portal](https://docs.microsoft.com/en-us/azure/app-service/configure-common#configure-app-settings)" in the Azure documentation.

{% ifversion fpt or ghes > 3.0 or ghae %}
5. Optionally, configure a deployment environment. {% data reusables.actions.about-environments %}
{% endif %}

## Creating the workflow

Once you've completed the prerequisites, you can proceed with creating the workflow.

The following example workflow demonstrates how to build, test, and deploy the Node.js project to Azure App Service when there is a push to the `main` branch.

Ensure that you set `AZURE_WEBAPP_NAME` in the workflow `env` key to the name of the web app you created. You can also change `AZURE_WEBAPP_PACKAGE_PATH` if the path to your project is not the repository root and `NODE_VERSION` if you want to use a node version other than `10.x`.

{% data reusables.actions.delete-env-key %}

```yaml{:copy}
{% data reusables.actions.actions-not-certified-by-github-comment %}

on:
  push:
    branches:
      - main

env:
  AZURE_WEBAPP_NAME: MY_WEBAPP_NAME   # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '10.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v2

      - name: Use Node.js {% raw %}${{ env.NODE_VERSION }}{% endraw %}
        uses: actions/setup-node@v2
        with:
          node-version: {% raw %}${{ env.NODE_VERSION }}{% endraw %}

      - name: npm install, build, and test
        run: |
          # Build and test the project, then
          # deploy to Azure Web App.
          npm install
          npm run build --if-present
          npm run test --if-present

      - name: 'Deploy to Azure WebApp'
        uses: azure/webapps-deploy@0b651ed7546ecfc75024011f76944cb9b381ef1e
        with:
          app-name: {% raw %}${{ env.AZURE_WEBAPP_NAME }}{% endraw %}
          publish-profile: {% raw %}${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}{% endraw %}
          package: {% raw %}${{ env.AZURE_WEBAPP_PACKAGE_PATH }}{% endraw %}
```

## Additional resources

The following resources may also be useful:

* For the original starter workflow, see [`azure.yml`](https://github.com/actions/starter-workflows/blob/main/deployments/azure.yml) in the {% data variables.product.prodname_actions %} `starter-workflows` repository.
* The action used to deploy the web app is the official Azure [`Azure/webapps-deploy`](https://github.com/Azure/webapps-deploy) action.
* For more examples of GitHub Action workflows that deploy to Azure, see the 
[actions-workflow-samples](https://github.com/Azure/actions-workflow-samples) repository.
* The "[Create a Node.js web app in Azure](https://docs.microsoft.com/en-us/azure/app-service/quickstart-nodejs)" quickstart in the Azure web app documentation demonstrates using VS Code with the [Azure App Service extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice).
