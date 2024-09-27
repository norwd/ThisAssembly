![Icon](img/icon-32.png) ThisAssembly
============

[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.svg?color=green)](https://www.nuget.org/packages/ThisAssembly)
[![License](https://img.shields.io/github/license/devlooped/ThisAssembly.svg?color=blue)](https://github.com//devlooped/ThisAssembly/blob/main/license.txt)
[![Build](https://github.com/devlooped/ThisAssembly/workflows/build/badge.svg?branch=main)](https://github.com/devlooped/ThisAssembly/actions)

<!-- include https://github.com/devlooped/.github/raw/main/sponsorlink.md -->
*This project uses [SponsorLink](https://github.com/devlooped#sponsorlink) 
and may issue IDE-only warnings if no active sponsorship is detected.*

<!-- https://github.com/devlooped/.github/raw/main/sponsorlink.md -->
<!-- #meta -->
Expose project and assembly level information as constants in the ThisAssembly 
class using source generators powered by Roslyn.

The main generated entry point type is `ThisAssembly` in the global namespace (by default), 
and is declared as partial so you can extend it too with manually created members.

> Use `$(ThisAssemblyNamespace)` MSBuild property to set a root namespace for `ThisAssembly`.

Each package in turn extends this partial class to add their own nestes types 
and members.

The [ThisAssembly](https://nuget.org/packages/ThisAssembly) meta-package includes 
all the other packages for convenience.

> For now, ThisAssembly only generates C# code.
<!-- #meta -->

## ThisAssembly.AssemblyInfo
[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.AssemblyInfo.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly.AssemblyInfo)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.AssemblyInfo.svg?color=green)](https://www.nuget.org/packages/ThisAssembly.AssemblyInfo)

<!-- include src/ThisAssembly.AssemblyInfo/readme.md#assembly -->
<!-- #assembly -->
This package generates a static `ThisAssembly.Info` class with public 
constants exposing the following attribute values generated by default for SDK style projects:

* AssemblyConfigurationAttribute
* AssemblyCompanyAttribute
* AssemblyTitleAttribute
* AssemblyDescriptionAttribute
* AssemblyProductAttribute
* AssemblyCopyrightAttribute

* AssemblyVersionAttribute
* AssemblyInformationalVersionAttribute
* AssemblyFileVersionAttribute

If your project includes these attributes by other means, they will still be emitted properly 
on the `ThisAssembly.Info` class.

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.AssemblyInfo.png)

<!-- #assembly -->
<!-- src/ThisAssembly.AssemblyInfo/readme.md#assembly -->

## ThisAssembly.Constants

[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.Constants.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly.Constants)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.Constants.svg?color=green)](https://www.nuget.org/packages/ThisAssembly.Constants)

<!-- include src/ThisAssembly.Constants/readme.md#constants -->
<!-- #constants -->
This package generates a static `ThisAssembly.Constants` class with public
constants for `@(Constant)` MSBuild items in the project.

```xml
  <ItemGroup>
    <Constant Include="Foo.Bar" Value="Baz" Comment="Yay!" />
    <Constant Include="Foo.Hello" Value="World" Comment="Comments make everything better 😍" />
  </ItemGroup>
```

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Constants.png)

These constants can use values from MSBuild properties, making compile-time values configurable 
via environment variables or command line arguments. For example:

```xml
  <PropertyGroup>
    <HttpDefaultTimeoutSeconds>10</HttpDefaultTimeoutSeconds>
  </PropertyGroup>
  <ItemGroup>
    <Constant Include="Http.TimeoutSeconds" 
              Value="$(HttpDefaultTimeoutSeconds)" 
              Type="int" 
              Comment="Default timeout in seconds for HTTP requests" />
  </ItemGroup>
```

The C# code could consume this constant as follows:

```csharp
public HttpClient CreateHttpClient(string name, int? timeout = default)
{
    HttpClient client = httpClientFactory.CreateClient(name);
    client.Timeout = TimeSpan.FromSeconds(timeout ?? ThisAssembly.Constants.Http.TimeoutSeconds);
    return client;
}
```

Note how the constant is typed to `int` as specified in the `Type` attribute in MSBuild. 
The generated code uses the specified `Type` as-is, as well as the `Value` attribute in that 
case, so it's up to the user to ensure they match and result in valid C# code. For example, 
you can emit a boolean, long, double, etc., but a `DateTime` wouldn't be a valid constant even 
if you set the `Value` to `DateTime.Now`. If no type is provided, `string` is assumed. Values 
can also be multi-line and will use [C# raw string literals](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/raw-string) 
if supported by the target language version (11+).

In this example, you could trivially change how your product behaves by setting the environment 
variable `HttpDefaultTimeoutSeconds` in CI. This is particularly useful for test projects, 
where you can easily change the behavior of the system under test without changing the code.


In addition to arbitrary constants via `<Constant ...>`, it's quite useful (in particular in test projects) 
to generate constants for files in the project, so there's also a shorthand for those:

```xml
  <ItemGroup>
    <FileConstant Include="@(Content)" />
  </ItemGroup>
```

Which results in:

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Constants2.png)

<!-- #constants -->
<!-- src/ThisAssembly.Constants/readme.md#constants -->

## ThisAssembly.Git

[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.Git.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly.Git)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.Git.svg?color=green)](https://www.nuget.org/packages/ThisAssembly.Git)

<!-- include src/ThisAssembly.Git/readme.md#git -->
<!-- #git -->
This package generates a static `ThisAssembly.Git` class with constants 
for the following Git properties from the current project:

* Commit
* Sha (first 9 chars from Commit)
* Root (normalized to forward slashes)
* Url (if PublishRepositoryUrl=true)
* Branch (from CI environment variables)

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Git.png)

This package relies on your project's installed
[Microsoft.SourceLink.*](https://www.nuget.org/packages?q=Microsoft.SourceLink) 
package reference according to your specific Git-based source control server 
(such as GitHub, Azure DevOps, BitBucket, etc).

> NOTE: from .NET 8 SDK onwards, SourceLink is included by default, so you 
> don't need to add it manually.

The `Branch` property is populated from environment variables provided 
by the currently supported CI systems: GitHub Actions, Azure DevOps, 
AppVeyor, TeamCity, Travis CI, Circle CI, GitLab CI, Buddy, and Jenkins.

Whenever the CI system provides a pull request number, the branch name is 
`pr[NUMBER]`, such as `pr123`. This makes it easy to use it as a semver 
metadata label.

> Note: by default, the values of these constants are populated during 
"real" builds (that is, not IDE/design-time builds used to populate 
intellisense). This is to avoid negatively affecting the editor's 
performance. This means, however, that the properties will seem to 
always be empty when inspecting them in the IDE (although never at 
run-time). If you want to force population of these values for 
design-time builds, set the `EnableSourceControlManagerQueries` property to `true`. 
This property is defined and documented by 
[dotnet/sourcelink](https://github.com/dotnet/sourcelink/blob/main/src/SourceLink.Common/build/Microsoft.SourceLink.Common.props#L14).

At the MSBuild level, targets can take a dependency on the provided 
`InitializeGitInformation` target, which sets the equivalent properties
named:

* RepositoryCommit
* RepositorySha
* RepositoryRoot
* RepositoryUrl
* RepositoryBranch

The names of these properties were chosen on purpose to match the 
properties used by [nuget pack](https://learn.microsoft.com/en-us/nuget/reference/msbuild-targets#pack-target) 
and [nugetizer](https://github.com/devlooped/nugetizer) to populate
the relevant package metadata. 

So if you have a GitHub repository, installing these three packages 
will ensure you have the proper metadata out of the box and the simplest 
packaging experience possible:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.SourceLink.GitHub" />
    <PackageReference Include="ThisAssembly.Git" />
    <PackageReference Include="NuGetizer" />
  </ItemGroup>
</Project>
```

<!-- #git -->
<!-- src/ThisAssembly.Git/readme.md#git -->

## ThisAssembly.Metadata

[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.Metadata.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly.Metadata)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.Metadata.svg?color=green)](https://www.nuget.org/packages/ThisAssembly.Metadata)

<!-- include src/ThisAssembly.Metadata/readme.md#metadata -->
<!-- #metadata -->
This package provides a static `ThisAssembly.Metadata` class with public 
constants exposing each `[System.Reflection.AssemblyMetadata(..)]` defined for 
the project.

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Metadata.png)

For an attribute declared (i.e. in *AssemblyInfo.cs*) like:

```csharp
[assembly: System.Reflection.AssemblyMetadataAttribute("Foo", "Bar")]
```

A corresponding `ThisAssembly.Metadata.Foo` constant with the value `Bar` is provided. 
The metadata attribute can alternatively be declared using MSBuild syntax in the project 
(for .NET 5.0+ projects that have built-in support for `@(AssemblyMetadata)` items):

```xml
  <ItemGroup>
    <AssemblyMetadata Include="Foo" Value="Bar" />
  </ItemGroup>
```

<!-- #metadata -->
<!-- src/ThisAssembly.Metadata/readme.md#metadata -->

## ThisAssembly.Project

[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.Project.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly.Project)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.Project.svg?color=green)](https://www.nuget.org/packages/ThisAssembly.Project)

<!-- include src/ThisAssembly.Project/readme.md#project -->
<!-- #project -->
This package generates a static `ThisAssembly.Project` class with public 
constants exposing project properties that have been opted into this mechanism by adding 
them as `ProjectProperty` MSBuild items in the project file, such as:

```xml
  <PropertyGroup>
    <!-- Some arbitrary MSBuild property declared somewhere -->
    <Foo>Bar</Foo>
  </PropertyGroup>
  <ItemGroup>
    <!-- Opt-in to emitting that property value as a constant in ThisAssembly.Project -->
    <ProjectProperty Include="Foo" />
  </ItemGroup>
```

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Project.png)

<!-- #project -->
<!-- src/ThisAssembly.Project/readme.md#project -->

## ThisAssembly.Resources

[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.Resources.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly.Resources)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.Resources.svg?color=green)](https://www.nuget.org/packages/ThisAssembly.Resources)

This package generates a static `ThisAssembly.Resources` class with public 
properties exposing shortcuts to retrieve the contents of embedded resources.

<!-- include src/ThisAssembly.Resources/readme.md#resources -->
<!-- #resources -->

This package generates a static `ThisAssembly.Resources` class with public 
properties exposing typed APIs to retrieve the contents of embedded resources.


```xml
  <ItemGroup>
    <EmbeddedResource Include="Content/Docs/License.md" />
  </ItemGroup>
```

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Resources.png)

Since markdown files are text files, the API will expose a `Text` property property 
for it that will read its content once and cache it:

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Resources2.png)

The `$(EmbeddedResourceStringExtensions)` MSBuild property allows customizing which 
file extensions get treated as text files. By default, it's defined as:

```xml
  <PropertyGroup>
    <EmbeddedResourceStringExtensions>.txt|.cs|.sql|.json|.md</EmbeddedResourceStringExtensions>
  </PropertyGroup>
```

You can append additional file extensions to this list, or override it completely.
The list must be pipe-separated.

You can always use the provided `GetStream` and `GetBytes` for more advanced scenarios (or for 
non-text resources).

Optionally, you can specify the `Kind` metadata for a specific `EmbeddedResource` you want 
treated as a text file:

```xml
    <EmbeddedResource Include="query.kql" Kind="Text" />
```

You can also add a `Comment` item metadata attribute, which will be used as the `<summary>` XML 
doc for the generated member.

## Customizing the generated code

The following MSBuild properties can be used to customize the generated code:

| Property                | Description                                                                                          |
|-------------------------|------------------------------------------------------------------------------------------------------|
| ThisAssemblyNamespace   | Sets the namespace of the generated `ThisAssembly` root class. If not set, it will be in the global namespace. |
| ThisAssemblyVisibility  | Sets the visibility modifier of the generated `ThisAssembly` root class. If not set, it will be internal. |

<!-- #resources -->
<!-- src/ThisAssembly.Resources/readme.md#resources -->

## ThisAssembly.Strings

[![Version](https://img.shields.io/nuget/vpre/ThisAssembly.Strings.svg?color=royalblue)](https://www.nuget.org/packages/ThisAssembly.Strings)
[![Downloads](https://img.shields.io/nuget/dt/ThisAssembly.Strings.svg?color=green)](https://www.nuget.org/packages/ThisAssembly.Strings)

<!-- include src/ThisAssembly.Strings/readme.md#strings -->
<!-- #strings -->

This package generates a static `ThisAssembly.Strings` class with public 
constants exposing string resources in .resx files or methods with the right number of 
parameters for strings that use formatting parameters. 

![](https://raw.githubusercontent.com/devlooped/ThisAssembly/main/img/ThisAssembly.Strings.gif)

In addition, it groups constants and methods in nested classes according to an optional 
underscore separator to organize strings. For example, *User_InvalidCredentials* can be
accessed with *ThisAssembly.Strings.User.InvalidCredentials* if it contains a simple string, 
or as a method with the right number of parametres if its value has a format string.

Given the following Resx file:

| Name                          | Value                                 | Comment           |
|-------------------------------|---------------------------------------|-------------------|
| Infrastructure_MissingService | Service {0} is required.              | For logging only! |
| Shopping_NoShipping           | We cannot ship {0} to {1}.            |                   |
| Shopping_OutOfStock           | Product is out of stock at this time. |                   |
| Shopping_AvailableOn          | Product available on {date:yyyy-MM}.  |                   |

The following code would be generated:

```csharp
partial class ThisAssembly
{
    public static partial class Strings
    {
        public static partial class Infrastructure
        {
            /// <summary>
            /// For logging only!
            /// => "Service {0} is required."
            /// </summary>
            public static string MissingService(object arg0)
                => string.Format(CultureInfo.CurrentCulture, 
                    Strings.GetResourceManager("ThisStore.Properties.Resources").GetString("MissingService"), 
                    arg0);
        }

        public static partial class Shopping
        {
            /// <summary>
            /// => "We cannot ship {0} to {1}."
            /// </summary>
            public static string NoShipping(object arg0, object arg1)
                => string.Format(CultureInfo.CurrentCulture, 
                    Strings.GetResourceManager("ThisStore.Properties.Resources").GetString("NoShipping"), 
                    arg0, arg1);

            /// <summary>
            /// => "Product is out of stock at this time."
            /// </summary>
            public static string OutOfStock
                => Strings.GetResourceManager("ThisStore.Properties.Resources").GetString("OutOfStock");

            /// <summary>
            /// Product available on {date:yyyy-MM}.
            /// </summary>
            public static string AvailableOn(object date) 
                => string.Format(CultureInfo.CurrentCulture, 
                    Strings.GetResourceManager("ThisAssemblyTests.Resources").GetString("WithNamedFormat").Replace("{date:yyyy-MM}", "{0}"), 
                    ((IFormattable)date).ToString("yyyy-MM", CultureInfo.CurrentCulture));
        }
    }
}
```

## Customizing the generated code

The following MSBuild properties can be used to customize the generated code:

| Property                | Description                                                                                          |
|-------------------------|------------------------------------------------------------------------------------------------------|
| ThisAssemblyNamespace   | Sets the namespace of the generated `ThisAssembly` root class. If not set, it will be in the global namespace. |
| ThisAssemblyVisibility  | Sets the visibility modifier of the generated `ThisAssembly` root class. If not set, it will be internal. |

<!-- #strings -->
<!-- src/ThisAssembly.Strings/readme.md#strings -->

<!-- include src/visibility.md -->
## Customizing the generated code

Set the `$(ThisAssemblyNamespace)` MSBuild property to set the namespace of the 
generated `ThisAssembly` root class. Otherwise, it will be generated in the global namespace.

The generated root `ThisAssembly` class is partial and has no visibility modifier by default, 
making it internal by default in C#.

You can set the `$(ThisAssemblyVisibility)` MSBuild property to `public` to make it public. 
This will also change all constants to be static readonly properties instead. 

Default:
```csharp
partial class ThisAssembly
{
    public partial class Constants
    {
        public const string Hello = "World";
    }
}
```

In this case, the compiler will inline the constants directly into the consuming code at 
the call site, which is optimal for performance for the common usage of constants.

Public:
```csharp
public partial class ThisAssembly
{
    public partial class Constants
    {
        public static string Hello => "World";
    }
}
```

This makes it possible for consuming code to remain unchanged and not require 
a recompile when the the values of `ThisAssembly` are changed in a referenced assembly.

If you want to keep the properties as constants, you can instead extend the generated 
code by defining a another partial that can modify its visibility as needed (or add 
new members). 

```csharp
// makes the generated class public
public partial ThisAssembly 
{
    // Nested classes are always public since the outer class 
    // already limits their visibility
    partial class Constants 
    {
        // add some custom constants
        public const string MyConstant = "This isn't configurable via MSBuild";

        // generated code will remain as constants
    }
}
```

<!-- src/visibility.md -->

# Dogfooding

[![CI Version](https://img.shields.io/endpoint?url=https://shields.kzu.io/vpre/Stunts/main&label=nuget.ci&color=brightgreen)](https://pkg.kzu.io/index.json)
[![Build](https://github.com/devlooped/ThisAssembly/workflows/build/badge.svg?branch=main)](https://github.com/devlooped/ThisAssembly/actions)

We also produce CI packages from branches and pull requests so you can dogfood builds as quickly as they are produced. 

The CI feed is `https://pkg.kzu.io/index.json`. 

The versioning scheme for packages is:

- PR builds: *42.42.42-pr*`[NUMBER]`
- Branch builds: *42.42.42-*`[BRANCH]`.`[COMMITS]`


<!-- include https://github.com/devlooped/sponsors/raw/main/footer.md -->
# Sponsors 

<!-- sponsors.md -->
[![Clarius Org](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/clarius.png "Clarius Org")](https://github.com/clarius)
[![Kirill Osenkov](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/KirillOsenkov.png "Kirill Osenkov")](https://github.com/KirillOsenkov)
[![MFB Technologies, Inc.](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/MFB-Technologies-Inc.png "MFB Technologies, Inc.")](https://github.com/MFB-Technologies-Inc)
[![Stephen Shaw](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/decriptor.png "Stephen Shaw")](https://github.com/decriptor)
[![Torutek](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/torutek-gh.png "Torutek")](https://github.com/torutek-gh)
[![DRIVE.NET, Inc.](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/drivenet.png "DRIVE.NET, Inc.")](https://github.com/drivenet)
[![Ashley Medway](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/AshleyMedway.png "Ashley Medway")](https://github.com/AshleyMedway)
[![Keith Pickford](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/Keflon.png "Keith Pickford")](https://github.com/Keflon)
[![Thomas Bolon](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/tbolon.png "Thomas Bolon")](https://github.com/tbolon)
[![Kori Francis](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/kfrancis.png "Kori Francis")](https://github.com/kfrancis)
[![Toni Wenzel](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/twenzel.png "Toni Wenzel")](https://github.com/twenzel)
[![Giorgi Dalakishvili](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/Giorgi.png "Giorgi Dalakishvili")](https://github.com/Giorgi)
[![Uno Platform](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/unoplatform.png "Uno Platform")](https://github.com/unoplatform)
[![Dan Siegel](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/dansiegel.png "Dan Siegel")](https://github.com/dansiegel)
[![Reuben Swartz](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/rbnswartz.png "Reuben Swartz")](https://github.com/rbnswartz)
[![Jacob Foshee](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/jfoshee.png "Jacob Foshee")](https://github.com/jfoshee)
[![](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/Mrxx99.png "")](https://github.com/Mrxx99)
[![Eric Johnson](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/eajhnsn1.png "Eric Johnson")](https://github.com/eajhnsn1)
[![Ix Technologies B.V.](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/IxTechnologies.png "Ix Technologies B.V.")](https://github.com/IxTechnologies)
[![David JENNI](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/davidjenni.png "David JENNI")](https://github.com/davidjenni)
[![Jonathan ](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/Jonathan-Hickey.png "Jonathan ")](https://github.com/Jonathan-Hickey)
[![Charley Wu](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/akunzai.png "Charley Wu")](https://github.com/akunzai)
[![Jakob Tikjøb Andersen](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/jakobt.png "Jakob Tikjøb Andersen")](https://github.com/jakobt)
[![Seann Alexander](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/seanalexander.png "Seann Alexander")](https://github.com/seanalexander)
[![Tino Hager](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/tinohager.png "Tino Hager")](https://github.com/tinohager)
[![Mark Seemann](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/ploeh.png "Mark Seemann")](https://github.com/ploeh)
[![Ken Bonny](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/KenBonny.png "Ken Bonny")](https://github.com/KenBonny)
[![Simon Cropp](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/SimonCropp.png "Simon Cropp")](https://github.com/SimonCropp)
[![agileworks-eu](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/agileworks-eu.png "agileworks-eu")](https://github.com/agileworks-eu)
[![sorahex](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/sorahex.png "sorahex")](https://github.com/sorahex)
[![Zheyu Shen](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/arsdragonfly.png "Zheyu Shen")](https://github.com/arsdragonfly)
[![Vezel](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/vezel-dev.png "Vezel")](https://github.com/vezel-dev)
[![ChilliCream](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/ChilliCream.png "ChilliCream")](https://github.com/ChilliCream)
[![4OTC](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/4OTC.png "4OTC")](https://github.com/4OTC)
[![Vincent Limo](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/v-limo.png "Vincent Limo")](https://github.com/v-limo)
[![Brooke Hamilton](https://raw.githubusercontent.com/devlooped/sponsors/main/.github/avatars/brooke-hamilton.png "Brooke Hamilton")](https://github.com/brooke-hamilton)


<!-- sponsors.md -->

[![Sponsor this project](https://raw.githubusercontent.com/devlooped/sponsors/main/sponsor.png "Sponsor this project")](https://github.com/sponsors/devlooped)
&nbsp;

[Learn more about GitHub Sponsors](https://github.com/sponsors)

<!-- https://github.com/devlooped/sponsors/raw/main/footer.md -->
