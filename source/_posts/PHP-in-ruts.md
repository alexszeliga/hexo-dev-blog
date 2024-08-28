---
title: "PHP-in-Ruts"
description: "I don't know how to build a web-app with Ruby-on-Rails, but if you hum a few bars..."
date: Aug 21 2024
---
I have spent the last four years of my life as a professional PHP developer working on a Laravel web app for a final mile logistics company. So a lot of what I did was to build specific web pages to support normal business operations like creating, retrieving, updating, and deleting orders, managing customers, managing user rights and permissions, and generating invoices and payments.

For the most part, our database schema was where complexity was housed, and our code was about as transparent as it could be using PHP. My mentor, also my boss, had a lot of experience working on a large eCommerce web app built in Ruby-on-Rails, which was at one time the new and hot framework in web development. As we worked our way up from Laravel 5 to 7 and eventually 10, we started implementing some of the more modern syntax and niceties from these versions. The log-jam started to clear and features really started to fly! So obviously the company downsized it's development team by half and the last bit of code in my portfolio happens to be from when I was in boot camp, so here we go lets build a Reddit clone with Laravel and talk about how it's really slick now.

Without burying the lead, [I've got a good jump on this project](https://github.com/alexszeliga/social-overlap), and I've got a few things to say about Laravel 11, Livewire and Volt.

First, Volt: If you look in the [controllers](https://github.com/alexszeliga/social-overlap/tree/main/app/Http/Controllers) directory, you'll see there are none. They are not necessary when you're using a [Volt route](https://github.com/alexszeliga/social-overlap/blob/main/routes/web.php) and a full page [Volt component](https://github.com/alexszeliga/social-overlap/blob/main/resources/views/livewire/pages/conversation/view.blade.php). Looking more closely, you can see that route model binding works the same way it does with existing Laravel routes. The resulting models can be type-hinted in the `mount` method on the Volt component. One of the really nice improvements added by the Volt package is the ability to have the logic and markup in the same file.

While a lot of the credit I'm giving here is handed out to the new bits that have become easier to implement in later versions of Laravel, the use of Blade syntax and modular Blade components is really the star that allows the Volt components to be oh-so-readable. I assume had I written out all the Tailwind classes in the component I linked above, it would probably be a nightmare to look at. While I'll never be a designer, my time at a marketing firm did help me establish a few good habits when it comes to separation of code from markup, the separation of markup from styling, and how to build reusable UI atoms.

I've pushed fewer than 100 commits to this project as of right now, but I have accomplished the following:

- Implement boilerplate Auth
- Create basic social media models:
    - Conversation: a user creates these by submitting a URL to a community
    - Community: a repository for conversations, users can subscribe by 'claiming' the community.
    - Users can comment in a conversation.
    - Users can comment on comments infinitely.
    - Users can vote on comments and conversations.
- Server side jobs: voting is queued
- WebRTC / websockets: new votes are broadcast and update when a voting job is completed

As always, testing in Laravel is incredibly easy. Without controllers, it seems much easier to figure out what and how to test my models. Coverage is currently around 85%. The issue in coverage emanates from the parts of the Auth system I'm not using. Livewire really does allow you to think of your web application as a set of views that are intrinsically bound to the models they are hydrated by.

While I'm sure Alpine.js is a fine lightweight JS library, but here's what I love about modern Laravel:

- Don't write controllers
- Don't write JavaScript
- Bring unit-testable model behaviors directly into the view