---
title: Just Because It's On the Microsoft Stack Doesn't Mean It's Closed Source
category: Rovani in C♯
date: 2014-03-06
---

I work almost exclusively on the Microsoft WISC stack. The first response that I hear from almost _every single developer_ that I've chatted with is something along the lines of "oh, that's all closed source software". It makes me want to throttle them. Mostly because it is an entirely antiquated line of thinking that just shows a lack of actual knowledge of the ecosystem.

Here is a list of the different libraries used in my current project:

- [ASP.NET MVC](http://aspnetwebstack.codeplex.com/)
- [DotNetOpenAuth](https://github.com/DotNetOpenAuth/DotNetOpenAuth)
- [MiniProfiler](https://github.com/SamSaffron/MiniProfiler)
- [MvcContrib](http://mvccontrib.codeplex.com/SourceControl/latest)
- [Newtonsoft.Json](https://github.com/JamesNK/Newtonsoft.Json)
- [Unit Application Block](http://unity.codeplex.com/SourceControl/latest)
- [WebActivator](http://unity.codeplex.com/SourceControl/latest)
- [CommonServiceLocator](http://commonservicelocator.codeplex.com/SourceControl/latest)

The list of JavaScript libraries is long, but the major packages that are used are:

- Knockoutjs
- jQuery
- jQueryUI
- Moment.js
- Globalize.js

The tools that I use are a mix of paid closed source, freemium closed source, and FOSS:

- Microsoft Visual Studio 2013 (paid closed source) - though the 14 different extensions that I use with VS are all free and open source.
- LinqPad (freemium closed source) - well worth every penny
- Visual Studio Online (freemium closed source service) - also known as Team Foundation Service

The core Microsoft part of the stack is paid, closed source - sure. They have to make their money somehow. However, just because the base part of the stack is closed source does not mean that every developer is producing closed source code. There is lots of open source software written for the WISC stack, and plenty more will be written. Both by Microsoft, and by other developers.

Elitism at its finest&hellip; and people wonder why sometimes I hate socializing with developers.