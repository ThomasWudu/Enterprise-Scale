# Deploy your own ARM templates with AzOps pipeline

This article describes how AzOps pipeline can be used to deploy resources in Azure by using ARM template and parameters files. This capability enables to bring “your own” ARM template to deploy resources in Azure at any scope.

## About AzOps pipeline for custom ARM template deployments

AzOps is a deployment pipeline that is intended for the deployment of the Enterprise-Scale platform in Azure. AzOps is not intended to be used as a single deployment pipeline for the Enterprise-Scale platform as well as all application teams for deploying resources in their landing zones. However, individual application teams can implement their own AzOps pipeline to deploy resources in their landing zones if needed.

## Deployment scopes with AzOps

AzOps allows you to deploy your own templates at the following scopes:

   - Tenant Root (/)
   - Management Group
   - Subscription
   - Resource group

The following picture depicts these deployment scopes:

![_Figure_](./media/deployment-scopes.png)

## How to deploy your own templates with AzOps

To deploy an ARM template and its corresponding parameters file by using the AzOps pipeline, it is only required to copy the ARM template and parameters file into the desired scope in your local clone of your GitHub repo. Once you have copied your ARM template and parameters file at the desired scope submit a pull request. This will instruct AzOps to deploy the ARM template into the corresponding scope in Azure.

To demonstrate this capability, we will use a custom ARM template to deploy a policy definition at the management group scope. 

> Before you start, please ensure, that your azops folder is in sync with your Azure environment. You can refer to the [Initialize Git With Current Azure configuration](./discover-environment.md) article for instructions on how to ensure your azops folder is in sync with your Azure environment.

1. Create a new feature branch. One way to create a feature branch from Visual Studio Code is by launching the command palette (CTRL + SHIFT + P) and select Git: Create Branch.

2. Create two new files (for example, policyDef-NamingConvention.json and policyDef-NamingConvention.parameters.json) in the __azops\Tenant Root Group (GUID)\path-to-your-managementGroup\managementGroup (displayName)__ folder with the following contents:

     policyDef-NamingConvention.json
    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/managementGroupDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "policyName": {
            "type": "string",
            "metadata": {
                "description": "Provide name for the policyDefinition."
            }
        },
        "policyDescription": {
            "type": "string",
            "metadata": {
                "description": "Provide a description for the policy."
            }
        },
        "namePattern": {
            "type": "string",
            "metadata": {
                "description": "Provide naming pattern."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/policyDefinitions",
            "apiVersion": "2019-09-01",
            "name": "[parameters('policyDescription')]",
            "properties": {
                "description": "[parameters('policyDescription')]",
                "displayName": "[parameters('policyName')]",
                "policyRule": {
                    "if": {
                        "not": {
                            "field": "name",
                            "like": "[parameters('namePattern')]"
                        }
                    },
                    "then": {
                        "effect": "deny"
                    }
                }
            }
        }
      ]
    }
    ```

    policyDef-NamingConvention.parameters.json
     ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "policyName": {
            "value": "enforce-resource-naming"
        },
        "policyDescription": {
            "value": "Policy to enforce naming convention pattern."
        },
        "namePattern": {
            "value": "es*"
        }
      }
    }
    ```  

     The picture below depicts an example configuration, where it is desired to deploy this policy definition at the **esiab** management group:

     ![_Figure_](./media/sample-deployment-scope.png)

     > Important:  
     > The parameters file must have the same name as your template file, and must be followed by .parameters.json. In our example, if the template file is called policyDef-NamingConvention.json, the parameters file must be called policyDef-NamingConvention.**parameters**.json

3. Commit changes to your feature branch and create a pull request.

4. __Wait for deployment to succeed__ and merge pull request to **main** branch. **Feature** branch can be deleted after the successful merge.

After a successful deployment, the resources defined in your template will be deployed at the selected scope. For the example above, a sample policy definition will be deployed at a management group scope.
