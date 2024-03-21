---
title: "Step away from your code"
slug: "step away from your code"
excerpt: Want to add value? Gently put that keyboard down and step away from your code.
---

# A True Story

It's 2002 A.D and the dawn of commercial grade "camera phones". The hardware
is primitive, processing power limited and memory scarce. We are a proud Engineering Team
at the forefront of pioneering this new technology for the phone manufacturers.
We get a requirement from our biggest customer and a phone giant:

> "For our new phone model, we would like to add 'Pan and Zoom' feature alongside the
> existing colour effects feature. These are flagship features so should be quickly
> accessible. Also Pan and Zoom' must be implemented in software as camera hardware
> doesn't have the capability."

<img style="float: right;" src="/blog/phone_small.jpg">

The engineers get excited. They start mentally writing the code even before they get to
their keyboard. The embedded/DSP (digital signal processor) programmers start writing the
the image scaling and cropping code. The application programmers start with the platform
and application code.

Then we hit a SNAG! You see, phones those days came with keypads, this one with
`Up-Down-Left-Right-Centre` buttons. The problem was how to map the zoom, pan and 
'colour effects' functions to these buttons. `Centre` button obviously to shoot. Mapping
`Left-Right` buttons to cycle through colour effects and `Up-Down` for zoom made
sense too. But we had no buttons left for Pan functionality. We got into
a huddle and came up with a novell solution: `Long press Up-Down-Left-Right` to pan in the
 up, down, left and right directions. We also put in an option in Settings to flip the
behaviour (i.e. `Long press for zoom/colour effects`). Kinda cool right? We thought so too.
It took us about 3-4 weeks to get it done (no Mobile dev frameworks existed then).
And oh boy were we smug!

**OK Who can see the problem here? (or is there a problem)**

- If you thought *'Cool ... you folks rock!'* - you get the "Worst User Need and Value" award.
- If you thought *'Well long press kinda sucks, and Settings option is not necessary'* - you're
  runner-ups for the above award but you thought about the user ... I'll give you that. 
- If you thought *'Your team just wasted a lot of time and provided little value'* - You sir/madam
  are a solution and user focussed individual who strives to deliver value.

**Why?**
Well, the pan functionality requires `ZERO code`. Zero, zip, zilch, nada, null! The user can just
move the phone around to pan. The team wasted time and effort implementing the wrong and completely
useless thing. We focused on the wrong goal, in that we focused on writing code and not solving the
the need of the user/customer.

# Solution Focused Mindset

As programmers, we come across a problem or requirement and we start thinking how can we solve it with
code, often even before we have understood the full problem/requirement. This takes our focus from
better possibilities and alternatives.

But Nilay, *Working software is the primary measure of progress* ... The Agile Manifesto and Principles
say so! Yes actually it does ... *"Working Software"* is mentioned thrice, *"Valuable Software"* once,
and "Solving user's needs" ZERO times :-(. And maybe that's a problem, the focus is on software or code
and not user's needs. Also "Valuable" is subjective - if you're billing by the hour it's
definitely 'valuable' even if completely useless.

But Nilay, this is a Product Owner/Manager problem, they should define requirements correctly.
Not quite, we're paid and allowed to think, and to deliver value to customers by solving their need.
This may require thinking beyond code.

**"Solution Focused Mindset"** requires letting go of pre-conceptions and beliefs (in our case code)
and being open to new possibilities and outcomes in order to provide the appropriate solutions.

# Step away from the code
When my team jumps to solve a user/business need, I usually do the US cop thing in my head: "Sir/Madam
drop your keyboard to the ground, put your hands where I can see them, and gently move away from your
code" :-). Then I start with questions:
- Can we solve this requirement/need without writing code?
- Not possible? Ok what's the minimal code we can get away with?
- What (tech) debt will this code bring?

That often throws up much better alternatives than immediately rushing to code.

> :information_source:
> Our main job is not to produce code, it is to produce software that fits the needs of the
> customer/business with as little code as we can get away with.
> [Youtube: ~Billy Hollis - Lies Developers Tell Themselves.](https://youtu.be/rySTB112z6w)
