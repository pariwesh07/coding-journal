---
title: Fast way to check if a file exists in WinRT
author: Igor Kulman
layout: post
date: 2012-10-26
url: /fast-way-to-check-if-a-file-exists-in-winrt/
onswipe_thumb:
  - SKIP
dsq_thread_id:
  - 1405278277
categories:
  - WinRT
tags:
  - 'c#'
  - metro
  - winrt
---
The StorageFolder class does not containt any method to determine if a file with given name exists in that folder. There are at least two ways how to implement such method, let us call it ContainsFileAsync.

The first method could take advantage of the GetFilesAsync method that returns a readonly list of all the files in a folder as StorageFile objects. The StorageFile contains a property called Name that you could compare with the given filename. The whole code would look like this

{{< gist 5849495>}}

This method looks fairly simple, elegant and works if the folder contains only a few files. When the folder contains tens or hundreds of files, this method gets very slow. In a folder with approximately 100 files the method takes about 1.5 seconds to return the result.

There is faster way, that works in constant time (about 50ms) even in folders with hundred files, but it is not so elegant. The idea is simple, try to access the files and if you get an exception, just return false.

{{< gist 5849500>}}

Although I am a big fan of elegance and readability when coding, I prefer and use the second method.