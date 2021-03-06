# Overview

Tailspin Toys has asked you to automate their development process in two specific ways. First, they want you to define an Azure Resource Manager template that can deploy their application into the Microsoft Azure cloud using Platform-as-a-Service technology for their web application and their PostgreSQL database. Second, they want you to implement a continuous delivery process that will connect their source code repository into the cloud, automatically run their code changes through unit tests, and then automatically create new software builds and deploy them onto environmentspecific deployment slots so that each branch of code can be tested and accessed independently.

## Solution architecture

  ![Continuous Collaboration](images/img1.png)

## Requirements

1. Microsoft Azure subscription

> Note: This entire lab can be completed using only the Azure Portal.

### Exercise 1: Create an Azure Resource Manager (ARM) template that can provision the web application, PostgreSQL database, and deployment slots in a single automated process 

Duration: 45 Minutes 

Tailspin Toys has requested three Azure environments (dev, test, production), each consisting of the following resources:

- App Service o Web App
   - Deployment slots (for zero-downtime deployments)
- PostgreSQL Server o PostgreSQL Database
- PostgreSQL Database

Since this solution is based on Azure Platform-as-a-Service (PaaS) technology, it should take advantage of that platform by utilizing automatic scale for the web app and the PostgreSQL Database PaaS service instead of running virtual machines.

#### Task 1: Create an Azure Resource Manager (ARM) template using Azure Cloud Shell

1. From within the **Azure Cloud Shell** locate the folder where you previously unzipped the Student Files. Open **Code** to this folder with the command below. It should also contain two sub-folders: **armtemplate** and **tailspintoysweb**.

  `code .`

  ![Continuous Collaboration](images/img2.png)

2. In the Code Explorer window, select the **armtemplate** sub-folder and open the **azuredeploy.json** file by selecting it. 

3. In the open editor window for the **azuredeploy.json**, scroll through the contents of the Azure Resource Manager Template. This template contains all the necessary code to deploy a Web App and a PostgreSQL database to Azure.

#### Task 2: Configure the list of release environments parameters 

1.	Next, you need to configure a list of release environments we'll be deploying to. Our scenario calls for adding three environments: dev, test, and production. This is going to require the addition of some manual code. At the top of the **azuredeploy.json** file, locate the following line of code (on or around line 4).

` "parameters": { `

    ```        
        "environment": {
            "type": "string",
            "metadata": {
                "description": "Name of environment"
            },
            "allowedValues": [
                "dev",
                "test",
                "production"
            ]
        },
    ```

After adding the code, it will look like this:

  ![Continuous Collaboration](images/img3.png)

> Note: The environment parameter will be used to generate environment specific names for our web app. 

Save the file.

#### Task 3: Add a deployment slot for the "staging" version of the site 

1. Next, you need to add the "staging" deployment slot to the web app. This is used during a deployment to stage the new version of the web app. This is going to require the addition of some manual code. In the azuredeploy.json file, add the following code to the "resources" array, just above the element for the "connectionstrings" (on or around line 145).

```

    {
        "apiVersion": "2016-08-01",
        "name": "staging",
        "type": "slots",
        "tags": {
            "displayName": "Deployment Slot: staging"
        },
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[resourceId('Microsoft.Web/Sites/', variables('webAppName'))]"
        ],
        "properties": {},
        "resources": []
    },
    
```

After adding the code, it will look like this:

  ![Continuous Collaboration](images/img4.png)

Save the file.

