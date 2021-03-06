# Referencing libraries and scripts

C# scripts can reference other scripts and dlls. Here's how...

## Referencing libraries
A default set of *references* and *usings* is included automatically but you can add your own as well. For example, the `System.IO.Compression` assembly isn't referenced by default. Here's how you can include it to use the `ZipFile` class:

``` C#
#r "System.IO.Compression" //add reference (the dll is in the GAC)
using System.IO.Compression; //include namespace 

string startPath = @"c:\example\start";
string zipPath = @"c:\example\result.zip";

ZipFile.CreateFromDirectory(startPath, zipPath);
```  

The `#r` directive adds a reference to the specified assembly. 

Here are some examples of how to use it:

- `#r "System.IO.Compression"` - reference an assembly from the GAC
- `#r "MyLib.dll"` - reference an assembly from the same folder as the current workbook
- `#r "C:\packages\lib.dll"` - reference a dll by absolute path
- `#r "\\serverA\packages\lib.dll"` - reference a dll on a network share

!!! Note
	- Support for NuGet is planned but not yet available.
	- You can customize included references and usings in custom connections. 

### Script references
When defining general purpose functions and classes, it can be useful to put them in their own script files. We can reference them using the `#load` directive. 

Here are some examples:
 
- `#load "c:\mycode\class123.csx"` load a script form a local file
- `#load "\\server\scripts\demo.csx"` load a script form a network share
- `#load "my script"` load an embedded script from the current workbook
- `#load "my folder\my script"` load an embedded script inside a folder in the current workbook
 
The `#load` directive accepts absolute paths and network shares, but it also accepts embedded scripts, as shown below:

![Referencing embedded script](https://i.imgur.com/RHSRZpg.png)

In the example above, the class `MyCustomClass` is defined in an embedded script called *my script*. When we load the script we can access the `MyCustomClass` type contained inside.