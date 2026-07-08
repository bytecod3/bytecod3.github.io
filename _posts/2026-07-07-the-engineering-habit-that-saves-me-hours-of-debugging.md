---
title: "The Engineering Habit That Saves Me Hours of Debugging"
date: 2026-07-07 00:00:00
categories: [SystemDesign]
---

## Limited Resources
Solve design problems first before solving code problems. Let's face it, as an embedded developer, one thing I dislike the most is having to debug a system I have already commisioned. Something about being *summoned* to fix *your* system just drives me nuts, honestly. However, professionaly, you cannot just decide to let it pass now that you are not feeling like fixing *your* system, especially if you value project ownership, given that you actually built it. 

Now, there are two ways to kick off this troubleshooting: 1) Open the code or 2) Open a design diagram. If you are really sure about what the issue might be, you can go right ahead and open your editor and update, delete or fix whatever is wrong with the code. One limitation with this kind of approach? It always assumes you still have full visibility of the code and how the dependencies were developed, especially if the system has not been commisioned too long ago. Here's the kicker, What happens if your system is two years old and you have built 10 other projects along the way? 

The human mind is volatile, especially for pieces of information that it does not use, or will not have use for, for a long time. This is what happpens for developers. You are always advised to comment your code, not only for other developers to understand your garbage, but for you to understand your garbage later on. And if you are like me, if you really hate to debug projects you have already delivered, you will find it helpful to have a better way of understanding your system at a single glance before opening ten ```.c``` files on your favourite editor. 

Now, the reason you have limited resources, is so that you can use a finite amount of these resources to squeeze out as much productivity from them as possible, that is assuming you do not have an illusion of infinite resources. It is to be noted here that the resource I am referring to here is time. Based on my experience in embedded design, I will explain to you why this is both a good and a bad thing, when it comes to architecture design.

## The Vicious Cycle 
A vicious cycle is a sequence of events where one problem causes another problem, and that new problem makes the original problem even worse. The cycle keeps reinforcing itself, making it increasingly difficult to break out of.  

When it comes to designing embedded systems, code is barely the hallmark of it all. Nowadays we have LLMs (Large Language Models) that can write code better than you can. The real work is in selecting the architecture, designing the architecture and justifying cost. I like to take my time to select and design the architectures, then justifying cost is a no-brainer. Code? That's a breeze. 

Let's design a hypothetical scenario. You have four projects to design in less than two months, this scenario is hypothetical because to me the fact that you have four projects to design in such a short time is in itself horseshit. Anyway, assuming you already have the project requirements, a beginner would immediately jump to writing code, because it is *cool*. After all you are a programmer, right? Wrong. 

Here's the trap. You skip system design to save time, because you do not have it anyway. The architecture becomes unclear. Bugs become harder to identify and diagnose. You spend more time debugging than you would have spent planning. Funny thing? Because you are already behind schedule, you are tempted to skip design again on the next project. And you will skip it believe. And now you are dead. You have four unmaintainable crappy projects. I plead guilty.


## The Approach
If you ever get a hold of my engineering notebooks, they are full of sketch block diagrams, state machine transitions and code flowcharts. The handwriting is unreadable most times yes, but I have the mental print of how the flow is going to be. This is where everything is born. I *always* sketch a block diagram of the system, without any meat in it. From first principles, I am able to follow my intended operation of the code or system from the second the system is turned on. 

![My block diagrams examples]({{"/assets/images/The-Engineering-Habit-That-Saves-Me-Hours-of-Debugging/my-blocks.png" | relative_url}})


I allow myself to dump the first raw, unfiltered thoughts of how the system *might* look like. I say might because after the 4th review, a lot has changed. It is always a battle between opening an editor and taking time to properly design. I take astronomical amounts of time to properly and carefully layout the block diagram. 

In addition to defining the system architecture, block diagrams separates responsibilities between modules of a system. A good embedded system is built from modules with well-defined responsibilities. Given that production level systems have several interdependencies, it is always critical to separate the responsibilities. For example, for my cubesat, I have a camera system. I might divide the system into a camera driver, image compressor, state machine, storage manager and the like. Each of these have a defined responsibility. Before writing a single line of code, I already know the layers of my software and the responsibilities of each. 

Before I discovered that block diagrams exist, I used to jump straight into code. I will testify here that I have been in a position where I had no idea how one module will pass data to the other module, even after writing hundreds of lines of code. Cringe. I found myself stuck in a position where I have written two complete modules, but no way of passing data between them. I had to go back and rewrite huge portions of the code just to create a proper data-flow. Block diagrams will get rid of this type of naivety. They expose and reveal module interfaces between components. The system design will allow you to ask the question: who calls whom? and using which type of API. Which module owns the hardware at a given time? 

![OTA-snippet]({{"/assets/images/The-Engineering-Habit-That-Saves-Me-Hours-of-Debugging/fota-block.png" | relative_url}})


Debugging becomes a breeze. If I do not remember the dependency relationships, I will start by following my block flow diagram. Then I can understand and remember, who was being called by who, and using what interface. Then I can directly go to that specific file or files and fix the issue. Or update, if it is a feature request. This is because, at least in my experience, a bug is always tied to a behaviour. The system behaves in a particular way, which the user (might be you) does not desire. Now, remember that every module has a responsibility? It is super easy to point a specific type of behaviour to a module. For instance, if my image is saved as a truncated set of bytes, I am pretty sure that the culprit is the image compression  module. I won't open the codde, I will open the system design diagram and check who calls the compression. And move on from there. This way, my debug time is cut in half. Sometimes I do not even have to reproduce the bug.


## OK. Yes. But Why?
Rarely have I used code to explain a project to someone. I always use block diagrams. I will always open a block diagram and just explain the flow from start to finish. If it is a state machine, I can guarantee you that someone will be more impressed by the state transition diagram that the transition tables in your code. Unless they insist in seeing the code of course, which in that case that is a difference level of interaction. 

In a team review, please do not open the code at first. Open the system design block diagram. I have done this experiment so you dont have to. Opening a code file and begininning your explanation from there just confuses everyone, might end up confusing you too. 
Opening a block diagram/state machine diagram first? Everyone is happy, even nerds that are not in your domain. You can explain the operation of a freaking space shuttle to a farmer. I'm not sure why you would do that, but the first thing to open, we can bet, won't be the code. 

## How do I practise
Let me lend you a tip. Go take your favourite embedded system project on GitHub, you can open the docs, if the developers were kind enough, and draw your own understanding of the system into a block diagram. I prefer to use the code, searching between files and looking for the APIs and function calls manually, you will discover precious knowledge. I promise you. As an example, here is a diagram I drew some time back to better understand the Mongoose Networking Library:

![Mongoose library block diagram]({{"/assets/images/The-Engineering-Habit-That-Saves-Me-Hours-of-Debugging/mongoose-block.png" | relative_url}})



## Conclusion
You read this far. Good. One of the biggest advantages of block diagrams is that they force you to solve design problems before coding problems. Writing code too soon will lead you to the land of migraines and burn-out if and when your system misbehaves. These are my thoughts. For embedded systems, spending time designing the system will deliver clean firmware that evolves cleanly and easier to maintaing and debug. 