The complete ARM template should look like the following: 

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "environment": {
            "type": "string",
            "metadata": {
                "description": "Name of environment"
            },
            "allowedValues": [
                "dev",
                "test",
                "production"
            ]
        },
        "siteName": {
            "type": "string",
            "defaultValue": "tailspintoys",
            "metadata": {
                "description": "Name of azure web app"
            }
        },
        "administratorLogin": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Database administrator login name"
            }
        },
        "administratorLoginPassword": {
            "type": "securestring",
            "minLength": 8,
            "maxLength": 128,
            "metadata": {
                "description": "Database administrator password"
            }
        },
        "databaseSkuCapacity": {
            "type": "int",
            "defaultValue": 2,
            "allowedValues": [
                2,
                4,
                8,
                16,
                32
            ],
            "metadata": {
                "description": "Azure database for PostgreSQL compute capacity in vCores (2,4,8,16,32)"
            }
        },
        "databaseSkuName": {
            "type": "string",
            "defaultValue": "GP_Gen5_2",
            "allowedValues": [
                "GP_Gen5_2",
                "GP_Gen5_4",
                "GP_Gen5_8",
                "GP_Gen5_16",
                "GP_Gen5_32",
                "MO_Gen5_2",
                "MO_Gen5_4",
                "MO_Gen5_8",
                "MO_Gen5_16",
                "MO_Gen5_32"
            ],
            "metadata": {
                "description": "Azure database for PostgreSQL sku name "
            }
        },
        "databaseSkuSizeMB": {
            "type": "int",
            "allowedValues": [
                102400,
                51200
            ],
            "defaultValue": 51200,
            "metadata": {
                "description": "Azure database for PostgreSQL Sku Size "
            }
        },
        "databaseSkuTier": {
            "type": "string",
            "defaultValue": "GeneralPurpose",
            "allowedValues": [
                "GeneralPurpose",
                "MemoryOptimized"
            ],
            "metadata": {
                "description": "Azure database for PostgreSQL pricing tier"
            }
        },
        "postgresqlVersion": {
            "type": "string",
            "allowedValues": [
                "9.5",
                "9.6"
            ],
            "defaultValue": "9.6",
            "metadata": {
                "description": "PostgreSQL version"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for all resources."
            }
        },
        "databaseskuFamily": {
            "type": "string",
            "defaultValue": "Gen5",
            "metadata": {
                "description": "Azure database for PostgreSQL sku family"
            }
        }
    },
    "variables": {
        "webAppName": "[concat(parameters('siteName'), '-', parameters('environment'), '-', uniqueString(resourceGroup().id))]",
        "databaseName": "[concat(parameters('siteName'), 'db', parameters('environment'), uniqueString(resourceGroup().id))]",
        "serverName": "[concat(parameters('siteName'), 'pgserver', parameters('environment'), uniqueString(resourceGroup().id))]",
        "hostingPlanName": "[concat(parameters('siteName'), 'serviceplan', uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "apiVersion": "2016-09-01",
            "name": "[variables('hostingPlanName')]",
            "type": "Microsoft.Web/serverfarms",
            "location": "[parameters('location')]",
            "properties": {
                "name": "[variables('hostingPlanName')]",
                "workerSize": "1",
                "hostingEnvironment": "",
                "numberOfWorkers": 0
            },
            "sku": {
                "Tier": "Standard",
                "Name": "S1"
            }
        },
        {
            "apiVersion": "2016-08-01",
            "name": "[variables('webAppName')]",
            "type": "Microsoft.Web/sites",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Web/serverfarms/', variables('hostingPlanName'))]"
            ],
            "properties": {
                "name": "[variables('webAppName')]",
                "serverFarmId": "[variables('hostingPlanName')]",
                "hostingEnvironment": ""
            },
            "resources": [
                {
                    "apiVersion": "2016-08-01",
                    "name": "staging",
                    "type": "slots",
                    "tags": {
                        "displayName": "Deployment Slot: staging"
                    },
                    "location": "[resourceGroup().location]",
                    "dependsOn": [
                        "[resourceId('Microsoft.Web/Sites/', variables('webAppName'))]"
                    ],
                    "properties": {},
                    "resources": []
                },
                {
                    "apiVersion": "2016-08-01",
                    "name": "connectionstrings",
                    "type": "config",
                    "dependsOn": [
                        "[concat('Microsoft.Web/sites/', variables('webAppName'))]"
                    ],
                    "properties": {
                        "defaultConnection": {
                            "value": "[concat('Database=', variables('databaseName'), ';Server=', reference(resourceId('Microsoft.DBforPostgreSQL/servers',variables('serverName'))).fullyQualifiedDomainName, ';User Id=', parameters('administratorLogin'),'@', variables('serverName'),';Password=', parameters('administratorLoginPassword'))]",
                            "type": "PostgreSQL"
                        }
                    }
                }
            ]
        },
        {
            "apiVersion": "2017-12-01",
            "type": "Microsoft.DBforPostgreSQL/servers",
            "location": "[parameters('location')]",
            "name": "[variables('serverName')]",
            "sku": {
                "name": "[parameters('databaseSkuName')]",
                "tier": "[parameters('databaseSkuTier')]",
                "capacity": "[parameters('databaseSkucapacity')]",
                "size": "[parameters('databaseSkuSizeMB')]",
                "family": "[parameters('databaseskuFamily')]"
            },
            "properties": {
                "version": "[parameters('postgresqlVersion')]",
                "administratorLogin": "[parameters('administratorLogin')]",
                "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
                "storageMB": "[parameters('databaseSkuSizeMB')]"
            },
            "resources": [
                {
                    "type": "firewallRules",
                    "apiVersion": "2017-12-01",
                    "dependsOn": [
                        "[concat('Microsoft.DBforPostgreSQL/servers/', variables('serverName'))]"
                    ],
                    "location": "[parameters('location')]",
                    "name": "[concat(variables('serverName'),'firewall')]",
                    "properties": {
                        "startIpAddress": "0.0.0.0",
                        "endIpAddress": "255.255.255.255"
                    }
                },
                {
                    "name": "[variables('databaseName')]",
                    "type": "databases",
                    "apiVersion": "2017-12-01",
                    "properties": {
                        "charset": "utf8",
                        "collation": "English_United States.1252"
                    },
                    "dependsOn": [
                        "[concat('Microsoft.DBforPostgreSQL/servers/', variables('serverName'))]"
                    ]
                }
            ]
        }
    ]
}    

```
