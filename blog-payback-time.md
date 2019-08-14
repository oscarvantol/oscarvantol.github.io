# It's payback time

*Building software changes all the time, for me it started with writing desktop apps that you would put on a floppy for someone to use.*

More than 20 years later I am working with a team at Virtual Vaults building SAAS by using PAAS and all the serverless goodies in Azure. Not only infrastructure 
and the delivery to the end user changed, but also the way we are able to continuously create, test and ship features with multiple people working closely together.

A big part of being able to build complex systems fast is because we can build on top of layers of abstraction. 
These days you just don’t build everything from scratch, you use services, packages. **If you would look at your entire stack, 
ever wonder how little code that is executed, is actually yours?** And how much of the external code is a library that you are paying for? 
I am betting that just like the software I am working on, most of the external software is pulled into your project as a package via NuGet or NPM. 
The majority of these packages have their source code in a Github project and have an open source license.

**So the foundations of the software we are making money with is made available for free. Create something for free, who would do that?**

Well, there are a few different groups doing this. First of all, enthiousiasts that started something to learn or make their life a bit easyer. Some of them grow their career because of the project and become trainers, speakers or specialized consultants. We also see big tech companies like Google and Microsoft pushing a lot of open source, in a lot of cases it will connect people to their platform making it a solid strategy. **But what about all the other businesses that build software, are we also giving back instead of only consumint and should we?**

Yes we should, and some of do! But how can we do that, not everyone at the office will be amused if we would fill our day with hobby projects and we can certainly not publish all our source code to the world. There are however a few options to balance it out. 

* The first one is obvious, when finding an issue or missing a feature in something you are using and you are able to fix it, 
do it and make the pull request, don’t build your internal version and keep it to yourself. 

* Another option, and maybe the one that is the most comfortable one starting with is documentation. 
* A lot of projects also accept changes in their docs, from simple things like correcting spelling or grammar to adding some examples. 
* You can also create a blogpost where you write down how you used it in your scenario, it can be useful for new team members but even for your future self.

* One of the more challenging options is creating and maintaining projects that you use specifically in your software. 
This will raise some questions, after all someone is paying the bill for this coding time, "is this not our intellectual property?". 
If it is something you need to create anyway, it is generic enough that other people might be interested in to use and contribute back into, it could be really interesting

*Note: There is a thing to be very aware of, don't let your project become a security issue. If you share or contribute to an open source project you are also letting the world know that you are using it. Make sure nothing gets into the libraries you use that you do not want in there, but this is already the case with everything you are using.*

I wrote this because I believe that not everyone in (tech)companies is aware that their development teams and their products are powered by the vast amount of communities and heroes working on pet projects. **If we increase contributing and sharing in our cultures I believe we can achieve even more.**

2019.08.14

