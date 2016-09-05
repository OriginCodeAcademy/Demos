# Clean Architecture

This demo is to show students the importance of good architecture in C# to allow for better unit testing and overall maintainability of a code base.

## High Level
* [ ] <High Level Steps>

## Steps

* [ ] Open the "unclean" Todo Project
* [ ] Add a new `Todo.Core` project to the `Todo` solution
	* [ ] Right click `Todo` solution
	* [ ] Choose `New > New Project`
	* [ ] Select `Visual C# > Windows > Class Library`
	* [ ] Name the new project `Todo.Core`
	* [ ] Click OK
	* [ ] Remove `Class1.cs`
	* [ ] Add `Domain` folder to `Todo.Core`
		* [ ] Move `Todo.API.Models.Todo.cs` to `Todo.Core.Domain` folder.
		* [ ] Rename namespace in `Todo.cs` to `Todo.Core.Domain`
		* [ ] Delete `Todo.API.Models.Todo.cs`.
* [ ] Add a new `Todo.Data` project to the `Todo` solution
	* [ ] Right click `Todo` solution
	* [ ] Choose `New > New Project`
	* [ ] Select `Visual C# > Windows > Class Library`
	* [ ] Name the new project `Todo.Core`
	* [ ] Click OK
	* [ ] Remove `Class1.cs`
	* [ ] Install `EntityFramework` via Nuget into `Todo.Data`
	* [ ] Add `Infrastructure` folder to `Todo.Data`
		* [ ] 


### Add a Todo.Core project
* [ ] Right click solution
* [ ] Choose new project
* [ ] Select `Class Library`
* [ ] Press OK

### Add files to Todo.Core