---
layout: posts
title: Laravel, Livewire, Tailwind, and Blade; separate your UI concerns!
date: 2025-04-20 08:30:00
tags: [tailwind,livewire,laravel,php,development,frontend,ui]
---

Recently, I had the idea of working on a social media app to toss in my portfolio, so I grabbed the latest version of Laravel and spun up a new project. I had previous experience using [Livewire](https://livewire.laravel.com/), so I added it to the project and then got to the work of setting up auth with [Breeze](https://github.com/laravel/breeze), and when Breeze gave me the option to use Livewire for the view components, I was the person who said "sure" in the Hawaiian punch commercial.

[Here's the repo](https://github.com/alexszeliga/social-overlap) if you want to follow along!

![Configure your way to success with Livewire!](/images/breeze_livewire.png)

I ran the migrations and the tests and looked at what I had configured, and the idea that Livewire is basically what [Taylor has always wanted for the frontend of Laravel](https://laraveldaily.com/post/livewire-inertia-what-taylor-otwell-says) started to make sense. The frontend is built with Vite and Tailwind is configured to build your a tree-shaken css file for any class used in your blade files.

![Configure your way to success with Tailwind!](/images/tailwind_config.png)

## Everyone want's readable mark-up, until it's their turn to review the splitting of the code!

I can already hear the JS framework devs reeling from the idea of using a ten year old PHP mark-up templating library. Do you assume it will be ugly and hard to read with Tailwind classes all over the place, or that it will be similar to other templated mark-up libraries? Would you think that one must use styled components or MUI or React Bootstrap or some other pre-baked component library to have readable markup, isolated logic, and modular components? Well I have great news. The promise of separation of logic, mark-up, and styling can be achieved with Laravel pretty much out of the box.

[Laravel's Blade Components](https://laravel.com/docs/11.x/blade#components) allow a developer to extact any unreadable mark-up into a reusable component. For example, a styled select box is pretty easy to achieve using Tailwind classes, but once you've created the look and behavior you want from it, it's ruined what was once a legible mark-up for a form:

![That's a lot of classes!](/images/select_element.png)

Extracting that to a component allows you to refer to it in templates as `<x-select-input />` or whatever semantic name you might apply. The ability to extract and refactor isn't going to solve all of your problems, however. So here's a few things to do to keep your components organized.

### Isolate Layout from Content
- Do separate your compositional and layout based components from the components that rely on a model or model property to be hydrated.
  - An article card component shouldn't be stored with the User avatar component.
- You should have a paragraph and heading components.
- You should have standardized versions of any branded element that will repeat.

### Component Title and Path Should Document Component Function and Model
- If a component is completely dependent on being hydrated by one model, or a set of properties from a single model, it should live in a folder for that model. Putting all the `User` components together is easier to comprehend than putting all the forms together.

### Be Atomic, Be Sub-atomic. Use 80 Columns for discipline
- If you want to write readable mark-up, then you need to have the discipline to identify when you are writing mark-up that will be hard to read.
- Keeping your editor set to break at 80 or 120 columns sounds crazy, but it means your code could be streamed on a teletype machine and still be readable.
- No matter what you do, mark-up will get highly indented. Agressively splitting your code will allow you to compose using semantic elements 

### Don't Repeat Yourself...much
- D.R.Y. is one of those concepts that sounds much simpler than it actually is in practice. It's better to have two components that both work but do a similar job than it is to have ugly markup.
- To paraphrase a mentor of mine, repeating yourself is not great, but it's better than refactoring code that isn't related to what your working on.
- If you find yourself making a third block of code that is similar to the other two then we have two problems: why do we keep requesting the same feature, and what trends can we follow as a guide to combining all three?

### Components: where the UI complexity goes
- Are you streaming images? Do you have complicated source sets? Do you need to completely flip your class list when this component is in a loading state? Will the user hover the mouse over this? Is this button a UI toggle?

### Set realistic boundaries
- You will still put simple HTML into your views. You will still create a few ad-hoc styled components, and that's okay. If you stop every time you need to create a novel element, it will slow you down. Depending on the size of your app, it may not even improve the readability. Every set of by-laws and every rulebook has some kind of discretion clause, it's always up to you to decide and defend a set of realistic boundaries that reflect the things important to your app, whether that is the security profile, the performance profile, the user experience, the cost, etc.

## Livewire and Volt for server-side code, and layout.
In the days of early-ish React when the class component v. functional component  arguments were still active there was one piece that both teams seemed to agree on: We the devs need to enforce readability because a framework might reinforce a few patterns, but this library does not.

When I first started playing with Livewire, I was put off by how quickly I was able to destroy performance. The more I built, the better I began to understand what Livewire was doing and I was able to establish a set of rules that worked well with the existing Laravel application I was working on.

1. It's a bad idea to try and make a single component that can update multiple rows of a table. Focusing on a single record is good. Focusing on a single piece of data is best.
2. Use Alpine or a client-side library for client side tasks, even though it's much easier and more convenient to use Livewire
3. If your "one record" is a complex query and performance becomes an issue, consider a database view.

#### Be Disciplined

[This component](https://github.com/alexszeliga/social-overlap/blob/main/resources/views/livewire/pages/contribution/edit.blade.php) is used to create posts for a social media site similar to Reddit, where a user can submit a link and users can comment and vote on it. I think it's a great example of how powerful and useful Livewire, Volt, and Blades are.

![Livewire Logic and a bit of validation](/images/livewire-logic.png)
This save method is used when a post (a contribution) is submitted. I love that I can use model methods backed by unit tests all the way out in my UI because that means I don't have to write another test. I don't need to write a contoller and my model's public attributes are enough to keep secrets out of my UI; not even a resource layer in sight.

![Layout up! Markup down!](/images/markup.png)
I'm not an organized person, but I do understand the value of time. The value of time that you can waste is not very valuable, but the value of time that you cannot afford to waste is infinite. Following a set of basic rules like `markup below the return, logic above` makes it easy to know when you have to split code. Once you've coded yourself into a situation where you have to break a rule, that's the moment to step back an analyze the rules and figure out what they can tell you about the game. Similarly saying `I won't junk files up with Tailwinds classes` means "making components" instead of "vibe coding". Following rules like `markup below the return, logic above` will allow you to isolate a problem to a specific file and eventually function faster. Saying `I won't junk files up with Tailwinds classes` means you're reading code, not configs.

![Classes are configs](/images/component-with-classes.png)
When you used styled components or React Bootstrap or MUI or Tailwinds or any other styling library, it requires configuration. Most of these take some kind of hash table to tell your components how to look and act in the browser, that hash table "transpiles" to CSS for the browser. You wouldn't junk up your `main` function with a whole complicated hash table of environment variables in plain text, you configure it elsewhere an pass it in. Do the same thing with your markup and styles because it is actually this easy to make a styled component.

## Try it:

Is this stack a silver bullet for every frontend challenge? Of course not. It's an integrated way to build dynamic user interfaces, often without needing to reach for JS at all. Creating readable, maintainable code always takes conscious effort, no matter the tools. What I've found valuable here is that the path to achieving that becomes much clearer. When you commit to using Blade Components diligently to organize those Tailwind classes, and when you set clear, sensible boundaries for your Livewire interactions, the whole development process starts to feel more harmonious. It becomes less about wrestling with two separate domains (backend and frontend) and more about simply building your application effectively. Treating styles as details to be abstracted into components, keeping logic neatly separated even within Livewire files – these aren't just suggestions, they're sound principles that this stack supports end encourages which will benefit you and anyone else who works on the code down the line.

[The repository](https://github.com/alexszeliga/social-overlap) is available if you'd like to explore it further. Give these techniques a try on a project; you'll likely find the clarity and control are quite rewarding, leading to a smoother, more focused development workflow.
