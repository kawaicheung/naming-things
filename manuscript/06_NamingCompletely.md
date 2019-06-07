# Naming Completely (Using names as a guide)

Most of us like to think of a codebase as a collection of distinct little modules--objects, methods, namespaces, libraries--all running independently with know knowledge of each other's implementations. That's how we achieve a maintainable codebase, one where we can update something over here and not worry about an unforseen impact over there.

That's certainly achievable. But, when we're running an app, the reality is everything is running under the same ecosystem, as a single unit. A typical request that pushes data from a browser to the database and returns a response probably traverses hundreds of components---ones we've built, ones we've inherited, and ones that live in the depths of frameworks we don't even think about. Our split modules may not know about each other, but they often work with the same data as workers in a request chain.

Here's a great little example by Brandon Rhodes.
[Brandon Rhodes -- naming]

* * *

[Naming can be a guide-- as you go]
There was a major change I had to make when I was building the new version of DoneDone. Whereas I was previously storing all user-generated content in Markdown format, I now realized that it would be far more beneficial to store content as HTML. I didn't make this change lightly--in fact, doing so impacted things up and down the layers of the codebase.

The new DoneDone wasn't a complete rewrite--it was iterative update after iterative update from the existing codebase. So, I wasn't starting from scratch. But, this meant that any fundamental technical change (like swapping out Markdown with HTML) required tedious updates around the entire codebase.
