# Dependency management

## Table of content

- [Dependency management](#dependency-management)
  - [Table of content](#table-of-content)
  - [Central package management](#central-package-management)

## Central package management

Handling packages can already be complicated for one project, how do you do that for multiple projects? That's were Central package management comes in handy! <https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management>.

You have a central .props file to handle package version for all our projects. Each project has it's own package reference file but they all choose the version indicated on the .props file. You can even define multiple package versions for different projects.
