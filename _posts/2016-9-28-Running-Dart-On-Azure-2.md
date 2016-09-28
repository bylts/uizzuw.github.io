---
layout: post
title: Running Dart on Microsoft Azure - Part 2 - Configuring Dart.Azure for custom solution directory structures
---

In the [previous part](http://www.underweasel.me/2016/09/21/Running-Dart-On-Azure.html "Running Dart on Microsoft Azure - underweasel.me") we have taken a brief look at setting up the [Dart.Azure](https://github.com/UizzUW/Dart.Azure "Dart.Azure extension on GitHub") extension. We have seen how we can run a client-server solution inside an Azure Web App, including Continuous Deployment. Unfortunately, that was merely scratching the surface. In that demo I have used a pre-built solution with a specific directory structure. Chances are, however, your solution won't have the same structure as I have chosen, but fear not! I have built the extension with this flexibility in mind.

Before we go any further, we must first understand how Kudu does its deployments. When you check-in to the deployment branch, Kudu uses webhooks to find out about that and trigger a new repository sync. It thus keeps a local copy of your repository, like you would manually do on any host machine. After syncing the repository, Kudu will trigger a deployment script - usually a Batch / Powershell script (although you'll see we can run anything really, but that on part 3 of this series).

Let's take a quick look at the [Dart.Azure deployment script](https://github.com/UizzUW/Dart.Azure/blob/master/Content/deploy.cmd "Dart.Azure deployment script"). When you install Dart.Azure, it will overwrite the default Kudu Sync deployment script (which is a fancy name for "copy-my-whole-repo-to-wwwroot"-script) with this one. That is why it is important to setup Continuous Deployment before the Dart.Azure extension - if you do it the other way around, Kudu will overwrite the Dart.Azure script. As I have said in the previous post, I will update this statement here once it will get fixed.

This script is written with some flexibility in mind, in the form of environment variables. For the server, for example, one interesting such environment variable is `DART_SERVER_SOURCE_PATH` :

```bat
REM ========== SERVER ==========

REM check if a custom server source path is set, or if the default one exists
REM DEPLOYMENT_SOURCE is your repo root
REM Dart.Azure sets DART_SERVER_SOURCE_PATH to 'server' by default
SET SERVER_SOURCE="%DEPLOYMENT_SOURCE%\%DART_SERVER_SOURCE_PATH%"

IF NOT EXIST %SERVER_SOURCE% (
    REM no server project found
    SET SERVER_SOURCE=
)

REM if we have found a server project to deploy
IF DEFINED SERVER_SOURCE (
    REM run pub get on project, xcopy it to production
)
```

You can use this environment variable to set the location of your server project source to anywhere in your repository - you can even set it to `.` if your whole repo is your server source (**CAREFUL - do NOT set it to an empty value** - the script will throw an error because it will see the variable as undefined).

Notice that some variable defaults are not defined here. While some, such as `DEPLOYMENT_SOURCE` come from Kudu, others, specific to the extension, are defined in the `applicationHost.xdt` [file](https://github.com/UizzUW/Dart.Azure/blob/master/Content/applicationHost.xdt "Dart.Azure applicationHost.xdt file") :

```xml
    <environmentVariables>
        <!-- ... more definitions -->
        <add name="DART_SERVER_SOURCE_PATH" value="server" />
        <add name="DART_CLIENT_SOURCE_PATH" value="client" />
    </environmentVariables>
```

Of course, the extension applies the same logic for the client, using the `DART_CLIENT_SOURCE_PATH` variable.

Using these two variables, you can configure the extension to only deploy a client solution, a server solution, or a mixed one. There are some more things to keep in mind about how the extension works :

* If the deployment script won't find any client or server projects at the defined (or default) locations, the script will carry on independently without failing.
* When deploying to production, Dart.Azure will copy **the entire repository** to the server location, in order to also keep any library dependencies you may have in your repository - I may optimize this in the future.
* Dart.Azure will run `pub build` on the client project and **will copy all contents of `DART_CLIENT_SOURCE_PATH\build` to `wwwroot`

Right now the extension is quite new and untested towards real-world solutions. It may thus feel a little bit hackish when setting up. This is why I am highly interested in everyone's feedback regarding it. It *can* be improved, but I am curious to know in which ways it would be the most useful - please let me know in the comments below!

Don't forget to keep an eye out for the last part when I will do a technical deep dive on how I have built the extension, from 0 to production. I will be writing it both so that experienced Azure Web App users will have a better understanding about what my extension is doing behind the scenes, as well as to give other people who want to port *their* favorite technology to Azure Web Apps the insight they need. [Twitter](https://twitter.com/uizzunderweasel) is a good place to be the first to find out when that comes out.

See you next time!