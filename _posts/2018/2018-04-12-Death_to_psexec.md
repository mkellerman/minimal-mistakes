---
layout: single
title: Death to PsExec
excerpt: "PowerShell alternatives to PsExec..."
permalink:
tags: 
- PowerShell
- PsExec
- NUnit
- PSSession
- Remoting
- Invoke-Command
published: true
comments: true
header:
    teaserlogo: 
---
{% include base_path %} 

As a first technical blog post, I've decide to document some of the work I did to move away from <a href="https://docs.microsoft.com/en-us/sysinternals/downloads/psexec">PsExec</a> in our environment. 

#### But why... What's wrong with PsExec?

Currently we use PsExec to remotely execute our NUnit tests against our test environment. It works most of the time, but when there is any network interruption or connectivity issue, the session drops, but the actual process continues on the remote machine. You then have to implement tons of error handling to re-connect and try to continue where you left off, or pick up the test results... it's simply a nightmare.

#### Why not use PowerShell Remote Sessions? 

Have you heard of the Double-Hop issue? Ashley McGlone (@GoateePFE) has a great <a href="https://blogs.technet.microsoft.com/ashleymcglone/2016/08/30/powershell-remoting-kerberos-double-hop-solved-securely/">article</a> explaining the issue and offering tons of solutions. 

#### So why not implement one of those solutions? 

Simple: I wanted to point my script to an IPAddress, and magically have it 'work'. I didn't want to have to setup an AD environment, or configure anything on the remote box.

#### Marc... You're too damn difficult.

I want a simple implementation that replicated the `Invoke-Command` cmdlet, but add a `-As $Credential` to provide the \<PSCredential> I want that \<ScriptBlock> to run on the remote machine. But have the connectivity benefits of the Powershell Remoting Session.

#### What I want it to look like:
```
Invoke-CommandAs -Session <Session> -ScriptBlock <ScriptBlock> -As <PSCredential>
```

First step, establish a Powershell Remote Session to the remote machine, and execute a process with a different set of credential... and return a powershell object? ¯\\_(ツ)_/¯

Should be simple enough...

#### Executing a process with a different set of credentials? Easy peasy lemon squeezy...

<!--

````
# Using Start-Process

Start-Process -FilePath 'powershell.exe' -ArgumentList '-Command Get-Process' -Credential $Credential
````
````
# Using Invoke-WmiMethod

Invoke-WmiMethod -Class Win32_Process -Name Create -ArgumentList 'powershell.exe -Command Get-Process' -Credential $Credential
````
````
# Using System.Diagnostics.Process

[System.Diagnostics.Process]::Start( "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe", "-Command Get-Process", $Credential.UserName.Split('\')[1] , $Credential.Password , $Credential.UserName.Split('\')[0] )
````
-->

I wanted to try out an replicated the `RunAs.exe` command, to start my powershell.exe process under another set of credentials. But I wasn't able to launch it remotely and provide the credentials to the console.

#### Invoke-CmdAs
A colleague of mine found a <a href="https://github.com/JetBrains/runAs">Jetbrains/runAs</a> project that replicates the RunAs.exe utility, but allows the credentials to be passed in the command line. View my wrapper here: <a href="https://github.com/mkellerman/PSRunAs/blob/master/Invoke-CmdAs/Invoke-ExpressionAs.ps1">Invoke-ExpressionAs</a>

But having binaries being copied over is definitely not elegant.

#### Invoke-RunAs
I then found <b>Invoke-RunAs</b> by Ruben Boonen (@FuzzySec) that uses Add-Type to load the DLL and invoke 'Advapi32::CreateProcessWithLogonW' the same way `RunAs.exe` is doing. View my wrapper here: <a href="https://github.com/mkellerman/PSRunAs/blob/master/Invoke-RunAs">Invoke-RunAs</a>

#### Start-ProcessAsUser
Also, verify similarly, <b>Start-ProcessAsUser</b> by Matthew Graeber (@mattifestation) with modifications by Lee Christensen (@tifkin_) uses Reflection to load the DLL and invoke 'Advapi32::CreateProcessWithLogonW' the same way RunAs.exe is doing. View my wrapper here: <a href="https://github.com/mkellerman/PSRunAs/blob/master/Start-ProcessAsUser">Start-ProcessAsUser</a>

All these implementations required some code wizardry to return a PowerShell Objects.

#### Invoke-ScheduledTask
Alternatively, one solution that is very often talked about is to create a Scheduled Task on the remote machine, and let 'SYSTEM' (or any supplied credential) execute your process. It's simple, and after some digging around, i found out it creates a ScheduledJob, and you can Receive-Job the result as a Powershell Object. 

So i created a `Invoke-ScheduleTask` cmdlet to help simplify this and created a wrapper to copy the implementations above. View my wrapper here: <a href="https://github.com/mkellerman/PSRunAs/blob/master/Invoke-ScheduledJob/">Invoke-ScheduledJob</a>

### Simpler is better

After playing around with all these solutions for a while, I've decided to implement the `Invoke-ScheduleJob` into my final solution, as it returns native PowerShell Objects, and doesn't break any of the output streams.

Go check it out here: <a href="https://github.com/mkellerman/Invoke-CommandAs">Invoke-CommandAs</a>.

It's also in the PowerShell Gallery!

````
# Install Module
Install-Module -Name Invoke-CommandAs
````
````
# Execute Locally.
Invoke-CommandAs -ScriptBlock { Get-Process }

# Execute As different Credentials.
Invoke-CommandAs -ScriptBlock { Get-Process } -As $Credential

# Execute Remotely using ComputerName/Credential.
Invoke-CommandAs -ComputerName 'VM01' -Credential $Credential -ScriptBlock { Get-Process }

# Execute Remotely using PSSession.
Invoke-CommandAs -Session $PSSession -ScriptBlock { Get-Process }

# Execute Remotely on multiple Computers at the same time.
Invoke-CommandAs -ComputerName 'VM01', 'VM02' -Credential $Credential -ScriptBlock { Get-Process }

# Execute Remotely as Job.
Invoke-CommandAs -Session $PSSession -ScriptBlock { Get-Process } -AsJob
````

I'm sure there is tons of other ways to do this, or some scenarios that makes one solution better than others, and would love to hear about them!

Msg me on Twitter!
