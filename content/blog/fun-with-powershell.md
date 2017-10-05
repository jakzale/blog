---
title: "Fun With PowerShell"
date: 2017-10-05T09:32:28+01:00
type: "post"
draft: true
categories: ["PowerShell", "macOS"]
description: "I recently got my hands on PowerShell for Mac."
---

# PowerShell Profile

The path to the current powershell profile is stored in the $profile variable.
```powershell
PS> $profile
/Users/jakub/.config/powershell/Microsoft.PowerShell_profile.ps1
```

You can check if the file exists using the `Test-Path` cmdlet:
```powershell
PS> Test-Path $profile
False
```

By default, the profile does not exist.  You can create it with the `New-Item`
cmdlet
```powershell
PS> New-Item $profile
```

And then you can open it in your editor of choice.  I use VSCode, which provides
the `code` command

```powershell
PS> code $profile
```

which opens the current powershell profile in VSCode.

# Adding Colors to ls
Before I proceed I wanted to add an alias for listing files in directory with
colors.  While on Bash/Zsh I could easily do

```sh
alias ls="ls -G"
```

doing it on powershell was slightly more involved, as powershell does not
permits for creating aliases to cmdlets with arguments.  Therefore I had to create a function that prepends the `-G` to the list of arguments:

```powershell
function Get-ChildItemColor {
    $args = ,"-G" + $args
    &/bin/ls $args
}
```

Now there are some things to unpack

```powershell
$args = ,"-G" + $args
```
concatenates an array containing `"-G"` and `$args`.

```powershell
&/bin/ls $args
```
uses the call operator `&`, which interprets its first argument as a command,
where we provide the full path to /bin/ls, otherwise we would lately get into
infinite loop when we would bound `Get-ChildItemColor` to `ls`.

The second argument of `&` is the array of arguments to apply.