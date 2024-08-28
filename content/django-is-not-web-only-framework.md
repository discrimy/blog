Title: Django is not web-only framework
Date: 2024-08-28

# Django is not web-only framework
I created a Telegram bot to send me a daily forecast (I forgot sometimes to check the weather manually in the morning and don't take an umbrella, and other tools to send me a daily forecast are full of ads). Before development began, I had to choose a stack, and the idea crossed my mind "Why not Django?". And it was a right choice.

Django is not a microframework, so nothing stops you from using it as a non-web framework:

1. Admin panel. It's a selling point for me. It's easy to create a panel and manage your data without building your own solution for that.
2. Models. You don't need to build your own tools to manage your data shape; Django has well-known and easy-to-use tools for that.
3. Utilities. There are a ton of libraries to solve almost any problem a developer can have, and not all of them are web-specific.

One of the problems I encountered while building a Telegram bot was working with sync/async (Django has async support actually, but it's not native and it's easy to misuse sync API in async context), but 
it's specific to the domain (library for telegram bot is async) rather than to Django itself. You can use `sync_to_async` and `async_to_sync` functions to overcome the problem.

It took me 4 hours from idea to deployed and working prototype. And then I'll choose a stack for next project, I'll consider Django.
