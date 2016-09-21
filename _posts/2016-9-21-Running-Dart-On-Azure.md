---
layout: post
title: Running Dart on Microsoft Azure - Part 1
---

### What is Dart and why have I done this?

If you have read pretty much any of my other articles on this blog, you would be somewhat aware of the undying hate I have for Javascript. While it seems like the whole world is trying so hard to show that Javascript does not suck as a programming language, they have yet to convince me. I've seen it all! I've seen node, I've seen Angular, I've seen Typescript, I've seen React, Polymer, Knockout, Ember, Gulp, Grunt, truth.js and many, many other frameworks, no matter big or small, that try to move mountains, while standing on the shoulders of such a fragile language. Do not get me wrong, I like these frameworks and what they are trying to achieve, but can't we develop them by using better technologies than such a simple, error-prone, anti-architectural, primitive language?

Luckily, I am not the only one feeling the need for a change! And funny enough, the ones that came to my rescue were actually **Google** - yes, **Google**! Probably the most heavily JS-dependant IT company in this world. If *they* are looking for an alternative to JS, shouldn't you start asking yourself some questions?

If I haven't completely thrown you off with the last two paragraphs, thank you for being such a patient, reasonable human being. From now on, I will refrain from such affirmations, and instead of talkong crap about JS, I will focus on its new, way more modern alternative : **Dart**.

