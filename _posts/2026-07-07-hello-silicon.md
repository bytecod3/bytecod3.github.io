---
title: "The Engineering Habit That Saves Me Hours of Debugging"
date: 2026-07-07 00:00:00
categories: [SystemDesign]
---

# Why you have limited resources
Let's face it, as an embedded developer, one thing I dislike the most is having to debug a system I have already commisioned. Something about being *summoned* to fix *your* system just drives me nuts, honestly. However, professionaly, you cannot just decide to let it pass now that you are not feeling like fixing *your* system, especially if you value project ownership, given that you actually built it. 

Now, there are two ways to kick of this troubleshooting: 1) Open the code or 2) Open a design diagram. If you are really sure about what the issue might be, you can go right ahead and open your editor and update, delete or fix whatever is wrong with the code. One limitation with this kind of approach? It always assumes you still have full visibility of the code and how the dependencies were developed, especially if the system has not been commisioned too long ago. Here's the kicker, What happens if your system is two years old and you have built 10 other projects along the way? 

The human mind is volatile, especially for pieces of information that it does not use, or will not have use for, for a long time. This is what happpens for developers. You are always advised to comment your code, not only for other developers to understand your garbage, but for you to understand your garbage later on. And if you are like me, if you really hate to debug projects you have already delivered, you will find it helpful to have a better way of understanding your system at a single glance before opening ten ```.c``` files on your favourite editor. 

Now, the reason you have limited resources, is so that you can use a finite amount of these resources to squeeze out as much productivity from them as possible, that is assuming you do not have an illusion of infinite resources. Based on my experience in embedded design, I will explain to you why this is both a good and a bad thing, when it comes to architecture design.

# The vicious cycle 
