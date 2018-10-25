---
title: "Implementing an IL Viewer for Ionide"
date: 2018-03-10T11:13:31+01:00
type: "post"
draft: true
categories: []
description: ""
---

Back in October 2017 I was thinking about implementing an IL Code Viewer for
Ionide, however I did not have enough time to work on it until recently.

# Viewing IL Code
In order to view IL code I have to find a suitable disassembler.  Since I am
working on macOS, I either use Mono or .NET Core, so I had to get a little bit
creative with finding a disassembler.

# Creating a Sample Assembly to DisAssemble
I have prepared a set of sample programs to disassemble available at [jakzale/DisassemblerSamples](https://github.com/jakzale/DisassemblerSamples)

I used the F# templates for `classlib` and `console` from the `dotnet new`
command.

I also updated all the samples to target `net461` as well.
``` xml
<PropertyGroup>
  <TargetFrameworks>netstandard2.0;net461</TargetFrameworks>
</PropertyGroup>
```

# Disassembling
Let's build and disassemble the SimpleLibrary assembly.

``` sh
msbuild /t:Restore
msbuild /t:Build
monodis src/SampleLibrary/bin/Debug/net461/SampleLibrary.dll
```

## Result
```
.assembly extern mscorlib
{
  .ver 4:0:0:0
  .publickeytoken = (B7 7A 5C 56 19 34 E0 89 ) // .z\V.4..
}
.assembly extern FSharp.Core
{
  .ver 4:4:1:0
  .publickeytoken = (B0 3F 5F 7F 11 D5 0A 3A ) // .?_....:
}
.assembly 'SampleLibrary'
{
  .custom instance void class [FSharp.Core]Microsoft.FSharp.Core.FSharpInterfaceDataVersionAttribute::'.ctor'(int32, int32, int32) =  (
		01 00 02 00 00 00 00 00 00 00 00 00 00 00 00 00 ) // ................

  .custom instance void class [mscorlib]System.Runtime.Versioning.TargetFrameworkAttribute::'.ctor'(string) =  (
		01 00 1C 2E 4E 45 54 46 72 61 6D 65 77 6F 72 6B   // ....NETFramework
		2C 56 65 72 73 69 6F 6E 3D 76 34 2E 36 2E 31 01   // ,Version=v4.6.1.
		00 54 0E 14 46 72 61 6D 65 77 6F 72 6B 44 69 73   // .T..FrameworkDis
		70 6C 61 79 4E 61 6D 65 14 2E 4E 45 54 20 46 72   // playName..NET Fr
		61 6D 65 77 6F 72 6B 20 34 2E 36 2E 31          ) // amework 4.6.1

  .custom instance void class [mscorlib]System.Diagnostics.DebuggableAttribute::'.ctor'(valuetype [mscorlib]System.Diagnostics.DebuggableAttribute/DebuggingModes) =  (01 00 03 01 00 00 00 00 ) // ........

  .hash algorithm 0x00008004
  .ver  0:0:0:0
}
.mresource public 'FSharpSignatureData.SampleLibrary'
{
}
.mresource public 'FSharpOptimizationData.SampleLibrary'
{
}
.module SampleLibrary.dll // GUID = {5AA3B641-1FCB-DE77-A745-038341B6A35A}


.namespace '<StartupCode$SampleLibrary>.$.NETFramework,Version=v4.6.1'
{
  .class private auto ansi abstract sealed AssemblyAttributes
  	extends [mscorlib]System.Object
  {

  } // end of class <StartupCode$SampleLibrary>.$.NETFramework,Version=v4.6.1.AssemblyAttributes
}

.namespace SampleLibrary
{
  .class public auto ansi abstract sealed Say
  	extends [mscorlib]System.Object
  {
    .custom instance void class [FSharp.Core]Microsoft.FSharp.Core.CompilationMappingAttribute::'.ctor'(valuetype [FSharp.Core]Microsoft.FSharp.Core.SourceConstructFlags) =  (01 00 07 00 00 00 00 00 ) // ........


    // method line 1
    .method public static 
           default void hello (string name)  cil managed 
    {
        // Method begins at RVA 0x2050
	// Code size 28 (0x1c)
	.maxstack 4
	.locals init (
		class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<string, class [FSharp.Core]Microsoft.FSharp.Core.Unit>	V_0,
		string	V_1)
	IL_0000:  ldstr "Hello %s"
	IL_0005:  newobj instance void class [FSharp.Core]Microsoft.FSharp.Core.PrintfFormat`5<class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<string, class [FSharp.Core]Microsoft.FSharp.Core.Unit>, class [mscorlib]System.IO.TextWriter, class [FSharp.Core]Microsoft.FSharp.Core.Unit, class [FSharp.Core]Microsoft.FSharp.Core.Unit, string>::'.ctor'(string)
	IL_000a:  call !!0 class [FSharp.Core]Microsoft.FSharp.Core.ExtraTopLevelOperators::PrintFormatLine<class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<string,class [FSharp.Core]Microsoft.FSharp.Core.Unit>> (class [FSharp.Core]Microsoft.FSharp.Core.PrintfFormat`4<!!0,class [mscorlib]System.IO.TextWriter,class [FSharp.Core]Microsoft.FSharp.Core.Unit,class [FSharp.Core]Microsoft.FSharp.Core.Unit>)
	IL_000f:  stloc.0 
	IL_0010:  nop 
	IL_0011:  ldarg.0 
	IL_0012:  stloc.1 
	IL_0013:  ldloc.0 
	IL_0014:  ldloc.1 
	IL_0015:  callvirt instance !1 class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<string, class [FSharp.Core]Microsoft.FSharp.Core.Unit>::Invoke(!0)
	IL_001a:  pop 
	IL_001b:  ret 
    } // end of method Say::hello

  } // end of class SampleLibrary.Say
}

.namespace '<StartupCode$SampleLibrary>'
{
  .class private auto ansi abstract sealed $Library
  	extends [mscorlib]System.Object
  {

  } // end of class <StartupCode$SampleLibrary>.$Library
}

```

Yay, works!  It is a bit wordy, but it does indeed work.

# Checking out Ionide
Ionide provides some nice guidelines on contributing ([CONTRIBUTING.md](https://github.com/ionide/ionide-vscode-fsharp/blob/master/CONTRIBUTING.md)).

