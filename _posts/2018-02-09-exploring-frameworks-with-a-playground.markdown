---
layout: post
title: "Exploring frameworks with a playground"
date: 2018-02-09 18:49:48 GMT
tags: tips xcode swift
---

Playgrounds are a wonderful way to explore new APIs, and to define algorithms. The problem is how to interact with frameworks, either created by ourselves, or imported trough Cocoa Pods or Carthage. 

It turns out that, with Xcode 9, the answer is pretty simple. All we need to do is to create a workspace, which, if we are using CocoaPods, is already there. And if not, it makes sense to start using workspaces when two or more projects are expected to interact either way. 

Then, we just need to create a simple playground, and for the sake of it, save it to the same folder where xcworkspace resides. Once there, let’s open the workspace, and deselect everything in the *Project navigator* (Perhaps, command clicking the selected Project). 

Now, let’s add the files from the playground. 

That’s all, no new target is required. No nothing. Have fun.