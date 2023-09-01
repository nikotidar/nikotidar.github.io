---
layout: post
title: "Change Indent in VSCode"
author: "rndtx"
---

## How to turn 4 spaces indents in all files in VSCode to 2 spaces
  * Open file search
  * Turn on regular expressions
  * Enter `( {2})(?: {2})(\b|(?!=[,'";\.:\*\\\/\{\}\[\]\(\)]))` in the search fields
  * Enter: `$1` in the replace field

## How to turn 2 spaces indents in all files in VSCode to 4 spaces
  * Open file search
  * Turn on regular expressions
  * Enter `( {2})(\b|(?!=[,'";\.:\\*\\\/{\}\[\]\(\)]))` in the search fields
  * Enter: `$1$1` in the replace field

Hope someone see this :)
