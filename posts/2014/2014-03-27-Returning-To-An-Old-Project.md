---
title: Returning To an Old Project
category: Rovani in C♯
date: 2014-03-27
---

> This old code? Sure, I think I can make a quick tweak to it. Let me take a look.

Thus, the yak shaving begins. The code compiles just fine, the tests all pass, and I can just edit a couple of lines and be done with this. Edit, build, deploy, drink coffee.

It was nice of NuGet, though, to notify me that one of the libraries that I am using has an updated version available. Well, I think I will just click that handy Update Packages button.

Oh, there is a conflict between a file that already exists and the new version coming in - go ahead and overwrite the file. I am pretty sure it is something minor that I will easily discover; and if it does not work, then that is what source control is for.

> Whoa! WHOA! Slow down there, cowboy. You told me there was only one package to update, not the fourteen you just updated. Now my project no longer builds. Wait&hellip; you REMOVED A PACKAGE? Now we have a problem.

At this point, I went back to TFS to figure out what package had been removed. Then I had to go research what that package used to do, find its replacement, discover how to interact with it, find that there is a better way to utilize it, rebuild the whole solution, fix a plethora of bugs this caused, and eventually get that small portion working.

One package update done; thirteen more to go. This was supposed to be a quick tweak.

Two days later, I decide that updating the packages is not worth it at this time. I document that the application needs package updates (all fourteen at this time), approximately how long I think it will take (a week), potential impact if it does not get updated (none), and what selling points for the user there are to updating (none).

Now, on to the next minor tweak that tries to convince me that library updates are smooth and painless. Also, I need more coffee.