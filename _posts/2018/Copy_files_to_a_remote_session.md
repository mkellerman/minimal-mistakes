---
layout: single
title: Copy-xItem to Remote Session
excerpt: "Need to copy files to a remote session using older versions of Powershell?"
permalink:
tags: 
- PowerShell
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

In PS 5.1, Copy-Item got a new property: -ToSession. I thought that was brilliant! I wish more cmdlets had this property. I then decided to implement my own version of this, so i can use it in older environments. 

Here's my implementation:

    function Copy-xItem {

        Param(
            [Session]$Session, 
            [String]$FilePath, 
            [String]$DestinationPath
        ) 

        Begin { }

        Process {

            # Read/Convert FilePath to base64string
            $base64string = [Convert]::ToBase64String([IO.File]::ReadAllBytes($FilePath))
            
            Invoke-Command -Session $Session -ScriptBlock {

                # Force create the path so it avoids errors if folder already exists
                New-Item -Path $Using:DestinationPath -Force | Out-Null

                # Convert/Write base64string to DestinationPath
                [IO.File]::WriteAllBytes($Using:DestinationPath, [Convert]::FromBase64String($Using:base64string))

            }

        }

        End { }
    }

As you can see, it's pretty straight forward. Read/Convert the file to a base64 string, and Convert/Write it to the DestinationPath in the remote session.

** If anyone know of any limitations from this implementation leveraging '$Using' (ex: MaxLength?), let me know. And yeah, i know it puts the entire file into memory.

So there you have it.. little duck tape, a couple paperclips, and presto, my own little script to copy files over to a remote powershell session. I'll dump in my github repo my Copy-xItem, Copy-xFunction and Copy-xModule, that uses all the same type of functionality.

Let me know what you think! And dont be shy to make PRs!