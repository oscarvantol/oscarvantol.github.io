[< back](index)

<sub>blogged: 2020.02.20</sub>

# Writing a custom task for Azure Devops Pipelines

Do you and your team sometimes wine about some things that 'we should really do at some point'? We had one of those, we are building really cool software at Virtual Vaults using Dotnet Core, Angular and all the Azure toys. We use Azure DevOps and have a lot of common nuget packages that we share internally using the Artifacts Feed. Everytime someone changes a package he or she will have forgot to bump the version at some point and a discussion starts how we should automate this and have a good branching strategy. But after it works we forget and move on until we bump into this again.

## Enough, let's do this

I was sick of hearing this over and over again "let's settle this once and for all". After some quick conversations with coworkers about how they did versioning in the past and settling on what we really wanted, I started. I found a lot of extensions in the [marketplace](https://marketplace.visualstudio.com) but none of them exactly did what I wanted. I started to create an extensive yml build file to try to do what I had in my head but it quickly became a complicated mess. I decided to write my own extension for a custom build task.

## Building an extension

Because I created a simple build task before I knew it would be pretty straigh forward. This was both true and untrue, the [documentation](https://docs.microsoft.com/en-us/azure/devops/extend/get-started/node?view=azure-devops) is really good and I had my basic logic running very quickly. But to get the thing properly packaged including what was needed from the node_modules had me googling for a while. Lots of people writing extensions have their code on GitHub and this helped me figuring out what I did wrong.

## Using the extension

If you upload your extension in the marketplace you can share it with your Azure DevOps organisation. Now I was ready to start testing it in the context of a build. I struggled a bit with setting up the permissions correctly but it worked like designed and I had an example pipeline running. 

Finally I added the custom task to the pipeline of the package I was working on during 'normal hours' and told everyone to have fun with it.

## Done, or maybe not?

Noooooo! Not done, I needed to write some documentation on our wiki or at least some comments in the yml file because otherwise this will get lost. 
**But if I am doing this anyway, let's do it right!** So I pushed to [sourcecode](https://github.com/oscarvantol/azure-pipelines-version-increment) to GitHub and wrote some documentation. Then I wanted to make the [extension](https://marketplace.visualstudio.com/items?itemName=ovantol.version-increment)  publicly available on the marketplace. This meant some more reading and adding information to my profile and package. And as a last step I am writing this, a small blog that describes my saturday evening ;)


By adding these extra steps:
* I forced myself to write some documentation and clean up
* I might help someone (or myself) write an extension in the future
* I might help my future self in need
* I might actually have someone else using use my extension!!!


[oscarvantol.nl](https://oscarvantol.nl)
