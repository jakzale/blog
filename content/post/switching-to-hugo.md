---
title: "Switching to Hugo"
date: 2017-09-21T15:40:40+01:00
draft: true
---

After repeatedly failing over and over again to start my own blog, I have
decided to try with [hugo](https://gohugo.io) and see how it works out.  

# Theme
Argh!  Decisions...  After some back and forth with all the themes available for
Hugo, I decided to go with the [Cactus](https://themes.gohugo.io/cactus/) theme,
which is a nice and minimalistic theme.

## Syntax Highlighting
Unfortunately, after trying to create my first blog post I have hit a wall --- namely the syntax highlighting for F# did not work.  Upon closer inspection I needed to update the [Highlight.js](https://highlightjs.org) to a more recent version.  Initialy I thought that I would have to fork the theme to do so, but then I realised that I could simply shadow the troublesome file from the theme by supplying an updated `highlight.js` in the `/static/js` folder.  Yay!

Here is an example of working syntax highlighting for F#:

```fsharp
(**
Now, let's see if some more complex F# code will work
*)

namespace Random.Namespace

[<RequireQualifiedAccess>]
module Hugo =
    type Greater () =
        let greet () =
            printfn "Hello, world from Hugo!"
        member __.Greet () =
            greet ()
```

# Summary
There are still some bits that are not finished and somewhat annoy me.  For
instance, there is a spurious *share* on Facebook button that I will probably
remove, but for now I am pretty happy with the outcome.

Also, for some unknown reason, the main title of the page is `Tile <dot> Title`, which is somewhat weird.