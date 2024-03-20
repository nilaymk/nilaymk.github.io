---
title: "Pen On A Chain"
slug: "pen on a chain"
excerpt: What can a simple lifehack of teach us about Software Engineering
         and what makes good 'Tech Leads' and 'Architects'?
---

# A Simple Lifehack

Ever found yourself looking for a pen to fill out a form at the Post Office, or to make
a note on your fridge calendar? If you're like me you proabaly spent over 5 minutes looking
for it, or entered into an argument about 'who moved the pen', and then forgot why you needed 
that pen in the first place. While the pen not expensive, the disruption caused when it
is mislaid or lost can be quite expensive. 

<table class="width-scaled">   
   <tr>
     <td><img src="/blog/pen_1.jpg"/></td>
     <td><img src="/blog/pen_2.jpg"/></td>
     <td><img src="/blog/pen_3.jpg"/></td>
   </tr>
</table>

That's when the "pen on a chain" lifehack comes to the rescue. It's genious in its
simplicity and effectiveness; no wonder it's so ubiquitous. The goal of this lifehack
is not to prevent theft, but to prevent the 'easy to make' honest mistake of mislaying
the pen.

So whats this got to do with Software Engineering and being a good Tech Lead and Architect?

# Make "Easy mistakes" Hard To Make

I have worked in teams with very smart people, and even so often enough these teams get
slowed down by 'easy to make' but largly preventable mistakes. One of my responsibility
as a Tech Lead is to pre-empt such mistakes from happening; and when they do happen,
employ simple but high-effective 'lifehacks' to pevent them from happening again and thus
minimise disruptions and maximise flow. I.e. My responsibility as a tech lead is
to put the 'pen on the chain'.

Common mistakes like formatting inconsistency, untested branches, untyped parameters, memory leaks,
missing returns, cyclic dependencies, etc. are easy to make. Luckily for us, they are also 
easily detectable with tools like formatters, linters, type chechers, memory leak
checkers, bounds checkers, coverage checkers, etc. Thus they are easily preventable. Heck, some
of these tools also fix our code for us! 

Employ these tools and have them run automatically in your development workflow (e.g. 
pre-commit/pre-push hooks, CI/CD pipelines). This is a 'pen on a chain' that your team will
thank you for.

> :warning: This may look easy but it can be hard depending on your code size and team dynamics.
> Also it is fair  to expect a bit of resistance initially. And almost always it works out in
> the long run.

# Don't stop there, go beyond code

**Make It Easy To Make "Easy Mistakes" Hard To Make!** Go beyond code and aim for a good
'Developer Experience'. Make this part of your "Architecture", and have this at very
early stages of development <u>especially if you're a startup or scaling your teams fast.</u>. 

Over 95% of new projects/code starts with copy + pasting existing project/code. You can leverage
this developer behaviour and provide a good developer experience. Here's how:

1. **Templatize your "pen on a chain":** Create sample projects with all the tools and associated
   rules, CI/CD hooks, CI/CD setup, etc. and have developers copy + paste from it.
  
1. **Make these templates easy to find:** Put these templates in your code repos. If you're a
   multi-repo shop developers can fork this repo. If you're a mono-repo shop it's a folder that
   they copy + paste.

1. **Keep updating the template with more pens on chains:** Found a new "easy mistake" or a better
   tool? Great - go update the template. This way new code can benifit from it.
   > What about consitency Nilay? What about it? Would you rather have consitently bad code
   > and expand your tech debt or some "legacy code" that you will update with new rules
   > when needed?

1. **Do away with that "Coding Standards" and "Best Practices Wiki page!:**
   Yes I'm serious. In my experience, these pages are viewed by developers once during their
   inductiod and rarely ever after. If they do, it is only if 2 developers dont agree at a code
   review and both of these developers could be agreeing on the wrong thing. If this is tooled
   and templatized you generally dont need such wiki pages.

*Going Beyond Code* and considering aspects like "Developer Experience" is what separtes good
"Software/System Architects" from developers who "do architecture".