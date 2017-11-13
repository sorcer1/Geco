# Geco, (Ge)nerator (Co)nsole
Simple code generator based on a console project, running on .Net core and using C# interpolated strings.

Geco runs on .Net Core 2.0. [Download and Install](http://dot.net)

What's the reasoning behind this utility?

One of the most popular code generation tools for Visual Studio over the years was T4 (Text Template Transformation Toolkit), which saw a great deal of success for simple to complex code genration tasks over the years.
Scott Hanselman has a nice (blog post)[https://www.hanselman.com/blog/T4TextTemplateTransformationToolkitCodeGenerationBestKeptVisualStudioSecret.aspx] about it. However since the advent of **.Net Core** 
and .Net going cross platform there isn't a simple way to generate code or other artifacts that will can also work in Visual Studio Code (or other code editors).

This is where the idea for this simple utility came from. Have a small utility for code generation that is **debuggable** *(one of the long standing issues of T4)*, provides some level of **intellisense**, can be run at build time,
is higly customizable, generic and has as small as possible footprint.

## Installation

Geco can be installed as `dotnet new` template. Run the following command on a console or terminal:

```Batchfile
dotnet new -i iQuarc.Geco.CSharp
```

Then create a new project using the newly installed template:

```Batchfile  
dotnet new geco
```

## Code Generation examples

Bellow are some snippets from the included code generation tasks that are in the default Geco package.

Next snippet is from the included Entity Framework Core Reverse model generator, which exemplifies part of the code generation:

![Geco Preview1](https://github.com/iQuarc/Geco/blob/dev/GecoResources/PreviewImage.JPG?raw=true)

This snippet is from theSQL Seed Scrip Generator, which generates SQL Merge scripts which can be used to Seed data:

![Geco Preview2](https://github.com/iQuarc/Geco/blob/dev/GecoResources/PreviewImage2.JPG?raw=true)

Next screen shot shows geco running in interactive mode (as a Console App), and the menu that is presented to the user:

![Geco Preview3](https://github.com/iQuarc/Geco/blob/dev/GecoResources/PreviewImage3.JPG?raw=true)


## Description

Geco uses C# 6.0 string interpolation as a template engine for code generation, in order to allow:

 - Easy customization of templates (Simply edit the .cs file)
 - Easy debugging (Place a breakpoint an run)
 - Easy extensibility (Add a new task by creating a new C# class and implement a simple interface `IRunnable`)
 
Geco uses task discovery at runtime and each task is configured using Dependency Injection. The generation tasks can be run during build or from interactive mode (Debug Run).

## Included generators

The following generator tasks are included in current version.

 - Entity Framework Reverse model generator
 - SQL Seed data script generator (Generates MERGE scripts)
 - SQL Script runner
 - Database cleaner

## Customizing

1. The first customization option involves changing task parameters from `appsettings.json` configuration file. Each geco task defines it's own set of parameters (see the corresponding Options class):

```JSon
{
    "ConnectionStrings": {
        "DefaultConnection": "Integrated Security=True;Initial Catalog=AdventureWorks;Data Source=.\\SQLEXPRESS;"
    },
    "RunAtBuildTasks": [
        "Generate EF Core model"
    ],
    "Tasks": [
        {
            "Name": "Generate EF Core model",
            "TaskClass": "Geco.Database.EntityFrameworkCoreReverseModelGenerator",
            "BaseOutputPath": "..\\..\\Generated\\Model\\",
            "OutputToConsole": "false",
            "CleanFilesPattern": "*.cs",
            "Options": {
                "ConnectionName": "DefaultConnection",
                "Namespace": "Model",
                "OneFilePerEntity": true,
                "JsonSerialization": true,
                "GenerateComments": true,
                "UseSqlServer": true,
                "ConfigureWarnings": true,
                "GeneratedCodeAttribute": false,
                "NetCore": true,
                "ContextName":"AdventureworksContext"
            }
        }
    ]
}
```

2. By customizing existing templates directly.

3. By creating new code generation tasks as a C# class which implements the `IRunnable` interface.

`IRunnable` is a single method interface containing the `Run` method:

 ```CSharp
    /// <summary>
    /// Represents a runable task, for code generation or purposes or others
    /// </summary>
    public interface IRunnable
    {
        /// <summary>
        /// Invoked when this task is executed
        /// </summary>
        void Run();
    }   
 ```
 or by deriving from one of helper base classes `BaseGenerator` or `BaseGeneratorWithMetadata`.

A code generation class can be accompanied by an Option POCO class to hold options which can be customized from `appsettings.json`

Example:

```CSharp
    public class SeedDataGeneratorOptions
    {
        public string ConnectionName { get; set; }
        public string OutputFileName { get; set; }
        public List<string> Tables { get; } = new List<string>();
        public string TablesRegex { get; set; }
        public List<string> ExcludedTables { get; } = new List<string>();
        public string ExcludedTablesRegex { get; set; }
        public int ItemsPerStatement { get; set; } = 1000;
    }
        /// <summary>
    /// Generates seed scripts with merge statements for (Sql Server)
    /// </summary>
    [Options(typeof(SeedDataGeneratorOptions))]
    public class SeedDataGenerator : BaseGeneratorWithMetadata
    {
    // ...
    }
    
```

## Version History

**Version 1.0.0.2**

- Changed user confirmation mechanism
- Added user confirmation for file cleaning
- Added more options for color output to console, formattable strings with tuple (value, ConsoleColor) parameters

**Version 1.0.0.1**

- First release as a `nuget dotnet` new template

**Version 1.0.0.0-beta**

- Initial Version
- Contains code generation core functionality and SimpleMetadata model for databases
