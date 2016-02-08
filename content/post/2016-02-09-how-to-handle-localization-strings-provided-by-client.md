+++
Categories = [ "Windows Phone", "WinRT" ]
Description = "I currently work on a Windows Phone 8.1 app for a client with an interesting approach to app localization. They have a Google Docs sheet with all the localization string for the app in all the supported languages and a tool that converts this localization strings in to per-language versioned JSON files. The generated JSON files are kept in a git repository. The Android, iOS and now my Windows Phone app should have the git repository added as a submodule. When a new version of the JSON files with the localization appear in the submodule, the app should use them. Seems reasonable and efficient, so how to approach this on Windows Phone?"
Tags = []
author = "Igor Kulman"
date = "2016-02-09T09:29:12+01:00"
title = "How to handle localization string provided by the client in a Windows Phone app"
url = "/how-to-handle-localization-strings-provided-by-client"

+++

I currently work on a Windows Phone 8.1 app for a client with an interesting approach to app localization. They have a Google Docs sheet with all the localization string for the app in all the supported languages and a tool that converts this localization strings in to per-language versioned JSON files. The generated JSON files are kept in a git repository. The Android, iOS and now my Windows Phone app should have the git repository added as a submodule. When a new version of the JSON files with the localization appear in the submodule, the app should use them. 

**Deciding the localization approach**

In the old days of Windows Phone 7 and Windows Phone 8 I would simply write an utility that reads the newest version of the JSON localization files and generate a RESX file for each language. RESX is XML format that is simple to generate. If you had a localized string with a key say Game, you would put it everywhere when the localization of the word game was needed. Simple 

In Windows Phone 8.1 (and 10) you should use RESW files for localization. This is the new x:Uid approach that, in my honest opinion, really sucks. It forces you to duplicate string if you use I string in multiple places, there is no design time support, you never know what UI element are localized and what you forgot to localize. Simply put, it is a mess. This approach is not usable at all with the string client provides. 

<!--more-->

There is no way to make any script generate a RESW file with duplicated string matching the multiple usages of each string. If you have a localized string with a key Game and 2 TextBlocks and 1 Buttons that use it, you need to put it into the RESW file 3 times, once for each UI element. SO instead of a Game key, you would have a GameTextBlock.Text key, an AnotherGameTextBlock.Text and a GameButton.Content key, all with the same value and applied in design time. Madness. 

**Using RESX files on Windows Phone 8.1**

One approach to solve this problem is to use the good old RESX files with your Windows Phone 8.1 project. Typically by creating a Portable Class Library (PCL) with the RESX files, doing a bit of configuration for the supported languages and linking this PCL project to your Windows Phone 8.1 project. You can find a few articles about this approach online and also a few gotchas and their solutions. It sounds like a good idea at first, but it is too problematic for my tastes. 

I tried this approach but ended frustrated with too many problems too solve, like Visual Studio hanging when deploying to a device when using RESX files. So I decided to go with a custom solution.

**Generating Windows Phone 8-like localization strings**

Finally I decided to generate a class that would look and behave similar to the old Windows Phone 8-like localization strings class. The idea is quite simple, no need to a PCL or any RESX files. 

The first step is to create a directory and empty RESW files for each supported language in the Windows Phone 8.1 project

{{% img-responsive "/images/languages.png" %}}

Then write a script that generates all the RESW files from the newest JSON localization files

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=localization-converter-1.cs"></script>

Now with the RESW files populated with all the localization string it is time to put them to use. First I created a `LocalizedStrings` class very similar to the one in Windows Phone 8:

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=LocalizedStrings.cs"></script>

and added it to `App.xaml` as a dictionary resources called `S`

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=App.xaml"></script>

The interesting ans still missing part is the `AppResources` class referenced in `LocalizedStrings`. It is a partial class, consisting of `ResourceLoader` instance to get the string at runtime and a Singleton to get the string from C# code

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=AppResources.cs"></script>

The second part of this class is generated by the same script that generates the RESW files. For each key in the localization string, it generates a property like this in the `AppResources` class

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=AppResources.strings.cs"></script>

The code to generate this is part of the script

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=localization-converter-2.cs"></script>

**The result**

When you put all this pieces together, you have a RESW files with all the localization strings and a localization class that can be generated on demand. I run the script and generate the files every time there is an update in the git submodule with the JSON localization files.

When I want to use any of these localization string in XAML, I simply use it like this

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=xaml-usage.xaml"></script>

Using a binding allows me to use to use the same localization keys for many elements and I do not have to duplicate the keys like in the x:Uid scenario. 

If I need to used a localization string in C#, I have a strongly typed access to it

<script src="https://gist.github.com/igorkulman/53a29b6e2143cac1ec8a.js?file=csharp-usage.cs"></script>

instead of creating a `ResourceLoader` instance and calling `GetString` with a string parameter. 

Another approach would be having a public property with the `AppResources` instance in a `ViewModel` base class, it would make the binding shorter. 