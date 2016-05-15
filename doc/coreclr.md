---
layout: doc
title: CoreCLR (.NET Core)
---

Npgsql 3.1 support CoreCLR (netstandard1.3).

Here is a sample project.json to get you started:

{% highlight json %}

{
  "buildOptions": {
    "emitEntryPoint": "true"
  },
  "dependencies": {
    "Npgsql" : "3.1.0-*"
  },
  "frameworks": {
    "net451": {
      "frameworkAssemblies": {
        "System.Data": { "version": "4.0.0.0", "type": "build" }
      }
    },
    "netcoreapp1.0": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "version": "1.0.0-*",
          "type": "platform"
        }
      }
    }
  }
}

{% endhighlight %}

