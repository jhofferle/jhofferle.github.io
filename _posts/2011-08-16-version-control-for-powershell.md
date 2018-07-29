---
id: 666
title: Using Version Control for PowerShell
date: 2011-08-16T19:00:29+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=666
permalink: /version-control-for-powershell/
categories:
  - PowerShell
tags:
  - PowerShell
---
Software developers wouldn&#8217;t think of working on a project without some type of version control for their source code. So why is this is a rare topic for IT Pros that maintain production scripts with hundreds or thousands of lines of code?

Version control, revision control and source control are terms used to describe the practice of tracking and controlling changes to source code. There is a whole slew of proprietary and open source software designed for managing this. Version control software allows a developer to easily see a history of the changes made to code, rollback changes and easily merge changes when several people are working on the same source files.

So how does someone writing some simple scripts benefit from using version control? There are a few signs that may indicate your life could be made easier.

Have you ever frantically searched for an old backup of a script because you can&#8217;t remember what you changed? A common question that&#8217;s asked when something breaks is &#8220;what changed?&#8221; Keeping production scripts in version control makes it very simple to determine what&#8217;s different about the latest version.

Are your scripts filling up with comments that provide a history of changes? Comments are supposed to help make a script easier to understand. I&#8217;ve seen scripts where half the code was commented out in order to maintain a history of changes. Version control is an infinitely better way to keep a record of changes made to a script over time.

![image-left](/assets/img/VersionHistoryInScript.png){: .align-left}

Are your directories full of various versions of a single script? This is something I&#8217;ve been guilty of in the past. It seemed like a perfectly resonable way to maintain a history, but then I realized there was freely available software designed to do all the work for me.

![image-left](/assets/img/ScriptVersions.png){: .align-left}

Have you ever had to manually merge different versions of a script because someone else made changes the same time you did? Merging the changes multiple people make to the same file is one of the primary reasons version control exists. For a single person it&#8217;s helpful, but when there are multiple people it becomes essential.

Using version control software provides an easy way to keep track of every change made to your scripts, and it allows multiple people to collaborate on scripts easily. 

![image-left](/assets/img/VersionControlHistory.png){: .align-left}

There is a ton of information on version control software, with some very strong opinions in favor of a certain package or against another. One thing to keep in mind when reading some of these discussions is that a large software project with hundreds of developers has different needs than a team of two or three network administrators that just want a simple way to keep track of their scripts. Developers can afford to agonize over minor details because the software they select is so central to their work. IT Pros primarily need something that&#8217;s simple to setup and easy to use.

Fortunately we don&#8217;t have a difficult choice, because one of the best and most widely-used programs available is also one of the easiest to use. My software of choice is [TortoiseHg](http://tortoisehg.bitbucket.org/), which is an interface for using the [Mercurial](https://www.mercurial-scm.org/) revision control system on Windows. TortoiseHg includes everything required to get started.

A great introduction to Mercurial can be found at [Hg Init](http://hginit.com/). On Windows, Mercurial can be controlled through the command line, or using the graphical interface provided by TortoiseHg. This tutorial focuses on the command line, but the real value is the way it teaches the concepts behind version control with Mercurial.

Tome Tanasovski wrote about using Mercurial with PowerShell in [Why Every IT Pro Should Use Mercurial for Source Control with PowerShell](http://powertoe.wordpress.com/2010/12/12/why-every-it-pro-should-use-mercurial-for-source-control-with-their-powershell-scripts/).

Art Vandalay has done some work on [a PowerShell provider for local Mercurial repositories](http://ripplingbrainwaves.blogspot.com/2010/03/powershell-provider-for-local-mercurial.html).

The question of [Version Control for PowerShell scripts](http://stackoverflow.com/questions/7070506/version-control-for-powershell-scripts) was recently asked at StackOverflow.