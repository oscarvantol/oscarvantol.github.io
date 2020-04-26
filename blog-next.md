[< back](index)

<sub>blogged: 2020.04.27</sub>

# Azure infrastructure as code 


*From the moment a developer writes a piece of code to support a new feature or fix a bug, a journey starts. This journey will hopefully start with some sort of commit and push in a source control system. The end goal of the journey is pretty clear, production (whatever that is in the context). What happens in between can be a short and simple or long and complicated, the time the journey takes will mostly be determined by the quality assurance process and the level of automation of the CI/CD pipeline.*


## Build and Release

These days it is pretty common for teams to have their builds automated and triggered on a new commit. In this first CI step it is also pretty common to run some form of (unit)testing. 

The next step is the release process, in most cases this process will take a build and deploys it to an environment wether it is a qa environment, a ring, canary or straight to production. 

But where is it releasing to? It can be a virtual machine, a real piece of metal or some cloud service. Because these resources were always so static, these and their configuration are regularly the last one in line to be automated.

If you cannot recreate your environment with its configuration with up to date documentation or automation you are suffering from technical debt that you should not underestimate.


## Keeping documentation perfectly up to date

```
// TODO: write some documentation
// TODO: Remove this code after 2009-01-01
```
Based on empirical evidence, there will be a remote possibillity that this will happen. And if it would, it is still extremely ineffecient.



## Infrastructure as code

Creating your resources automaticly will pay off really quickly. Let's say you dream up some new piece of code that uses some storage service and web service in Azure, it is only a few clicks to create it in the portal. But you will probably do this in a safe development environment were you have all the access you can dream of. Your two new services and their configuration may also be needed in one or more test environments and the production environment. A extra benefit of automating this is that it can be reviewed and that issues with for example naming convention can be spotted early on. If you treat you creation scripts as code and store them in a version control system you have perfect historical record of how a resource was created.

**Great let's do this! Tell me how!**

Like always, there is more than one way to accomplish this. Azure exposes apis to manipulate resources, there are a lot of tools that can talk to these apis. For now let's focus on the options closest to Azure itself that can fairly easily be triggered in an Azure Pipeline.

The following three options contain an example that is taken from the official Azure Documentation pages. These examples show the creation an Azure Web App with a Service Plan.

### ARM Templates

Maybe the most common way to do 'Infrastructure As Code' in Azure is to use ARM Templates. ARM stands for Azure Resource Manager and has been the standard set of apis for some years now, the services that are refered to as 'Classic' in Azure are controlled with ASM (Azure Service Management) api. 

ARM Templates are a JSON representation of one or more resources in Azure, in a single template you can define a whole set of complex resources and their dependencies for a resource group. If you are in the Azure portal you can go to any resource and export a ARM template, this sounds like the way to start doesn't it? Even though it sounds like a perfect and simple solution this will generate a complext script that is a lot better readable by a machine than a human.
If you look at the example below it might be readable enough, but for me something like this is hard write from scratch without too much experience.

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "webAppName": {
      "type": "string",
      "metadata": {
        "description": "Base name of the resource such as web app name and app service plan"
      },
      "minLength": 2
    },
    "sku":{
      "type": "string",
      "defaultValue" : "S1",
      "metadata": {
        "description": "The SKU of App Service Plan, by default is Standard S1"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources"
      }
    }
  },
  "variables": {
    "webAppPortalName": "[concat(parameters('webAppName'), '-webapp')]",
    "appServicePlanName": "[concat('AppServicePlan-', parameters('webAppName'))]"
  },
  "resources": [
    {
      "apiVersion": "2018-02-01",
      "type": "Microsoft.Web/serverfarms",
      "kind": "app",
      "name": "[variables('appServicePlanName')]",
      "location": "[parameters('location')]",
      "properties": {},
      "dependsOn": [],
      "sku": {
        "name": "[parameters('sku')]"
      }
    },
    {
      "apiVersion": "2018-11-01",
      "type": "Microsoft.Web/sites",
      "kind": "app",
      "name": "[variables('webAppPortalName')]",
      "location": "[parameters('location')]",
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]"
      ]
    }
  ]
}
```



### Azure CLI scripts

Command line tools everywhere and also the Azure CLI has gained popularity in recent years. The Azure CLI is basically a command line version of the Azure Portal with good feature discoverabillity due to the inline help functionallity. Because you can script the use of a command line tool in powershell or bash you can create parameterized automation scripts that you can store in source control and run in CD pipelines. The flow of creating resources will be similar to the steps you would normally take to create something in the portal. 

```
az group create -l westeurope -n MyResourceGroup
az appservice plan create -g MyResourceGroup -n MyPlan -l westeurope
az webapp create -g MyResourceGroup -p MyPlan -n MyUniqueAppName
```


### Azure Management Libraries for .NET

Normally you would see this library inside the parts of your application that needs to control infrastructure in Azure. For example, if you have a multi tenant SAAS application that has SQL databases for every tenant you can use the libraries to create a new db on a new customer signup or maybe you have created some smart code because you have some really speficic scaling requirements. But if the developers are comfortable writing C# all day this might be a fine way to script/code the creation of resources. With the Azure Management Libraries you can define services using a fluent api. 

```
IAzure azure = Azure.Authenticate(credFile).WithDefaultSubscription();

var webApp = azure.WebApps.Define(appName)
    .WithRegion(Region.USWest)
    .WithNewResourceGroup(rgName)
    .WithNewFreeAppServicePlan()
    .Create();
```

This option provides maximum control in complex setups but be aware that you are creating an extra application that also needs to be built.


## Making a choice
The most important part is embracing the fact that it is important to commit the definition of the infrastructure your code needs to run on into a source control system. It still seems like the official recomondation is to use ARM templates. The strong part of the ARM templates is that you do not define a script but you define a state that you want your resources in, this means that you have reproducable behavior no matter of the state you start with.
The CLI commands are simple to write from scratch, but you are not defining a state so you might need to script out a bit of flow. Most of the commands are idempotent and will not throw errors on recreation of a resouce, but I found out that some of the commands still have some unexpected behavior. (oops,  TODO)






[oscarvantol.nl](https://oscarvantol.nl)
