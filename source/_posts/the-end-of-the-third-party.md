---
title: "The End of the (Third) Party"
description: "Whys and wherefores of integrating third party code."
date: Aug 8 2022
tags: [firebase,nextjs,coding,libraries,philosophy]
---

It's frustratingly stupidly difficult to integrate third party code into an existing project.

When I'm looking for solutions to problems I'm being paid to solve, I can usually think about it from at least two perspectives outside of my own and still make progress. When I try and defend more than three positions, it becomes increasingly difficult to believe that anything shared across all isn't just an aftifact of my bias.

In my day-to-day life as a developer, a feature usually requires me to think of two abstract characters: "the business" and "the next developer". I can have a conversation with these two entities in my head and usually come up with a solution to any problem that I'm being paid to solve which I can feel confident will be ready to present to anyone on my team or within my organization.

The introduction of third party code is like the introduction of a completely new voice to the conversation, and it's the voice of someone who doesn't even go here
<img src="https://assets.untappd.com/photos/2020_01_26/1eae2d347fc227f609dc7881ea273115_640x640.png" alt="She doesn't even go here meme"/>

When you consider adding third party code to your codebase, you're asking for another voice in the room, but it's not a voice you should expect to harmonize perfectly with your existing trio. You should expect this collaboration to be difficult. That being the case, going in prepared is the best way to be successful. The promise of third party code is always the moon and the stars...which are actually free and not special.

The advice that I can provide to people who are considering or are about to add third party code is simple: make sure you're implementation requires third party code, know the ecosystem beyond the most popular libraries, implement based on your needs.

If possible, use only "first party" third party code. If you are implementing Firebase Auth on your NextJS front end, you may not need to install anything beyond the Firebase NPM package, or you may want to install next-firebase-auth and you might want to include react-firebaseui to eliminate as much development time as possible. All of these choices are paths that can result in success, but each choice will result in a convuluted package manifest and pinned requirements. The fewer moving pieces you add, the more time you'll have to spend to implement your solution. The more you add, in this case, nets you a vastly simpler developer experience to get started, but customization will come at the cost of a simple codebase.

...but then again...The nuclear option would be if you can "drink from the firehose". Firebase has a public API. There's no reason I have to use any extra code, I should be able to do this with "fetch" alone...

Do you hear yourself? You sound crazy.