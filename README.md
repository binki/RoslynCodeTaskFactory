# Roslyn CodeTaskFactory

[![Build status](https://ci.appveyor.com/api/projects/status/4uy3aoqsa5lwi0en?svg=true)](https://ci.appveyor.com/project/CBT/roslyncodetaskfactory)
[![NuGet package](https://img.shields.io/nuget/v/RoslynCodeTaskFactory.svg)](https://www.nuget.org/packages/RoslynCodeTaskFactory)
[![NuGet](https://img.shields.io/nuget/dt/RoslynCodeTaskFactory.svg)](https://www.nuget.org/packages/RoslynCodeTaskFactory)


An MSBuild TaskFactory that uses the Roslyn compiler to generate .NET Standard task libraries which can be used by inline tasks.  It is a replacement of the built in CodeTaskFactory which uses CodeDom and does not work in .NET Core.

# Getting Started

To get started, add a PackageReference to the RoslynCodeTaskFactory package.

#### NuGet Package Manager UI
Search for `RoslynCodeTaskFactory`

#### NuGet Package Manager Console
```
Install-Package RoslynCodeTaskFactory
```

#### DotNet CLI
```
dotnet add package RoslynCodeTaskFactory
```
The package sets an MSBuild property named `$(RoslynCodeTaskFactory)` which should be used in a `<UsingTask />`.  The RoslynCodeTaskFactory is implemented exactly like the stock MSBuild [CodeTaskFactory](https://msdn.microsoft.com/en-us/library/dd722601.aspx?f=255&MSPPError=-2147217396) with the only difference being which `AssemblyFile` to use.

This example is the same as the one from MSDN with only the `AssemblyFile` attribute changed to use the RoslynCodeTaskFactory:
```xml
<Project ToolsVersion="12.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">  
  <!-- This simple inline task does nothing. -->  
  <UsingTask  
    TaskName="DoNothing"  
    TaskFactory="CodeTaskFactory"  
    AssemblyFile="$(RoslynCodeTaskFactory)"
    Condition=" '$(RoslynCodeTaskFactory)' != '' ">
    <ParameterGroup />  
    <Task>  
      <Reference Include="" />  
      <Using Namespace="" />  
      <Code Type="Fragment" Language="cs">  
      </Code>  
    </Task>  
  </UsingTask>  
</Project>  
```

## Hello World
Here is a more robust inline task. The HelloWorld task displays "Hello, world!" on the default error logging device, which is typically the system console or the Visual Studio Output window.
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">  
  <!-- This simple inline task displays "Hello, world!" -->  
  <UsingTask  
    TaskName="HelloWorld"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="$(RoslynCodeTaskFactory)"
    Condition=" '$(RoslynCodeTaskFactory)' != '' ">
    <ParameterGroup />  
    <Task>  
      <Using Namespace="System"/>  
      <Using Namespace="System.IO"/>  
      <Code Type="Fragment" Language="cs">  
<![CDATA[  
// Display "Hello, world!"  
Log.LogError("Hello, world!");  
]]>  
      </Code>  
    </Task>  
  </UsingTask>  
</Project>  
```
You could save the HelloWorld task in a file that is named HelloWorld.targets, and then invoke it from a project as follows.
```xml
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">  
  <Import Project="HelloWorld.targets" />  
  <Target Name="Hello">  
    <HelloWorld />  
  </Target>  
</Project>  
```

## API

The API is based on that offered by MSBuild’s `CodeTaskFactory`. For the most part, few or no changes need to be made to port from `CodeTaskFactory` to `RoslynCodeTaskFactory`.

### ParameterGroup

Specify parameters by specifying `<ParameterGroup/>` as a child of `<UsingTask/>`. Each element in `<ParameterGroup/>` is the name of the parameter. For Fragment and Method tasks, RoslynCodeTaskFactory automatically creates properties in the implicitly created class for each parameter. For Class tasks, you must declare matching public properties in class you write. For Class tasks, you may use `<Code AutoDetectProperties="True"/>` instead of providing `<ParameterGroup/>`. The attributes are:

* `ParameterType`: The default type is `System.String`. To create parameters accepting [Items](https://docs.microsoft.com/visualstudio/msbuild/msbuild-items) (i.e., lists of values), set this to `Microsoft.Build.Framework.ITaskItem[]`. To accept a single Item which allows you to inspect its metadat, set this type to `Microsoft.Build.Framework.ITaskItem`. For Class tasks, this must exactly match the type of the property declared in your class.
* `Output`: The default is `False`. Set to `True` to have MSBuild copy the property’s value back via the [Output](https://docs.microsoft.com/visualstudio/msbuild/output-element-msbuild) mechanism.
* `Required`: The default is `False`. Set to `True` for MSBuild to emit an error if the caller of your task does not provide a value for the property.

### Task

Specify `<Task/>` as a child of `<UsingTask/>`. Under `<Task/>`, you provide the following elements to define the task:

#### Reference

Specify a `<Reference Include="AssemblyName"/>` for each reference. Your `AssemblyName` may also be a full path to an assembly. Some examples of references include:

* `<Reference Include="Microsoft.Build.Tasks.Core"/>` to access built-in tasks such as `Copy`.

#### Using

Specify namespaces to import with `<Using Namespace="System"/>`. Mostly useful for Fragment and Method tasks because C# and VB.net require usings/imports to be outside of any class or method.

#### Code

Use `<Code/>` to set the language and source. For inline tasks, the body is used as the source code. You may want to utilize XML’s [`<![CDATA[ MyCode(); ]]>`](https://www.w3.org/TR/xml11/#sec-cdata-sect) feature to escape code. The following attributes may be used:

* `AutoDetectParameters`: Default false, only valid with `Type="Class"`. An extension (this functionality is not available in MSBuild’s `CodeTaskFactory` implementation). When set to `True`, causes the `<ParameterGroup/>` element to be ignored and parameters are instead automatically discovered from the compiled class using reflection just as if the class were loaded as an assembly instead of through `RoslynCodeTaskFactory`.
* `Language`: Default `CS`. Valid values: `CS`, `VB`. Specifies the language the inline task is provided in.
* `Source`: Default unspecified. If specified, refers to an external source code file which is loaded and compiled in Class mode. You may not specify any content to `<Code/>` if this is specified.
* `Type`: Default Fragment. May be one of Class, Method, or Fragment. For Class, you may specify the entire source file, including the `using`/`Import` lines. You must declare a public class with a name matching the `<UsingTask TaskName="ClassName"/>` and implement `ITask` or extend `Task`. For Method, you must specify an override of the [`Execute()`](https://docs.microsoft.com/dotnet/api/microsoft.build.framework.itask.execute?view=netframework-4.7.2) method. For Fragment, write the body of the task.

## Samples
Open [Samples.sln](https://github.com/jeffkl/RoslynCodeTaskFactory/blob/master/Samples/Samples.sln) to see more samples of the [inline task](https://github.com/jeffkl/RoslynCodeTaskFactory/blob/master/Samples/Directory.Build.targets#L5) in a .NET Framework and .NET Standard project.  
