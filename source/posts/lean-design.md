---
title: Lean design
date: 2012-11-21
author: fred
categories: [opinion, design] 
---

After years of frustration with the traditional front end development process, I decided to try and figure out a better way. I'm sharing the rules I came up with in the hope that they may inspire you to experiment with your process.

## Philosophy
Design (I use the term _design_ interchangeably with front end dev) is a process comprised of discrete steps, and the success of the process hinges largely on how the designer allocates their time. For reasons both historical and selfish, the traditional way of creating interfaces is, in my opinion, fundamentally outmoded.

This is the process:
1. Stakeholders discuss the design
2. Sketches are made
3. Photoshop drafts are produced
4. Drafts are critiqued
5. Repeat steps 2-4 incessantly
6. Time crunch. Drafts become final
7. The design is translated into a working interface
8. Go back to step 1.

If you've experienced this process as a stakeholder you already know how shitty it is. Our tools and work processes have completely changed since this way of working was established, so I think design is due an update. Given that we now build our startups the 'lean' way, it makes sense to me to optimise  for iteration speed in the design process.

## Lean startups & lean design
The [lean startup methodology](http://en.wikipedia.org/wiki/Lean_Startup) is centred around eliminating wasted time, and creating a fast and efficient feedback loop. Lean startuppers aim to write the least code to test their hypotheses in order to get the product in front of users as soon as possible - to ship. What is an interface if not a hypothesis: of how the user will interact with your product; of the most and least important functionality; of how much information to display in any one screen.

My belief is that tight iteration loops are equally beneficial for the design process.

## The first rule of Photoshop is… don't use Photoshop
As a designer you are mentally modelling an interface, and then transferring that model into media. In lean design, your job is to translate that mental model into `<div>markup</div>` as quickly as possible. The shorter the route to a working interface, the quicker you can get feedback from real users, and the quicker you can arrive at the 'best' solution.

Unfortunately, Photoshop represents an additional layer in between the mental model and the eventual format, and just serves to slow this whole process down.

There are other reasons why using Photoshop as an intermediary to test your front end design sucks:

1. You haven't used an interface until you've… used it. Critiquing flat designs without being able to interact and actually _use_ the design tends to focus the critique on polish and beauty, rather than on usability. It can create false expectations; a website is an organic medium, which responds not just to the designer but to the user's browser, and the changing nature of content that it displays. It's like falling in love with a air-brushed model, only to be disappointed by their blemishes and imperfections when you meet in reality. Well, kind of. 

2. This misplaced focus distracts stakeholders at the most crucial early stage, which can in turn lead you - the designer - astray. It's also a side effect of how complex Photoshop is, and how bad it is at the fundamentals of web design (typography, column-based layouts, etc.). Photoshop actually encourages you to focus on making that button drop shadow _just right_, or that text anti-aliasing really _pop_ while ignoring the fundamental flaws in your layout.

3. Implementing a sketch in front end code is now much faster than doing so in Photoshop. With HTML5 and CSS3, a lot of the crap that used to be image-based (drop shadows, rounded corners, gradients) and hence required the use of a bitmap editor is now built in to the user's browser.

So trash Photoshop.

## Process
Instead, optimise for time to working interface. My process goes like this:

1. __Sketch__. Sit with stakeholders and establish exactly what [_job the user is hiring your interface to do_](http://hbswk.hbs.edu/item/5170.html), continually focusing the group on this question. Your job at this point is to guide the group to a consensus around your sketch. This consensus will reflect the group's view of your user, and is a great place to start.

2. __Code__. Translate your sketch into a working interface. You don't have to hook it up to the backend yet, but it needs to be clickable and scrollable and all that good stuff. DO NOT STYLE. Use [Bootstrap](http://twitter.github.com/bootstrap/), your own minimal framework or simply a [reset.css](http://meyerweb.com/eric/tools/css/reset/). The emphasis at this point is on high-level stuff like layout, rhythm and focus of the page, NOT on beauty or polish.

3. __Use__! Share the page on a discrete URL, and get everyone to _use_ it. Spend time watching people try to achieve the predefined job. _Time_ is important here: resist the urge to skip this step, and instead spend time with your new page. At this point you are already miles ahead of Mr. Traditional. With no frills in the way, and no sugar on top of the interface, problems are far more obvious and manifest quicker. You have qualitative feedback, and you already see some glaring fails in the page, which you now correct. No shuffling back and forth between media. No missed expectations. Just the simplicity and elegance of a mental model, quickly represented.

4. __Design__ :) Round the corners, drop the shadows, design all the things. Iterate the visual design and polish with stakeholders, all in code. Figure out animations, transitions and the like.

This process focuses on usability and user interaction, and optimises for speed. It really is effective at keeping the designer's attention on the important areas, and avoiding the kind of blinkered work that results in uncreative solutions.