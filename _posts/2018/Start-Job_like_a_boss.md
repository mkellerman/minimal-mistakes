---
layout: single
title: Start-Job like a boss
excerpt: "Ever wanted to Start-Job with Throttle?"
permalink:
tags: 
- PowerShell
- Start-Job
- Throttle
- Timeout
- Parallel
published: true
comments: true
header:
    teaserlogo: 
---
{% include base_path %} 

Ever wanted to Start-Job with some Throttle? Or with a Timeout? And how about loading your session functions inside your Background Job?

I use Start-Job daily. Managing +300 VMs for our QA environment in VCenter (VMWare) as SCVMM (Hyper-V). It's a breeze when you know how to run things in parallel. Many scripts/modules are out there to help you do it quickly and efficiently. Most of them, are built around Runspaces and for good reasons. Using Runspaces are blazing fast! But in some cases using Runspaces isn't the right option. For those times, i've built Invoke-Job.

I wanted to replicate the Start-Job cmdlet, but add a few extra tweaks: 
  * ImportFunctions
  * Throttle
  * Timeout
  * PassThru

Let's make a simple example:
```
Import-Module Invoke-Job

# Load a script in this user session
# https://gallery.technet.microsoft.com/scriptcenter/Get-PendingReboot-Query-bdb79542
Import-Module .\Get-PendingReboot.ps1

# Make a simple ScriptBlock that will receive the computername from the pipeline, and execute the script against the remote computer. In this case, Get-PendingReboot that we've loaded in this current session.

$ScriptBlock = {
  Process {
    
    # Collect the InputObject sent through the Pipeline.
    $ComputerName = $_
    
    Try   { 
    
      # Because Get-PendingReboot has been imported into this session, I can call the function directly.
      # I'm also $Using variables from the parent session. Everything works as expected.
      
      Invoke-Command -ComputerName $ComputerName -Credential $Using:Credential -ScriptBlock ${function:Get-PendingReboot} 
    
    } Catch { 
    
      Write-Error $_.Exception.Message 
    
    }
  
  }
}

# Get list of computers and set the credential used for the remote execution.

$Computers = Get-Content -Path '.\computers.txt'
$Credential = Get-Credential

# Start-Job 4 jobs at a time, with a timeout of 30 seconds, and return the results (not the Job object). And beautify it by presenting the results in a table.

$Computers | Invoke-Job -ScriptBlock $ScriptBlock -Throttle 4 -Timeout 30 -ImportFunctions -PassThru | Format-Table
```
And here you go. Simple, and easy to read code. If i had to put the code here to handle the Throttle, the Timeout and the PassThru, it would have been a +100 lines of code and hard for someone else to read and maintain. 

Let me know what you guys think!
Msg me on Twitter!
