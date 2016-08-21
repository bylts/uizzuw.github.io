---
layout: post
title: Serializing and deserializing C# Lambdas to and from JSON - Querying servers redefined
---

Some of you may know that recently I've launched this puppy called CleoDB. I did it on a whim, while teaching C# to a friend. I needed some simple database that could do caching, and SQL + EF was too much overhead at the time. This project is turning out to be quite the beast, actually, but that's not what this article is about.

I was looking at ways to expose CleoDB so that other systems could (remotely) interface with it. While I was researching that, a random crazy idea just popped up into my mind : can we serialize lambdas? No - let me do it properly :

Can we serialize C# lambdas into JSON? Apparently YES!



At first I thought to use the Roslyn APIs to dynamically rebuild it from a string, or some gimmick like that. Then I thought of JSON de/serialization. So I looked it up and came across this StackOverflow thread, which ultimately pointed me to Aq.ExpressionJsonSerializer - one of the most innovative and brilliant things I've seen lately! Check out the unit tests to see just how much this beast can do!

So I decided to test it myself. [Here's the Gist of it!](https://gist.github.com/UizzUW/945c5740c93ecbf505f789143734d22f)

First of all I defined this "JsonNetAdapter" that is basically a wrapper around JSON.NET with Aq.ExpressionJsonSerializer as its converter. I did it precisely as in the StackOverflow post :

```csharp
public static class JsonNetAdapter
{
    private static readonly JsonSerializerSettings _settings;

    static JsonNetAdapter()
    {
        var defaultSettings = new JsonSerializerSettings { TypeNameHandling = TypeNameHandling.Objects };
        defaultSettings.Converters.Add(new ExpressionJsonConverter(Assembly.GetAssembly(typeof(User))));
        _settings = defaultSettings;
    }

    public static string Serialize<T>(T obj) => JsonConvert.SerializeObject(obj, _settings);

    public static T Deserialize<T>(string json) => JsonConvert.DeserializeObject<T>(json, _settings);
}
```

Then I defined a test POCO :

```csharp
class User
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

And then wrote this test-ish Main :

```csharp
class Program
{
    static void Main(string[] args)
    {
        Expression<Func<User,bool>> lambda = x => x.Age > 20;
        var serializedLambda = JsonNetAdapter.Serialize(lambda);
        var deserializedLambda = JsonNetAdapter.Deserialize<Expression<Func<User, bool>>>(serializedLambda);
        var users = new List<User>
        {
            new User { Name = "Bobbie", Age = 15 },
            new User { Name = "Angie", Age = 25 },
            new User { Name = "Carol", Age = 17 },
            new User { Name = "Billy", Age = 34 },
            new User { Name = "Patrick", Age = 20 },
        };
        var gtn20 = users.Where(deserializedLambda.Compile());
        gtn20.ToList().ForEach(u => Console.WriteLine(u.Name));
        Console.ReadLine();
    }
}
```

The result was unbelievable - IT WORKS.

```CMD
Angie
Billy
```

I don't want to be dramatic about it but this changes EVERYTHING. It redefines what you can do with IQueryables and remote procedures, and if done well, it can have an unbelievable impact.

So let's do that. Stay tuned.