[Dart](https://www.dartlang.org/ "Official Dart site") is a programming language mainly developed by Google, whose main goal is to be an alternative to JS. It is not just a programming language, but a completely new technology - it has got its own runtime and it runs on Windows, macOS *and* Linux! It can be used in place of JS in websites for DOM manipulation, AJAX and any other JS-like task for that matter. The gotcha here is that no mainstream browser supports Dart. Not Firefox, not Chrome, not Safari and deffinitely not Edge or IE.

Ludicrous! That's like the entire web population! some would say. Actually, the only browser that natively supports Dart is Dartium, a Chromium fork with a built-in DartVM. The trick, however, is that **Dart compiles to JS** - and before you leave away in disappointment, shouting "IT'S TYPESCRIPT ALL OVER AGAIN!", please calm down and allow me to enlighten you as to why this isn't actually end of the world.

Modern browsers today only support JS. You CANNOT use other languages in place of JS because there is no runtime to understand and execute code written in that language. If you want to support another runtime, you would have to create a separate version of a browser an inject the runtime into it, but that is useless because you (probably) won't get your user base to change their browser merely for running your app. That is why the only way Dart can work on modern browsers is to compile to JS. But is that overhead truly worth it? Why not write the JS code in the first place? There are three main reasons for that :

* Dart is a truly OOP language
* Dart has its own runtime
* Dart's syntactical fullness brings back proper error handling

**Dart is not Typescript**. It is not a syntactical superset of JS, nor does it include its whole base syntax. **It instead is a completely separate runtime**. Yes, that's right. Written in C++ and everything! Check out the [source code](https://github.com/dart-lang/sdk "dart-sdk source code") if you don't believe me.

Developing against a new runtime changes the rules. It may not look like it at first, but it really does. Real developers can "*feel*" the difference between a good and a crappy runtime. And while the Dart runtime may not be perfect, it certainly packs some really intriguing things - they've got [threading](https://lucamezzalira.com/2013/06/11/isolates-how-to-work-with-multithreading-in-dart/ "Isolates in Dart - Luca Mezzalira") and [async](https://www.youtube.com/watch?v=MUDOIAssBDs "Async in Dart - Dart Dev Summit 2015") right, for example.

I will not waste any more time preaching the awesomeness of Dart - this is not the scope of this post. If you want to go deeper into Dart, I recommend taking a look at some [tutorials](https://www.dartlang.org/tutorials "Dart Tutorials"). You can find lots of them on The Internet, from good ol' DOM manipulation to heavy native scripting. Now, however, let's come back to the main topic.

### Running Dart in production today is a pain

It truly is. There are not many viable hosting solutions out there - Heroku and Firebase are the most common, but I find them very limitating, at least from a free standpoint. That is one aspect I hate about cloud computing today - it is so bloody expensive just to play around with it for non-production means!! 

Luckily, Microsoft have quite an accessible solution to this in their Azure catalog. [Azure Web Apps](https://azure.microsoft.com/en-us/services/app-service/web/ "Azure Web Apps") are a PaaS cloud service that basically hosts web applications, both front-end and back-end. They mostly support ASP.NET and Node solutions officially, but *anyone* can write an extension to offer support for their favorite technology (which, in fact, is also what I have done with Dart). The best part about Azure Web Apps is that they have a free tier - meaning you can mess around and try out your solution all you want, as long as you don't need huge ammounts of storage and traffic quota.

Creating a Dart extension for Azure Web Apps came as a natural challenge for me the minute I found out there is already a [Jekyll extension](https://github.com/SyntaxC4-MSFT/JekyllExtension "JekyllExtension by SyntaxC4") in place. For those that do not know what Jekyll is, it is a framework written in Ruby for serving static websites written in [Markdown](https://en.wikipedia.org/wiki/Markdown) - which [this very blog is also based on](https://github.com/UizzUW/uizzuw.github.io). So if someone managed to get the madness that is Ruby to run on Azure Web Apps, porting literally anything else should be a breeze, right? Right?

### Enter Dart.Azure

After a couple of evenings (more than I had planned initially), [Dart.Azure](https://github.com/UizzUW/Dart.Azure "Dart.Azure on Github") finally came to be. I have looked into ways to make it as seamless and easy to use as possible, and I think I've finally hit a decent point in its development so that I can let it roam free into the [extension gallery](https://www.siteextensions.net/packages/Dart.Azure/ "Dart.Azure on SiteExtensions") and start asking for feedback. Let's try to setup an Azure Web App running a Dart solution from scratch. There are a few gotchas here and there, but I will point them all along the way. For this demo I will be using my [Dart number of the day](https://github.com/UizzUW/dart_notd "dart_notd on Github") sample solution - you can fork it and follow along if you'd like to. You also need to have an Azure subscription capable of creating at least a free tier Web App.

The last preparation note I am going to make is that we will also be using Kudu for some things. You can think of Kudu as the go-to admin tool / backdoor control panel for your Web App. Kudu is the component responsible with CD, extension management, and you can also use it for more advanced things, like writing and running Batch or Powershell scripts for your website. We will only be using it for CD and extensions in this article, though.

So assuming you've got all that in place, let's go ahead and get started. First of all, we need to provision a new Azure Web App.

Note that if you do not have an Azure subscription, you can try them for free for a limited trial time [here](https://azure.microsoft.com/en-us/services/app-service/web/ "Azure Web Apps"). They will give you a temporary Web App along with its URL that you can play with, in the form of **GUID.azurewebsites.net**. They give you a Git repository URL that you can push to in order to simulate CD, which is nice. You will also have access to Kudu via **GUID.scm.azurewebsites.net**, but **it will not have any authorization** so be careful who you share that link with during the trial. We will be using Kudu to install the Dart.Azure extension either if you're using a trial Web App or a free one provisioned from the portal, mainly because we need to bypass some imposed timeout, but more on that later.

![try azure web apps](/images/dart_azure_1/dart_azure_6.png "Provisioning a new Web App in Azure")

If, however, you are lucky enough to have an Azure subscription, this process will be way simpler :

![new azure web app](/images/dart_azure_1/dart_azure_1.png "Provisioning a new Web App in Azure")

Give it a name of your liking and provide it with a resource group and service plan. We will not be using Analytics, so untick that. I would also like it to get pinned to the dashboard. Click "Create" and wait for Azure to provision everything for you.

![azure web app settings](/images/dart_azure_1/dart_azure_2.png "Azure Web App Settings")

After the site is up and running, the first thing we need to do is **setup Continuous Deployment**. Right now it is required to do this **before** you install the extension, because Kudu will overwrite the custom deployment script injected by the extension if you setup CD afterwards. This will get fixed in a future version and I will make sure to update this post when that has been achieved.

Make sure you have set up your repository information correctly. I will be using my *dart_notd* repository, as I said, and will be deploying from the master branch.

![setting up continuous deployment](/images/dart_azure_1/dart_azure_3.png "Setting up CD for the Web App")

After you click OK, Kudu will go ahead and pull the entire selected branch of your repository. It will also push everything in it to the website's **wwwroot**. That is obviously not what we want, and it is happening because the default Kudu deployment script is being ran. Once installed, Dart.Azure will overwrite that with a proper Dart-speciffic deployment script I wrote. For now though, let it do its thing and make sure the source is being pulled successfully.

![successful repository sync](/images/dart_azure_1/dart_azure_4.png "Successful Kudu repository sync")

If all went well until now, don't feel too happy. Now's when the hacky stuff starts happening. We now need to install the Dart.Azure extension, which will in turn install the Dart SDK and setup proper solution deployment, environment variables, web server configuration and a couple of other things. If you've gone through the list of options on the left in your Web App blade, you might have come across an option called "Extensions" and would assume we will be using that in order to install Dart.Azure. We will *not* be using that, because **Azure has a timeout of one minute in place for the installation of any extension**. Dart.Azure takes a bit longer than that - it first needs to download the SDK and then to extract it. If you go ahead and try installing it using this method, you will get the following error after exeactly 60 seconds :

![timeout error](/images/dart_azure_1/dart_azure_8.png "Azure Web App extension WebApiClient timeout error")

After going through the depths of The Internet in search of a decent solution for this, I have found out that you can set the `SCM_COMMAND_IDLE_TIME` environment variable to go around this. **However, it seems this only works when you install extensions from the Kudu extension gallery and not the Azure portal too!**. So keep that in mind and go define that environment variable in your Web App settings. I have read the value is supposed to be in seconds, but I've set it to something that would work in milliseconds as well, just to be overly-superstitious about it.

![setting the kudu extension install timeout](/images/dart_azure_1/dart_azure_11.png "Setting the Kudu extension install timeout")

Now we're finally ready to install Dart.Azure! Open *Advanced Tools* from the options list on the left (or just search for *kudu* in the search box at the top), go to the *Site Extensions* tab, then to *Gallery* and search for *dart* :

![searching for the Dart.Azure extension](/images/dart_azure_1/dart_azure_5.png "Searching for the Dart.Azure extension")

Click on the + icon to install it and wait for about two minutes. After the install finished, you will be prompted to restart the website - do that. This is required for the environment variables needed by Dart.Azure to be set. You can also restart it from the app's blade if you would prefer that :

![restart azure web app](/images/dart_azure_1/dart_azure_9.png "Restarting the Azure Web App")

One more step and we're done! Go to *Deployment options* and re-sync, so that the new deployment script kicks in and does the proper deployment. Don't worry, it's written in such a way that it cleans up after the initial Kudu deployment mess. If everything goes well you should see the green check mark on the latest deployment once again. If not, click on it and look in the deployment task log. It should be verbose enough to give you an idea about what might have happenned.

![resync azure web app](images/dart_azure_1/dart_azure_10.png "Resyncing the Azure Web App")

The moment of truth! Go to your website. If you have used *dart_notd* as well, you should see something like this :

![dart_notd running on azure](/images/dart_azure_1/dart_azure_7.png "Dart Note of the Day running on Azure")

**THat's it!** you are now running Dart on Microsoft Azure! Isn't that insane!?

If something went wrong along the way, please post in the comments below. The extension has just been released and is in a very alpha stage. However, I have spent a lot of time developing it and I most likely have an idea what your problem may be. I will also be posting two more parts to this - one explaining how you can configure the extension and structure your solution for compatibility, and one for the advanced Kudu kiddies, explaining every small detail regarding the development process - how I got it to work, from 0 to production.

Hope you guys like the extension! I am eager to hear your feedback - please let me know your thoughts, issues and suggestions either in the comments or on Twitter. Thank you!