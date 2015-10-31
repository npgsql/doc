---
layout: doc
title: CoreCLR (.NET Core)
---

Npgsql's current development branch (representing Npgsql 3.1) fully support CoreCLR. The current stable version (3.0) does not.
Alpha nuget packages of Npgsql 3.1 are occasionally released to nuget.org and can be used to test CoreCLR support.

Here is a sample project.json to get you started:

{% highlight json %}

{
  "version": "1.0.0-*",
  "description": "Sample Npgsql CoreCLR app",
  "authors": [ "roji" ],
  "tags": [ "" ],
  "projectUrl": "",
  "licenseUrl": "",

  "dependencies": {
    "npgsql": "3.1.0-*"
  },

  "frameworks": {
    "dnx451": {
      "frameworkAssemblies": {
        "System.Data": "4.0.0.0"
      }
    },
    "dnxcore50": {
      "dependencies": {
        "Microsoft.CSharp": "4.0.1-*",
        "System.Collections": "4.0.11-*",
        "System.Console": "4.0.0-*",
        "System.Linq": "4.0.1-*",
        "System.Threading": "4.0.11-*",
        "System.Data.Common": "4.0.1-*"
      }
    }
  }
}

{% endhighlight %}

