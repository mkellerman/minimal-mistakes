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

Ever wanted to Start-Job with some Throttle? Or with Timeout? How about loading your session functions inside your Background Job?

I use Start-Job daily. Managing +300 VMs for our QA environment in VCenter (VMWare) as SCVMM (Hyper-V). It's a breeze when you know how to run things in parallel. Many scripts/modules are out there to help you do it quickly and efficiently. Most of them, are built around Runspaces and for good reasons. Using Runspaces are blazing fast! But in some cases using Runspaces isn't the right option. For those times, i've built Invoke-Job.

I wanted to replicate the Start-Job cmdlet, but add a few extra tweaks: 
  * ImportFunctions
  * Throttle
  * Timeout
  * PassThru

Let's make a simple example:

# Load script in this user session
# https://gallery.technet.microsoft.com/scriptcenter/Get-PendingReboot-Query-bdb79542
Import-Module .\Get-PendingReboot.ps1
Import-Module VMWare.PowerCLI

$ScriptBlock = {
    Get-PendingReboot -ComputerName $Input
}




