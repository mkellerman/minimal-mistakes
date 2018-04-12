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

As a first technical blog post, i've decide to document some of the work I did to move away from <a href="https://docs.microsoft.com/en-us/sysinternals/downloads/psexec">PsExec</a> in our environment. 

#### But why... What's wrong with PsExec?

Currently we use PsExec to remotely execute our NUnit tests against our test environment. It works most of the time, but when there is any network interruption or connectivity issue, the session drops, but the actual process continues on the remote machine. You then have to implement tones of error handling to re-connect and try to continue where you left off, or pick up the test results... it's simply a nightmare.

#### Why not use PowerShell Remote Sessions? 

Have you heard of the Double-Hop issue? Ashley McGlone has a great <a href="https://blogs.technet.microsoft.com/ashleymcglone/2016/08/30/powershell-remoting-kerberos-double-hop-solved-securely/">article</a> explaining the issue and offering tones of solutions. 

#### So why not implement one of those solutions? 

Simple: I wanted to point my script to an IPAddress, and magically have it 'work'. I didn't want to have to setup an AD environment, or configure anything on the remote box.

#### Marc... You're too damn difficult.

I want a simple implementation that replicated the Invoke-Command cmdlet, but add a '-As \<Credential>' to provide the \<Credential> I want that \<ScriptBlock> to run on the remote machine. But have the connectivity benefits of the Powershell Remoting Session.

#### What I want it to look like
```
Invoke-CommandAs -Session \<Session> -ScriptBlock \<ScriptBlock> -As \<PSCredential>
```

First step, establish a Powershell Remote Session to the remote machine, and execute a process with a different set of credential... and return a powershell object? ¯\\_(ツ)_/¯

Should be simple enough...

#### Executing a process with a different set of credentials? Easy peasy lemon squeezy...

Test: Execute powershell.exe with a different set of credentials, and return Get-Process.

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


I started by trying to execute the RunAs.exe command on the remote machine. I wasn't able to provide the credential to RunAs.exe, and searched for alternatives.