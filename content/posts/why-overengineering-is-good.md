---
title: "Why you should overengineer your code (sometimes)"
date: 2022-01-19T18:35:59+01:00
tags: [development,shower-thoughts]
---

Let's firstly clarify what I mean by overengineering.

I'm not saying you should do things like use [is-even](https://www.npmjs.com/package/is-even) or solve *trivial* math problems with more loops than you can count with your fingers...
I am saying that you should only do it if you can't think of something better on the spot.

This might sound strange at first, but hear me out, ok?

To prove my point, let's use some diagrams to show the developer mental state during the development and maintenance phases.

**Baseline**
![Baseline](/why-overengineering-is-good/base.png "Baseline")

The above diagram shows the baseline we'll be using.
Above the middle line is the positive mental state -- everything is working good, everyone is happy.
Below the middle line is the negative mental state -- stuff is breaking, and clients are on our tail.

* Right at the top, we have the good'ol **god complex** which makes us an absolute coding legend.

* In the middle, spanning slightly into the negative, we have the state you'll generally experience, the **normal state**.
  Things will be good, but stuff can go south.

* After that, we have a whole section dedicated to the initial phase of debugging... checking the git blame to see who caused the bug.
  If **someone else** is the author, let everyone know that your but is safe.
  If **you're** the commit author, it was probably because you were refactoring the code or use some other excuse you see fit.

* Right after that, spanning into the void.
  We realise that what we've done should not have been done.
  We have a phase where we secretly commit patches to patch away the nasty stuff and hope no one notices.

Now then, diagrams for the two extreme cases -- the noob developer and the professional noob developer.

**Noob developer**
![Noob developer](/why-overengineering-is-good/noob.png "Noob developer")

You thought of a banging new approach to solve the problem.
You do some research find validation that the solution is the best.
You start implementing.
It works... kind of...
There are some limitations the solution doesn't quite address...
You find some janky copy-paste solution that fixes it.
Phew, job well done, stuff works and the managers are happy.

Then comes the maintenance phase, where you'll need to maintain your code because no one in your team knew it was possible to write such a... *masterpiece*.
The problem is, not even you fully realise what you've done.
You think it will be easy, but you soon realise that you are indeed mistaken.

**Pro developer**
![Pro developer](/why-overengineering-is-good/pro.png "Pro developer")

Ah yes... modern problems require modern solutions.
As more experienced you become, the more knowledgeable you are.
You apply a good solution with theory behind it and... you find a library that does something close enough (use the "limited by the technology of my time" for things that are not supported).

The maintenance couldn't be any simpler.
You use the appropriate testing theories to pinpoint the exact issue, what we could do and... open up an issue on the library's github, asking for frequent updates until they fix it.

# So then... why should you overengineer?

Well... since you're starting out, your code is probably kinda bad -- loop-galore, buggy code, and janky solutions, to name a few.

When doing something foreign (learning to code, writing some new strange feature) it is usually more rewarding to just have it working -- we can worry about making it look gooder while making it work less later.

So with that logic, you should go all out and try to cram as much nonsense as humanly possible while keeping your code doing what it is supposed to do.

# So... when should you do it?

I think you should consider doing these things when starting off with programming or when you're implementing something you've never done before (or you just want to test your colleagues a bit)

# Possible implications

> "I'll get chewed up by the code review, and my mentor will be upset with me..."

A code review is not considered valid by defacto standards unless the reviewer finds as many issues as humanly possible (petty issues, such as trailing whitespace, give extra points).

Have a little vengeance and make them read through the entire code to properly understand it and suggest appropriate modifications.

Who knows, it might even turn out that the overengineered solution was just enough overengineered.
You might even make the reviewer have to do his own research to determine why your solution is not good.

> "The codebase will be corrupt, the system will be unstable..."

And well thought out, well-tested code can't do the same?
If your solution works and has appropriate testing behind it... consider it an easter egg and call it a day.

> "The feature will be hard to extend with additional stuff we might not have thought of..."

Who cares! You'll probably need to do a lot of refactoring to make the feature fit in anyway.

You might even do some refactoring to make the feature better since you've learned new things from when you've written the initial version.

# And how should you do it?

While I think this is valid and can be quite valuable from the learning perspective (for both you and your colleagues), you shouldn't make it seem like you're trying to *sabotage* them.

Let your team *(somehow) know* that this might not be the best solution for the problem.
Add some `@todo`'s and `@note`'s to where you wish your reviewer to pay extra attention.

Show them your code, let them flame you and tell you why something is not ok.
Do you not agree with what they are telling you?
Have a discussion about it; if you have arguments to back you up and they don't, you'll probably win.

Do not try to hide the crappy code in a pile of other changes or bypass the review.
This will make it go south in the long run.

And whatever you do, do not tell them I've sent you.
