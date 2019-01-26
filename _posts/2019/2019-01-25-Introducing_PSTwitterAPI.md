---
layout: single
title: Introducing PSTwitterAPI - a PowerShell module for the Twitter API
excerpt: "the most complete solution available out there!"
permalink:
tags: 
- PowerShell
- PSTwitterAPI
- Twitter
- API
- DataScience
- PSWordCloud
- WordCloud
published: true
comments: true
header:
    teaserlogo: 
---
{% include base_path %} 

## Introducing PSTwitterAPI
`<salespitch>`Ever wanted to get twitter notifications on your build jobs? Or collect some tweets for a report? Or automate some tweets for your blog post? All in PowerShell? This is the module for you! `</salespitch>`

Seriously, for whatever the reasons you need to interact with the Twitter's API, this is the most complete solution available out there!

## Twitter API Helper functions
I originally created the module [InvokeTwitterAPIs](https://github.com/mkellerman/invoketwitterapis) that is sitting in the [PowerShell Gallery](https://www.powershellgallery.com/packages/InvokeTwitterAPIs/2.5), but a lot has changed since then. I wanted to refactor the whole project and make it easier/faster overall. It wasn't complete, and many endpoints where missing.

Instead of writing each endpoints by hand, I created a Script to scrape the Twitter API documentation, get the ResourceUrl, Description and Parameters, to generate all the functions dynamically. You can go check that part out here: [GitHub: APIHelper](https://github.com/mkellerman/PSTwitterAPI/tree/master/APIHelper)

It's not pretty, but it does the job!

## Example 1: Get Twitter user information
```powershell
Import-Module PSTwitterAPI

# Provide Authentication for the Twitter API
# https://twittercommunity.com/t/how-to-get-my-api-key/7033
Set-TwitterOAuthSettings -ApiKey $env:ApiKey -ApiSecret $env:ApiSecret -AccessToken $env:AccessToken -AccessTokenSecret $env:AccessTokenSecret

# Get user twitter profile
Get-TwitterUsers_Lookup -screen_name 'mkellerman'
```
1. Provide authentication token to the module
1. Use one of the +120 helper functions to get/send requires to Twitter
1. Profit

## Example 2: Create WordCloud from your recent tweets

```powershell
Import-Module PSTwitterAPI
Import-Module PSWordCloud

Set-TwitterOAuthSettings -ApiKey $env:ApiKey -ApiSecret $env:ApiSecret -AccessToken $env:AccessToken -AccessTokenSecret $env:AccessTokenSecret

# Request the last 200 tweets from the desired user.
$TwitterStatuses = Get-TwitterStatuses_UserTimeline -screen_name 'mkellerman' -count 200

# Get list of stop words that should be excluded from the world cloud.
$ExcludeWord = (Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Alir3z4/stop-words/master/english.txt' -UseBasicParsing).Content -Split "`n"

# Generate a word cloud from the text in your tweets.
New-WordCloud -InputObject $TwitterStatuses.text -Path ".\TwitterStatuses.png" -ExcludeWord $ExcludeWord -ImageSize 720p

```
1. Provide authentication token to the module
1. Get last 200 tweets from your user
1. Get stop words to be excluded from the world cloud
1. Generate WordCloud
1. Profit

{% include figure image_path="/assets/posts/2020-01-25-Introducing_PSTwitterAPI_WordCloud.png" alt="PSWorldCloud for @mkellerman using PSTwitterAPI" caption="PSWorldCloud for @mkellerman using PSTwitterAPI" %}

## Open Source: Come and help!
I've built the foundation, but there are many things left to do:
- Test each helper function (Manual/Pester).
- If you see a missing API Helper, create an Issue.
- Any improvements you can suggest/make **please** make a PR!

You are using Twitter via Powershell? 
Join me on the PowerShell [Slack](http://powershell.slack.com)/[Discord](https://discord.gg/HjV8R5) #PSTwitterAPI channel! 

And as always, go check it out and let me know what you think!
