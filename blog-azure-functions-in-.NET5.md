[< back](index)

<sub>blogged: 2021.03.26</sub>

# Azure Functions in .NET 5 and beyond
In November of 2020 .NET 5 got released, the promise of one dot net. The 'core' addition has been dropped and Full Framework evolution has stopped at the 4.8 version. This does not mean Full Framework support ends, this is tied to the support of windows therefore you can expect an eternity of patches. But when you are building something new at this moment you should go for .NET 5 or .NET core 3.1 because of its long term support. 

Right from the release we could start using .NET 5 in for example Azure App Service but for Azure Functions this was not the case.

## Why wasn't it there already?
The reason that this is a bit more challenging is because the way the .NET model for Azure Function is implemented. The functions that you create will build a library that is loaded dynamically and hosted inside a running process. That means that that hosted process determines what version of .NET running. Azure Functions V1 was running .NET Full Framework and from V2 this was .NET core. The design originates from Azure WebJobs and as you can imagine this has some history by now. All this makes it a lot of work to get every to a next version of .NET but also to get everything on the Azure side working side by side with a lot of versions.

## What's different?
By the end of 2020 the team [reported](https://techcommunity.microsoft.com/t5/apps-on-azure/net-5-support-on-azure-functions/ba-p/1973055) that they would start building a new model for .NET, the "out-of-process" or "isolated" model. This model means that the application you build is no longer loaded inside the host but it is running by itself and communicating with the Functions runtime. This model is similar to the way all other (non .NET) languages run in Azure Functions.

## What's new?
Early March 2021 this [article](https://techcommunity.microsoft.com/t5/apps-on-azure/net-on-azure-functions-roadmap/ba-p/2197916) appeared and we got a 1.0 version of the new ["isolated process"](https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide) model, the classic model is now referred to as the [class library](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-class-library?tabs=v2%2Ccmd) model.

The loose coupling means less possibility for dependency conflicts that you might have ran into in the past. The complete separation would also mean that you could start using future preview releases. The startup of your application is similar to the model in ASP.NET using the hostbuilder where you can handle your configuration, dependency injection and almost identical middleware support.

## Whats the catch?
It will definitely take some time to upgrade your functions, the leap might be even bigger than going from v1 to v2. The new model makes a lot of sense, feels clean and light weight but is also not as mature yet. A lot of features that we are used to are not there yet. The main branch is moving pretty fast and apis are being opened up for all kinds extensibility, be sure to monitor that.

 The elephant in the room is most definitely Durable Functions. The design of DF and rich bindings rely on the tight coupling that we no longer have. It is on the roadmap to get this back but this will take some time because in November there is another LTS release where the team needs to focus on because the class library model will be supported in .NET 6. 

## What should I do?
The new model is the future of Azure Functions for .NET and I would recommend getting some experience with it, but be aware it is not ready yet for all workloads. Early adopters and teams that thrive on the edge of chaos will be fine with juggling different models at the same time. But if you move a bit more carefully or  rely on Durable Functions you have years of support ahead going from .NET core 3.1 to .NET 6. 

## Show me how
If you want to get started, go to this [readme](https://github.com/Azure/azure-functions-dotnet-worker/blob/main/README.md) on GitHub. In the same repository you can find a lot of [Samples](https://github.com/Azure/azure-functions-dotnet-worker/tree/main/samples) that the team created. 

> Be sure to check the [Known issues](https://github.com/Azure/azure-functions-dotnet-worker/wiki/Known-issues) prevent frustration.



[oscarvantol.nl](https://oscarvantol.nl)
