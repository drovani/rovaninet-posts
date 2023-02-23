---
title: Establishing KnockoutJs Design Patterns
category: Rovani in C♯
date: 2014-03-21
---

Now that I have been tinkering with Knockout for about a year, I have finally settled on a host of design patterns that I feel strike a nice balance between rigid consistency and flexible prototyping. I like to have all of my code look very similar. I can look at most code that I have written and know about when I worked on it last. I can usually even look at code that several other developers have touched and see what is mine and what is everyone else's code. Now that I have a team to direct and an expanding code base to maintain, I have realized that I need to document my own code patterns.

[!["It's good to be the king"](/images/good-to-be-king.jpg)](https://youtu.be/lZKiYgcgBAY "It's good to be the king'")

Certainly, my way probably is not the absolute best, but it is what I like and what has worked best for me. Might I change my mind? Absolutely. Probably even before I finish writing this post, I will have changed my mind on several points. However, this is a good place to start.

-----

Two days later, as I started to write up the actual rules I had been following, I decided to look around and see if other people have put together lists of their own. It brought me to this ["Programmers (Stack Exchange)"](http://programmers.stackexchange.com/questions/1323/does-your-company-have-a-coding-standard) question, which had a very nice explanation on why what I was doing was a bad idea.

> It is important for a team to have a single coding standard for each language to avoid several problems:
>
> - A lack of standards can make your code unreadable.
> - Disagreement over standards can cause check-in wars between developers.
> - Seeing different standards in the same class can be extremely irritating.
>
> I'm a big fan of what [Uncle Bob has to say](http://c2.com/cgi/wiki?UncleBobOnCodingStandards) about standards:
>
> 1. Let them evolve during the first few iterations.
> 1. Let them be team specific instead of company specific.
> 1. Don't write them down if you can avoid it. Rather, let the code be the way the standards are captured.
> 1. Don't legislate good design. (e.g. don't tell people not to use goto)
> 1. Make sure everyone knows that the standard is about communication, and nothing else.
> 1. After the first few iterations, get the team together to decide.

So much for that idea.

What have I learned? Well, I have learned that I like this idea of doing two weeks of off-the-cuff posts, and then spending one week putting together a longer post with lots of research and information in it. I also learned that I should do the research for that post at the beginning of the week, instead of at the end of the week. I think I will be coming back to this idea, but it will be less formal that the list of edicts I started putting together. Perhaps a screen-shot of source code with annotations will be the good balance between formality and flexibility.