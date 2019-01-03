---
layout: single
title: Copy-xItem to Remote Session
excerpt: "Need to copy files, functions and modules to a remote session using older versions of Powershell?"
permalink:
tags: 
- PowerShell
- Copy
- Module
- PSSession
published: true
comments: true
header:
    teaserlogo: 
---
{% include base_path %} 

### Copying a file to a Powershell Remote Session

Most of the time I write my script locally. I then add a second layer to run it remotely, and make sure everything works well. Building my scripts like a cake.

I've build a few tools to help me copy files, functions and modules over to a remote machine. I didn't want to go the UNC share to copy files over the conventional way.. because <insert valid, but pointless reason here>. 

In PS 5.1, Copy-Item got a new property: -ToSession. I thought that was brilliant! I wish more Cmdlets had this property. I then decided to implement my own version of this, so i can use it in older environments. 

#### Copy-xItem

````
Import-Module Copy-xItem
$Session = New-PSSession -ComputerName $ComputerName -Credential $Credential
Copy-xItem -Session $Session -Path 'C:\Workspace' -Destination 'C:\Workspace' -Recurse -Force

Invoke-Command -Session $Session -ScriptBlock { Get-ChildItem 'C:\Workspace' }
````

#### Copy-xFunction

````
Import-Module Copy-xItem
Import-Module .\Get-PendingReboot.ps1

$Session = New-PSSession -ComputerName $ComputerName -Credential $Credential
Copy-xFunction -Session $Session -Name 'Get-PendingReboot'

Invoke-Command -Session $Session -ScriptBlock { Get-PendingReboot }
````

#### Copy-xModule

````
Import-Module Copy-xItem
Import-Module .\Get-PendingReboot.ps1

$Session = New-PSSession -ComputerName $ComputerName -Credential $Credential
Copy-xModule -Session $Session -Name 'Get-PendingReboot'

Invoke-Command -Session $Session -ScriptBlock { Get-PendingReboot }
````

### Conclusion

So there you have it.. little duck tape, a couple paperclips, and presto, my own little script to copy over files, functions and modules to a remote PowerShell Session. I'll dump the source of Copy-xItem, Copy-xFunction and Copy-xModule, on GitHub.

https://github.com/mkellerman/Copy-xItem

Let me know what you think! And dont be shy to make PRs!
