---
title: "Pen On A Chain"
slug: "pen on a chain"
excerpt: What can a simple lifehack teach us about Software Engineering
         and what makes good 'Tech Leads' and 'Architects'?
---

# A Simple Lifehack

Ever found yourself looking for a pen to fill out a form at the Post Office, or to make
a note on your fridge calendar? If you're like me you probably spent over 5 minutes looking
for it, or entered into an argument about 'who moved the pen', and then forgot why you needed 
that pen in the first place. While the pen is not expensive, the disruption caused when it
is mislaid or lost can be quite expensive. 

<table class="width-scaled" >   
   <tr>
     <td><img src="/blog/pen_1.jpg"/></td>
     <td><img src="/blog/pen_2.jpg"/></td>
     <td><img src="/blog/pen_3.jpg"/></td>
   </tr>
</table>

That's when the "pen on a chain" lifehack comes to the rescue. It's genius in its
simplicity and effectiveness; no wonder it's so ubiquitous. The goal of this lifehack
is not to prevent theft, but to prevent the 'easy to make' honest mistake of mislaying
the pen.

So what's this got to do with Software Engineering and being a good Tech Lead and Architect?

# Make "Easy mistakes" Hard To Make

I have worked in teams with very smart people and even so the teams get
slowed down by 'easy to make' but largely preventable mistakes. One of my responsibilities
as a Tech Lead is to pre-empt such mistakes from happening; and when they do happen,
employ simple but highly-effective 'lifehacks' to prevent them from happening again and thus
minimise disruptions and maximise flow. I.e. My responsibility as a tech lead is
to put the 'pen on the chain'.

Common mistakes like formatting inconsistency, untested branches, untyped parameters, memory leaks,
missing returns, cyclic dependencies, etc. are easy to make. Luckily for us, they are also 
easily detectable with tools like formatters, linters, type checkers, memory leak
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
   tool? Great - go update the template. This way new code can benefit from it.
   > What about consistency Nilay? What about it? Would you rather have consistently bad code
   > and expand your tech debt or some "legacy code" that you will update with new rules
   > when needed?

1. **Do away with that "Coding Standards" and "Best Practices Wiki page!:**
   Yes I'm serious. In my experience, these pages are viewed by developers once during their
   induction and rarely ever after. If they do, it is only if 2 developers don't agree at a code
   review and both of these developers could be agreeing on the wrong thing. If this is tooled
   and templatized you generally don't need such wiki pages.

*Going Beyond Code* and considering aspects like "Developer Experience" is what separates good
"Software/System Architects" from developers who "do architecture".
