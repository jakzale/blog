---
title: "Xamarin.Forms on macOS With F#"
date: 2017-09-21T20:42:47+01:00
type: "post"
draft: true
categories: ["Xamarin", "F#"]
---

Here is a post that I have been wanting to write for a long time, namely, how to
bootstrap a Xamarin.Forms application for macOS on F#.

The most recent version on Xamarin.Forms add the support for Xamarin.Forms on
macOS.  There is a brief introduction available on [Xamarin
Blog](https://blog.xamarin.com/preview-bringing-macos-to-xamarin-forms/) for C#,
however doing the same for F# is slightly tricky, so I took the libery of
updating their tutorial and pointing out all shortcomings.

Unfortunately, I do not have the nice screencasts at hand (like the guys from
Xamarin), therefore you will have to bear with me now.

# Setting up a new Xamarin.Forms application
For the purpose of this tutorial I will create a new Xamarin.Forms application.
Now, there is a small ceveat with that, namely if you plan to use a *shared*
project for your code, make sure that it is in the same programming language.

Shared projects simply copy their contents to other projects that reference
them, and since a mix of C# and F# code is not supported in any of the toolchain
I am aware of, you need to make sure that both the shared project and the
project that references the shared project are written either in C# or F#, but
not a mix of them.

## Steps
Below is the list of steps I took

1. Click on *New Project...* in Visual Studio for Mac
2. Multiplatform > App > Blank Forms App > F#
3. Provide the following:
    - App name: FormsExperiments
    - Organization Identifier: pl.medapp
    - Target Platforms: iOS
    - Shared Code: Use Shared Library
    - Use XAML for user interface files: No

Then I check if everything builds and whether I could run it in the Simulator.

**IMPORTANT:** There is a bug in the template (at least in VS for Mac, 7.1.5
(build 2)).  If you specified a different name for the solution and the project,
there will be a bug in your AppDelegate file.  You need to edit
`FinishedLaunching` method, (line 15 in my file) from

```fsharp
    override this.FinishedLaunching (app, options) =
        Forms.Init()
        this.LoadApplication (new SolutionName.App())
        base.FinishedLaunching(app, options)
```

to

```fsharp
    override this.FinishedLaunching (app, options) =
        Forms.Init()
        this.LoadApplication (new ProjectName.App())
        base.FinishedLaunching(app, options)
```

where `SolutionName` is the name of your solution and `ProjectName` is the name
of your project.

With this fix, you should check if you can build and run your application.  In
my case there were still *some* problems reported by the IDE, but I was able to
compile the whole solution and run the application on an iPhone simulator.

# Adding a macOS Project to our Solution
As mentioned in the tutorial from Xamarin's Blog, there is no template for
creating a Xamarin.Forms application for Mac (or at least I have not found any),
therefore we need to create a Cocoa application in F#/Xamarin.Mac first.

## Steps
1. R-click on the solution, then Add > Add New Project
2. Mac > App > Cocoa App > F#
3. As a rule of thumb, set the Project name to: ProjectName.MacOS

Again, as a rule of thumb, try building and running the Cocoa application.  In
my case, there were some problems reported by the IDE, but I was able to compile
the Cocoa application and run it.

### Fixing References
Before we proceed, I decided to fix the problems I was getting with references.  IDE was reporting issues with the following references:
- FSharp.Core
- Xamarin.Mac

I removed the reference for FSharp.Core and instead added a Nuget package for
FSharp.Core 4.1.18.  If I remember correctly, there were some changes in
FSharp.Core 4.2.x, therefore I decided to use the 4.1.x version.

I fixed the issue with Xamarin.Mac reference, by removing it and then re-adding
it again.

After fixing the references I was able to compile everything and run the Cocoa
app.

# Changing the Cocoa app to Xamarin.Forms app
Now, we finally get to changing the Cocoa application to a Xamarin.Forms one,
that will run on macOS.

## Steps
1. Open Info.plist
    - go to Source
    - Delete the NSMainStoryBoardFile entry
2. Delete the Main.storyboard file
3. Install the latest Xamrin.Forms package from Nuget (In my case it was 2.4.0.275-pre3)
4. Add reference to your shared project from your macOS project.
5. Add reference to System.XML from your macOS project.

## Update Main.fs
Now, update the `main` method in `Main.fs` to

```fsharp
module main =
    [<EntryPoint>]
    let main args =
        NSApplication.Init ()
        NSApplication.SharedApplication.Delegate <- new AppDelegate()
        NSApplication.Main (args)
        0
```

## Update AppDelegate.fs
Add the folowing `open` declarations on top of `AppDelegate.fs`

```fsharp
open Xamarin.Forms
open Xamarin.Forms.Platform.MacOS
```

then update your definition of `AppDelegate`:
```fsharp
[<Register ("AppDelegate")>]
type AppDelegate () =
    inherit FormsApplicationDelegate ()
    let style  = NSWindowStyle.Closable
                 ||| NSWindowStyle.Resizable
                 ||| NSWindowStyle.Titled
    let rect   = new CoreGraphics.CGRect(200.0, 1000.0, 1024.0, 768.0)
    let window = new NSWindow(rect, style, NSBackingStore.Buffered, false)
    do
        window.Title <- "Xamarin.Forms on Mac!"
        window.TitleVisibility <- NSWindowTitleVisibility.Hidden

    override val MainWindow = window

    override this.DidFinishLaunching(notification: NSNotification): unit =
        Forms.Init()
        this.LoadApplication(new ProjectName.App())
        base.DidFinishLaunching(notification)
```

where `ProjectName` is the name of your project/Xamarin.Forms app.

Now, you should have a working Xamarin.Forms application on macOS.