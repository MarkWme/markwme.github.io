---
layout: post
title:  "Microsoft White Paper: Modern Application Modeling and Configuration for Infrastructure Clouds"
categories: cloud
tags: cloud azure arm "azure resource manager" devops
---
Microsoft has announced the publication of a White Paper titled ["Modern Application Modeling and Configuration for Infrastructure Clouds"][white-paper].  The document does a good job of describing the current state of typical on-premise IT infrastructure in medium to large organisations that have invested heavily in Microsoft technologies such as System Center and gives some clues as to the future direction for these tools and on which technologies to focus.

The paper helpfully covers off a number of current Microsoft technologies that appear to overlap with each other and helps with understanding which horse you should back.  For example, the question of PowerShell DSC vs System Center Configuration Manager's Compliance Settings is one that I have been interested in recently.  My suspicion, given the prominent positioning of all things PowerShell these days, was that PowerShell DSC would likely end up integrating with / superceding Config Manager's features and this paper appears to confirm that.  Another area is that of Service Management Automation (SMA) vs. System Center Orchestrator and again, it seems as though the new kid on the block is going to win out in the end.

The paper certainly isn't suggesting an imminent demise for System Center's technologies, but it does appear to be strongly advocating a switch to their more recent cousins.  None of this should come as a surprise to anyone who's been following Microsoft's server and cloud technologies closely over the past two or three years.

The biggest issue is that for many of these organisations that are using System Center extensively, a lot of time and money has gone into building that infrastructure and starting a shift away to new technologies is likely to be a slow process.  With the best will in the world, everyone wants to be using the latest and greatest technology, but larger companies wont want to change something that's apparently working very well right now.  The way I approach this is to plan for all the items that need to change to reach the end state, break them down into their various components and dependencies and then start to change small things and piece by piece, you'll get to where you want to be.

[white-paper]:	https://gallery.technet.microsoft.com/Modern-Application-5930f388