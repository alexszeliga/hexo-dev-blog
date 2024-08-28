---
title: "Note to Self: Always Use Marquee HTML Elements"
description: "If you only use bad SEO, they can't hurt you."
date: Jun 19 2022
---

The last time I worked on this blog, I was deeply dissatisfied with my job, while being incredibly happy about where I found myself in my career. Now I find myself with an entirely different set of reasons for wanting a platform to publish things. I've mostly ghosted all social media. I have a great job on an awesome team writing code that is helping me to reinforce my skills as a software developer. My work has me using a very popular framework that is changing rapidly and uses an almost unknown language. While I've flirted with the idea of (seriously) adding Content Creator to my byline, I've never had anything to say that people might actually want to listen to.

But that might be changing. I've been slowly but surely shining light into the darkest corners of my company's mobile app. Before I arrived, it was created using Flutter. At the time, the promise of being able to target iOS and Android with one codebase was a hand-in-glove fit for the small team tasked with creating the app. In large part, it was built by a single developer and was deployed to production with more than a few anti-patterns deeply rooted. Libraries and plugins were used whenever the complexity of the task obscured the way forward, but there wasn't always time to wrap dependency behavior with semantic layers.

Upon joining the project, I discovered that we were version-locked because of dependencies that were version-locked to dependencies, which couldn't be upgraded because the repository where said dependencies' dependencies lived was no longer operating. And while all this dependency hell sounds bad, features were also added nightly without testing and no canonical example of the production codebase exists.

I've written web code my whole life, so I haven't had to compile software since the 90s. On top of that, I've been interested my whole life, but this is my first gig as a full stack developer. So I was on the struggle bus trying to learn the following (all at once):
- Flutter
  - Both 1.22 and 2.8 syntax
- Dart
- Gradle
- Android emulators
- Compiler errors
- iOS and Android build processes
- Improve a codebase that can only be described as "The Diary of a Madman's Fever Dreams"

So partially, this is a preamble. Also, it's an excuse as much as anything. Long story short, I'm gonna start writing about Flutter soon because the SO articles are garbage and there's basically nothing outside of the procedurally generated docs. The first topic I will probably tackle is how to have a collaborative multi-platform app that uses secrets in a fluent way. Unless I'm seriously missing something, there's not really a good way to handle .env files in Flutter and Dart in such a way that they will be available at build time as well as run time. You shouldn't need to have a separate place for build/compile time secrets and a place for runtime secrets.