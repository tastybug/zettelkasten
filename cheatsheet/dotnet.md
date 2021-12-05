# Dotnet

List and alter dependencies of a project 
* `dotnet list [package|reference]`
* `dotnet add package newtonsoft.json`
* `dotnet remove package System.Net.Http` 

Add references to other projects on disk (e.g. a lib to an app)
* `dotnet add ./MonitoringApp.csproj reference ../MonitoringLib/MonitoringLib.csproj`

Download deps of a project according to its project file
* `dotnet restore` 

Build lifecycle
* `dotnet clean`
* `dotnet build [-c Release]`
* `dotnet run`
* `dotnet test`

Create a NuGet package
* `dotnet pack [-c Release]`

Create an assembly (either self contained or with deps)
* `dotnet publish -c Release -r win10-x64`

