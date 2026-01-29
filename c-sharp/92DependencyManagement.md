# Dependency management

## Table of content

- [Dependency management](#dependency-management)
  - [Table of content](#table-of-content)
  - [Central package management](#central-package-management)

## Central package management

Handling packages can already be complicated for one project, how do you do that for multiple projects? That's were Central package management comes in handy! <https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management>.

In order to handle packages globally, you need two files. On indicating which libraries to use, one telling which are the by default versions and then each project has a csproj in order to say which specific packages you need per project.

Examples of files Directory.Build.Props and Directory.Packages.Props:

```props
<Project>
  <PropertyGroup>
    <RunAnalyzersDuringBuild>true</RunAnalyzersDuringBuild>
    <RunAnalyzersDuringLiveAnalysis>true</RunAnalyzersDuringLiveAnalysis>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <TreatWarningsAsErrors>false</TreatWarningsAsErrors>
    <AnalysisLevel>latest</AnalysisLevel>
    <AnalysisMode>All</AnalysisMode>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="SonarAnalyzer.CSharp" PrivateAssets="all" />
    <PackageReference Include="Meziantou.Analyzer" PrivateAssets="all" />
    <PackageReference Include="Roslynator.Analyzers" PrivateAssets="all" />
  </ItemGroup>
</Project>
```

If you need to change globally the version of a package, use the global package.

```props
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="SonarAnalyzer.CSharp" Version="10.18.0.131500" />
    <PackageVersion Include="Meziantou.Analyzer" Version="2.0.286" />
    <PackageVersion Include="Roslynator.Analyzers" Version="4.15.0" />

    <PackageVersion Include="coverlet.collector" Version="6.0.4" />
    <PackageVersion Include="Microsoft.NET.Test.Sdk" Version="17.14.1" />
    <PackageVersion Include="xunit" Version="2.9.3" />
    <PackageVersion Include="xunit.runner.visualstudio" Version="3.1.4" />
  </ItemGroup>
</Project>
```
