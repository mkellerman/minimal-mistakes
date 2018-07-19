---
layout: single
title: Import custom functions into your PSSession/BackgroundJob
excerpt: "use this one-liner to do all the heavy lifting"
permalink:
tags: 
- PowerShell
- PSSession
- BackgroundJob
- TipOfTheDay
published: true
comments: true
header:
    teaserlogo: 
---
{% include base_path %} 

Want to use a custom function inside a PSSession or BackgroundJob, use this one-liner to do all the heavy lifting.

```
Invoke-Expression ${Using:Function:<functionname>}.Ast.Extent.Text
```

Here's a simple example:
```
function Invoke-HelloWorld {
    Write-Warning "Hello World"
}

$ScriptBlock = {
    Invoke-Expression ${Using:Function:Invoke-HelloWorld}.Ast.Extent.Text
    Invoke-HelloWorld
}

Start-Job -ScriptBlock $ScriptBlock | Receive-Job -Wait -AutoRemoveJob
```

How does it work? Let's break it down:
```
${} : Curly braces in PowerShell variable names allow for arbitrary characters in the name of the variable.
$Using:<VariableName> : Using Local variables in remote session.
$Function:<FunctionName> : Provides access to the functions defined in Windows PowerShell.
```
We are basically doing some kind of inception, pulling the function definition from our local session.

But why `${}.Ast.Extent.Text`? my head hurts...

We dont want the scriptblock inside the function. We want the entire function definition as a string.

Hope this helps!

And dont forget to msg me on Twitter (<a href="twitter.com/mkellerman">@mkellerman</a>), love getting your feedback!

References:

* About Remote Variables: <a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_remote_variables?view=powershell-6">Link</a>

* Function Provider: <a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/providers/function-provider?view=powershell-6">Link</a>

