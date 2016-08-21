---
layout: post
title: Running ScriptCS in the cloud with WebAPI and Powershell
---

One of the funnest things to mess around with lately has been ScriptCS. From doing simple scripts, to creating the random data generator API that is BaconAPI, it performed quite well. The thing I like the most about it is that I can do things FAST, yet C#-level COMPLEX.

However one thing that makes it a bit unaccessible is that you have to install it each time you're on a new computer, even if all you want to do is run a simple script, once. Wouldn't it be nice if you could have your own script running server somewhere in the clouds? Oh wait...

So I looked up the current available hosting methods and I found out they have a C# API for that - convenient! What's even more convenient is that you can even use it with WebAPI - just like they show in this sample on GitHub.

I downloaded and cleaned the source code of all the huge, unnecessary pile of MVC / front-end code that they seem to have left in the sample for some reason, and just left in the WebAPI bits. Then I fired up a new App Service in Azure and deployed to it.

I then found myself writing a quick script deployer, again using ScriptCS - just for lulz - because obviously you'd have to install ScriptCS to use that. I then thought of converting it to a pre-compiled .exe, but that still wasn't good enough. So I turned to Powershell and wrote this puppy :

```powershell
param (
    [string]$server = "http://myawesomescriptrunner.azurewebsites.net",
    [string]$script = "script.csx"
)

$scriptContents = Get-Content $script

$headers = @{}
$headers["Content-Type"] = "application/json"

$body = "{ `"Code`":`"${scriptContents}`" }"

$result = Invoke-WebRequest -Uri $server -Headers $headers -Method POST -Body $body

echo $result.Content
```

As you can see, I've exposed the $server and $script parameters, but also set their defailts for convenience.

Then I wrote this :

```csharp
(new Random()).Next(0,1000);
```

And then ran this puppy :

```powershell
PS C:\.\csxrunner
130
PS C:\.\csxrunner
444
PS C:\.\csxrunner
208
PS C:\.\csxrunner
277
PS C:\.\csxrunner
152
PS C:\_
```

So you can imagine how fun this would be if you could also store your scripts and call them whenever you need them, without even needing to oepn up a text editor.

Challenge accepted?
Sure thing.