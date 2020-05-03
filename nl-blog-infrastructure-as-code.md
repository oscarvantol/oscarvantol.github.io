# Infrastructure als code met Azure

*Vanaf het moment dat een ontwikkelaar een code schrijft ten behoeve van een nieuwe feature of om een bug te verhelpen, begint een reis. Hopelijk start deze reis met een vorm van een commit richting een versiebeheersysteem. De eindbestemming van deze reis is bekend, de productieomgeving (wat dat in de betreffende context ook mag zijn). De reis kan kort en simpel zijn of lang en ingewikkeld, de tijd zal voornamelijk bepaald worden door het kwaliteitsbewakingsproces en het niveau van automatisering van de CI/CD pipeline.*

## Build en Release

Vandaag de dag is het vrij normaal dat teams hun builds geautomatiseerd hebben en dat deze starten na een nieuwe commit. Binnen deze eerste stap in het CI proces is het ook gebruikelijk om bijvoorbeeld unit tests te draaien. De volgende stap zal het release proces zijn, in de meeste gevallen gebruikt dit de output van de build en plaatst deze op een omgeving. Dit kan een test omgeving zijn, een ring, een kanarie of misschien wel direct naar productie.

Maar waar gaat die release exact heen? Vroeger was dit een stuk metaal maar tegenwoordig is het gebruikelijker dat dit een virtuele machine, container of hippe cloud service is. Omdat van oudsher deze omgevingen statischer waren is het proces om deze op te tuigen minder vaak al geautomatiseerd. Maar als je je omgeving en de benodigde configuratie niet opnieuw kan creëren door middel van een documentatie of een (geautomatiseerd) script ben je in het bezit van 'technical debt' die je niet moet onderschatten!

## Documentatie volledig up-to-date houden

Als we op empirisch bewijs af moeten gaan, is er een minuscule kans dat dit lukt. En als het het geval is, zou het nog steeds extreem inefficiënt zijn.

```
// TODO: write some documentation
// TODO: Remove this code after 2009-01-01
```


## Infrastructure als code

Het is de investering meer dan waard om je resources automatisch te creëren. Stel voor dat je iets nieuws hebt gemaakt dat een storage service en een web service behoeft in Azure, met een paar kliks in de portal ben je klaar. Dit werkt prima in de veilige ontwikkelomgeving waarin je volledige toegang hebt. De twee nieuwe services en hun configuratie zullen waarschijnlijk ook nog naar één of meerder testomgevingen en productieomgeving moeten. Het scripten en parametriseren van deze werkzaamheden zal je snel efficiënter maken. Een bijkomend voordeel van het uitschrijven in een script is dat het gereviewd kan worden en bijvoorbeeld het breken van afspaken over naamgeving op tijd gespot kunnen worden. Als het script net als alle ander code netjes in een versiebeheersysteem is geplaatst ben je meteen in het bezit van een compleet historisch beeld hoe een resource is aangemaakt.

**Top ik ben om! Hoe dan?**

Zoals altijd zijn er meerdere oplossingen om dit te bewerkstelligen. Azure stelt api's beschikbaar om resources te manipuleren, er zijn veel tools die met deze api's overweg kunnen. Voor nu zoomen we in op de opties die het meest dicht bij Azure liggen en die gemakkelijk in een Azure DevOps Pipeline gebruikt kunnen worden.


De volgende drie opties tonen een voorbeeld dat van de officiële Azure Documentation pagina's komt. Deze voorbeelden laten zien hoe een Azure Web App met een Service Plan gemaakt kan worden.

### ARM Templates

Misschien wel de meest voor de hand liggende methode om 'Infrastructure as Code' in Azure te verwezenlijken is het gebruik van ARM Templates. ARM staat voor Azure Resource Manager en dat is al enkele jaren de standaard set van api's om resources te manipuleren, de services die aangeduid worden met 'Classic' kun je beheren met de ASM (Azure Service Management) api.

ARM Templates zijn een JSON representatie van één of meerdere resources in Azure. In een template kan je een complexe set van resources en hun afhankelijkheden definiëren. In de Azure portal heb je bij elke resource de mogelijkheid om een ARM template te exporteren, dit klinkt als dé manier om te starten toch? Ondanks dat het voor de hand ligt om deze optie te gebruiken zal dit je een script geven die vooral goed te lezen is door een machine maar niet zozeer door een mens. Het onderstaande voorbeeld is op zich vrij leesbaar, maar dit is knap lastig om uit het niets te schrijven als je hier nog niet heel ervaren in bent.

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

In afgelopen jaren zijn 'Command Line Lools' erg gegroeid in populariteit, zo ook de Azure CLI. De Azure CLI is eigenlijk een command line versie van de Azure Portal, de inline hulp functie helpt je de tool makkelijk verkennen. Omdat je de commando's in een geparametriseerd Poweshell of bash script kan opnemen is het makkelijk in een versiebeheersysteem op te slaan en uit te voeren in een CD pipeline. Het aanmaken van resources in Azure met behulp van de CLI doorloopt vrijwel dezelfde stappen als dat je het via de Azure Portal zou doen.


```
az group create -l westeurope -n MyResourceGroup
az appservice plan create -g MyResourceGroup -n MyPlan -l westeurope
az webapp create -g MyResourceGroup -p MyPlan -n MyUniqueAppName
```


### Azure Management Libraries for .NET

Deze library zal je normaalgesproken vooral vinden binnen de onderdelen van een applicatie die direct met de Azure infrastructuur moet communiceren. Als je bijvoorbeeld een multi-tenant SAAS applicatie hebt waarbij elke 'tenant' van de applicatie een eigen SQL database heeft, wil je bij het aanmaken van een nieuwe klant deze database vanuit de code creëren. Het kan ook zo zijn dat je op een hele specifieke manier je applicatie wil kunnen schalen en daar eigen logica voor hebt gebouwd. Het kan zo zijn dat je als developer zo comfortabel bent met het schrijven van C# dat het coderen van deze scripts de voorkeur heeft. De Azure Management Libraries' client heeft een eenvoudige fluent api.

```
IAzure azure = Azure.Authenticate(credFile).WithDefaultSubscription();

var webApp = azure.WebApps.Define(appName)
    .WithRegion(Region.USWest)
    .WithNewResourceGroup(rgName)
    .WithNewFreeAppServicePlan()
    .Create();
```


## Een keuze maken

Het belangrijkste is het omarmen van het feit dat het essentieel is om de definitie van infrastructuur waar je applicaties op draaien vast te leggen in een versiebeheersysteem. Vooralsnog is de officiële wijze het gebruiken van de ARM templates. Het voordeel van het gebruik van ARM templates is dat je een eindstaat beschrijft, dat betekend dat je reproduceerbaar gedrag krijgt onafhankelijk van de beginsituatie. Met de Azure CLI is het eenvoudiger om vanuit het niets met een script te starten, hierbij zal je hier en daar wel wat 'flow' moeten schrijven. De meeste commando's zijn idempotent en zullen geen errors opleveren als een resource al bestaat, toch zijn er hier en daar een aantal [commandos](https://github.com/Azure/azure-cli/issues/11863) die wat onverwacht gedrag hebben. Voor maximale controle om bijvoorbeeld complexe omgevingen op te kunnen tuigen zijn de Azure Management Libraries een optie, het gebruik hiervan betekend wel dat je deze code ook eerst moet compileren naar een executable.

*Nuttige links:*
- [ARM Templates documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview)
- [Azure CLI command reference](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest)
- [Azure Management Libraries for .NET](https://github.com/Azure/azure-libraries-for-net)


