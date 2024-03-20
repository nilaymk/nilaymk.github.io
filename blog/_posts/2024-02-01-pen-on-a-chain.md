---
title: "Pen On A Chain"
slug: "pen on a chain"
excerpt: This simple lifehack of putting the "pen on a chain" can teach
         us about a great deal Software Engineering and what makes a
         good 'Tech Leads' and 'Architects'.
---

# A Simple Lifehack

Ever found yourself looking for a pen to fill out a form at the Post Office, or make a note
on your fridge calendar? If you're like me you proabaly spent over 5 minutes looking for it,
or entered into an argument about 'who moved the pen', and then forgot why you needed 
that pen in the first place. The pen not expensive but the disruption caused when it
is mislaid or lost can be reletively expensive. 

<table class="width-scaled">   
   <tr>
     <td><img src="/blog/pen_1.jpg"/></td>
     <td><img src="/blog/pen_2.jpg"/></td>
     <td><img src="/blog/pen_3.jpg"/></td>
   </tr>
</table>

That's when the "pen on a chain" lifehack comes to the rescue. It's a genious hack due to 
its simplicity and effectiveness; no wonder it's so ubiquitous. The goal of this lifehack
is not to prevent theft, but to prevent the 'easy to make' honest mistake of mislaying the pen.

So whats this got to do with Software Engineering and being a good Tech Lead?

# Make "Easy mistakes" Hard To Make

I have worked in teams with very smart people, and even so often enough these teams get
slowed down by 'easy to make' but largly preventable mistakes. One of my responsibility
as a Tech Lead is to pre-empt such mistakes from happening; and when they do happen,
employ simple but high-effective 'lifehacks' to pevent them from happening again and thus minimise
disruptions and maximise flow. I.e. one of my responsibility as a tech lead is
to put the 'pen on the chain'.

Common mistakes like formatting inconsistency, untested branches, untyped parameters, memory leaks,
missing returns, cyclic dependencies, and many more are easy to make. Luckily for us, they are also 
easily detectable and thus preventable with tools like formatters, linters, type chechers, memory leak
checkers, bounds checkers, coverage checkers, etc. Heck, some of these also fix our code for us! 

Employ these tools and have them run automatically in your development workflow (e.g. 
pre-commit/pre-push hooks, CI/CD pipelines). This is a 'pen on a chain' that your team will
thank you for.

> :warning: This may look easy but it can be hard depending on your code size. Also it is fair
> to expect a bit of resistance initially. And almost always it works out in the long run.

# Don't stop there, go beyond code

**Make It Easy To Make "Easy Mistakes" Hard To Make!** Go beyond code and aim for a good
'Developer Experience'. Make this part of your "Architecture", and make this step at a very
early stage of development especially if you're working in startup or scaling fast. 

Over 95% of new projects/code starts with copy + pasting existing project/code. As an architect
you can leverage this developer behaviour and provide a good developer experience.

E.g.
- If you work with multi/micro repos, create a template repo with all the tools, example code, and
  CI/CD setup already in place. This way developers can clone/fork the repo and save a lot of time.
- If you work in a mono-repo setup, you can still create such templates and just not build it for
  your release or deployments.

It's one of those thing that transforms people from architecting good code to being a good
Software/Systems Architect.