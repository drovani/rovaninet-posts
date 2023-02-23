---
title: False Errors Are Better Than Silent Failures
category: Rovani in C♯
date: 2014-04-11
---

In terms of reaction to a request for work, there are several responses. In order from most desirable to least desirable:

1. Successful action with a message of success.
1. Failed action with a message of failure.
1. Successful action without a message.
1. Successful action with a message of failure.
1. Failed action without a message.
1. Failed action with a message of success.


I ran into an issue this week where users were saying that an action they were attempt to perform was reporting success. However, the later steps in their workflow were showing that the original task had failed. This was a disaster for the user experience. I dug into the code and found the offending switch. It was an entirely acceptable failure and a very common occurrence. However, because it was reporting success, users were obviously confused and overall productivity quickly ground to a halt. When my success is determined by the productivity gains that my software provides to others, this is clearly a very serious concern of mine.

From the time the issue was first report to me to the time it was fixed was approximately four minutes. No one in the organization really gave it a second thought, and merrily they went about their day. I, however, was thoroughly distraught. How did this pass through automated tests, manual testing, and user testing, and find its way into the production environment?

Today, I finally rearranged the entire testing suite, streamlining both the ability to add a new test and to determine code coverage. In reporting back to my boss about what I did this week ("re-factored the testing procedures"), he was thoroughly confused why I spent more than twenty hours on something that was fixed in four minutes. Even the explanation of "so it won't happen again" only slightly satisfied his question. However, as soon as I took the deep breath to fully explain why I did what I did and how it will help, he just cut me off say "it won't happen again? it's fine?" and he moved onto other things. It is really comforting knowing my boss has this level of trust. That he knows that whatever I have decided to spend my time on is almost assuredly results in the best service to our organization.

Or maybe he has no trust in me, but I am reading it as a message of success.