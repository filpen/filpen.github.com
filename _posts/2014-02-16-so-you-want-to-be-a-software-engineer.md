---
layout: post
title: "So you want to be a software engineer"
description: ""
category:
tags: []
date: 2014-02-16 11:20 UTC
---

I remember the software engineering course I took during my last two semesters of study. We learned UML, OOP, project planning, CASE tools, documentation, a bunch of design patterns and lots of other tactics to tame huge hypothetical abstract software codebases. The problem was that, in fact, they remained **abstract** in my mind.

As students, we never dealt with code longer than some thousands of lines of code. I distinctly remember how proud I was the first time I wrote a C program that was longer than 1000 lines. If you enjoy programming though, chances are that you will get some sort of job involving development. Additionally, you will likely deal with a **large legacy codebase**.

![Ravenholm](/images/ravenholm.jpeg)

> That's the old passage to Ravenholm. We don't go there anymore.
> -- Alyx Vance, Half Life 2

That's what all the nice things you learned during your software engineering lectures were for, right? It turns out it's not that easy to **apply that knowledge**. For example, at the beginning I had a hard time to understand design patterns in practice. All I had was my [GoF book](http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612/) sitting there and judging me from the shelf. I wanted to keep things pragmatical, though, and not overcomplicate things, so I rarely saw where patterns fit the picture. Considering that I was working on a legacy codebase that had no distinguishable patterns whatsoever, it didn't look like a problem.

Little did I know that the codebase I was working on at the time was **very far from the quality standards that the industry strives to achieve**. Lacking any form of automated tests, working on the code was painful and many bug  fixes felt like hacks. I was even more oblivious to the fact that a lot of projects are in this state nowadays all around the world.

Without a mentor, I turned to the Internet. I gathered a list of books about principles we didn't treat in our coursework and started avidly reading. So **this is the list of books I wish I had 5 years ago**, when I was a junior developer working on a huge legacy codebase.

#### [Code Complete](http://www.amazon.com/Code-Complete-Practical-Handbook-Construction/dp/0735619670)

This book I found on [Jeff Atwood's blog](http://www.codinghorror.com/blog/), and it was so influential for him that he named his blog after a mechanism used in the book to identify bad practices, highlighted as "coding horrors". I started reading it during my last year of study, but I went back to it when I started working and found out it's full of practical tips. Very recommended at the beginning of your career.

#### [Clean Code](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)

This one contains a lot of best practices and is a wake up call for professionalism. It is opinionated, but opinions are what a junior developer needs, especially without mentors around. A must read for everyone in our industry working in object oriented land.

#### [Refactoring: Improving the Design of Existing Code](http://www.amazon.com/Refactoring-Improving-Design-Existing-Code/dp/0201485672/) and [Working Effectively With Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052/)

When you know how code is supposed to look like, you want to make your code "cleaner". These two books show you how to get there starting from a sub-optimal codebase. It is not easy and it requires patience and discipline, but it's far from impossible. While you are untangling the coupled concepts from the codebase, you will want to test them. That is why the next book is…

#### [The Art of Unit Testing](http://www.amazon.com/Art-Unit-Testing-examples/dp/1617290890/)

This is a very easy read about unit testing and the concepts behind it. It also contains lots of practical advice on how to structure tests and how to get started if all you have is a blank slate. If you feel the need to have a deeper understanding of testing, stubs and mocks you can always dive into [xUnit Test Patterns](http://www.amazon.com/xUnit-Test-Patterns-Refactoring-Code/dp/0131495054/). It's a bit long, but I found the first part of the book especially useful.

#### [Head First Design Patterns](http://www.amazon.com/First-Design-Patterns-Elisabeth-Freeman/dp/0596007124/)

HFDP takes the GoF patterns and puts them into scenarios where you can see how and why they fit. It has been very useful to get some practical insight that was lacking from the GoF book.

#### [Becoming a Technical Leader](http://www.amazon.com/Becoming-Technical-Leader-Problem-Solving-Approach/dp/0932633021/)

Weinberg wrote this a long time ago, but I believe it still contains very relevant advice. When you know how to write better software, you will want to tell your colleagues and convince your boss, and it is possible that you will meet some resistance in the process. This book tells you how to present your ideas and gain traction, whether you want to pursue the management path or not.

What do you think, what books should junior developers read?

