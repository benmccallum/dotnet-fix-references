# dotnet references

![Nuget](https://img.shields.io/nuget/dt/dotnet-references) <a href="https://www.buymeacoffee.com/benmccallum" target="_blank"><img src="https://bmc-cdn.nyc3.digitaloceanspaces.com/BMC-button-images/custom_images/orange_img.png" alt="Buy Me A Coffee" style="height: auto !important;width: auto !important;" ></a>

A [dotnet global tool](https://docs.microsoft.com/en-us/dotnet/core/tools/global-tools) 
that aids with bulk solution and project file references changes and related directory 
organisation.

(Formerly `dotnet-fix-references`, see prior documentation for it [here](docs/README-dotnet-fix-references.md).)

# Installation

```dotnet tool install --global dotnet-references```

> Note: If using this in a scripted process you want to be consistent (like a build), you should pin to a specific version with `--version x.y.z`.

# Usage

```
Usage: dotnet-references <Mode> [options]

Arguments:
  Mode                                       The mode to run (fix or internalise)

Options:
  -ep|--entry-point                          The entry point to use (a .sln file or a directory)
  -wd|--working-directory                    The working directory to use (under which all .sln and .csproj files are contained). 
                                             If not provided, defaults to directory of entry point.
  -rupf|--remove-unreferenced-project-files  Should unreferenced project files be removed?
                                             (Caution! Deletes files! Only supported in fix mode).
  -reig|--remove-empty-item-groups           Should ItemGroup elements in project files that are empty be removed?
                                             (Only supported in internalise mode).
  -?|-h|--help                               Show help information
```

Supports the following modes which have varying use cases.

## Mode 1: Fix
This mode can fix references between solution files and projects in one of two ways.

### Directory-first 
By passing a directory as the entry, the tool will assume that the current directory structure is the source of truth and will fix all project references inside all .sln and .csproj files to the correct relative path.

> dotnet references fix -ep ./src

Use cases:
 You have moved your source code into a new folder structure (via a script or otherwise) and don't want to manually updates all your project references in .sln and .csproj files. (Project file names must be the same).

### File-first
By passing a .sln file, the tool will assume the it, and all .csproj files in the dependency tree, are the source of truth, creating and moving the .csproj files into the correct directory structure.

> dotnet references fix -ep Company.Project.sln

Use cases:
1. Overcoming Dockerfile COPY globbing limitations... See [here](docs/Dockerfile-use-case.md).

## Mode 2: Internalise (PackageReference --> ProjectReference)
This mode "internalises" references, by turning package references to project references.
(The project name must be the same as the package name).

> dotnet references internalise -wd ./src

Use cases:
1. You've consolidated separate projects (and packages) into a mono repo and want to swap all package references to local project references where possible.

Note: It currently doesn't handle transitive dependencies (dependencies of dependencies), which you'll need to add manually.

## Mode 4: ProjectReference --> PackageReference
Will do if there's interest or I have a need. (Bit more complicated than 3 as you'd be searching package sources rather than local file system.

Use cases:
* You've split out from a mono repo to NuGet packages...

# Word of warning
Every version until v1.0 could contain breaking changes. You should definitely pin to specific versions until then.

This tool updates/deletes files in-place. Backup anything you care about before running this tool. 

# Feature backlog
1. Support fix mode w/ .csproj entry
