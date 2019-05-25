# Introduction

Here's a little insight into what I'm thinking about when I write code.

So, I'm working on--what I'll call for now--a data migration feature. It's for a new version of an issue tracking tool I helped create ten years ago called DoneDone. For the past handful of years, I've been its sole developer. Being the lone developer on a product for that long has some drawbacks, but for someone like me, I crave this kind of quiet, sustained work--like a scupltor chiseling away endlessly at this large piece of stone.

The goal of this feature is to give our customers an easy way to bring their existing data over from the old version of DoneDone (which we call _Classic_) over to this new version. I wish I could tell you the process is a simple mapping of database tables and columns from Classic over to the new version, but it's not. The new DoneDone is markedly different than its predecessor: Some data maps simply, other data requires some massaging, and some stuff simply can't be mapped at all.

For the next six days, I work away at this feature--the whole bit. I develop a screen to sign in to the old system, one to let users choose the projects they want to move over, and one to see the progress of their migration request. On the backend, I work on a number of database updates to store these requests. I then write an out-of-band service that picks up these requests to perform the dirty work of moving this data over cleanly. Then, there are other tangential pieces like emailing the requester when the migration is complete or notifying us of any errors.

It's intense work but I get it all done and tested. Things go suprisingly smoothly for such a large addition.

When the dust settles, I give my code another onceover--it's like re-reading a manuscript from the beginning again with a fresh set of eyes. You tend to pick out things you don't like about your code best that way.

I always try to use the same terms when I write code as when I talk about a feature. This avoids any unnecessary mental mapping when I transition between the screen and the rest of the world. So naturally, my codebase is littered with the word _migration_ now. There's a `ClassicMigrator` project in my solution, methods named `QueueMigrationRequest()` and `MigrateClassicProjects()`, object properties like `EligibleForMigration` and `HasMigratableProjects`. There are models, views, and controllers with the derivatives of `Migrate` sprinkled around. The copy on the application uses the words _migrate_ and _migration_ too.

On this re-read, the word is really eating at me. Mike (my business partner) and I have been using the word "migration" in reference to this feature the whole time. It's an important word to get right. 

Sometimes you use a word so much that you no longer think about what it actually means; You just know what it's _supposed_ to mean. Here's the problem: We aren't actually migrating data. 

Migrate has this connotation that something is leaving one place to go to another, like a flock of birds migrating south for the winter. In our case, data isn't leaving the Classic version. That data is still there--untouched--after the migration. I wrote it this way so existing customers can try the new system using their existing data, but if they don't like it, they can just go back to Classic.

Migrate is misleading. Using that word in the application copy might make customers apprehensive about their existing data. Using that word in code might confuse future developers.

So, next, I think about replacing migration with _copy_. It's clear that copying doesn't mean removing the original. But, it isn't right either. As I mentioned, this data transfer isn't a literal copy. There are some things that don't translate perfectly, or at all. Copying also seems like a fast, mindless operation--a simple `CTRL+C` `CTRL+V` exercise. That's not what this is.

I told Mike about my conundrum. Immediately, he suggested using the word _import_ instead. Ah hah!

There's a heftiness to the word _import_, isn't there? Whenever I think about importing data, I think metal gear icons spinning and CPU graphs spiking. Even when you import things in the real world, it has that same feeling of heft--huge cargo ships meandering across the ocean lugging thousands of tons of goods.

And unlike migrate, the word import doesn't feel like data is leaving one place and going to another. In the technical sense, I think of importing data from a file I've uploaded, where I know that data isn't being removed from the file. I also don't necessarily expect this one-to-one mapping between my data and the imported data. Import seems like the perfect word to use.

So, I end up substituting _migrate_ (and all its various derivatives) with _import_.

I'm the kind of programmer that obsesses over these seemingly inconsequential verbal details. Finding that just-right name gives me the same kind of adrenaline boost I get after I've solved a difficult problem or figured out a much cleaner approach to an ugly solution. 

* * *

If you've written code long enough, there's a good chance you're familiar with this quote, attributed to Phil Karlton.

> There are only two hard things in Computer Science: cache invalidation and naming things.

Karlton was a developer at Netscape back in the day and I'm sure he did great work. But, it's this quote that he will be forever remembered by in the programming industry.

Exactly when I first heard it, I do not recall--I just remember chuckling to myself. First, because I can think of many other things that are difficult for me in programming. Second, because naming wasn't initially among those things; I had never thought about naming things as _hard_.

Of course, it can be hard--sometimes excrutiatingly so. Unlike so many other things in programming, a bad (or good) name won't be caught by the compiler. There are no metrics for naming. A bad name won't break your code. A good name won't speed up your build.

Yet, we all have written and read names that confuse, misdirect, conflate, or otherwise mistify us. So, how do you name things well? It's something I've become incredibly passionate about over the years, so much so that I decided to dedicate an entire book to this strange niche of programming topics---all of this stemming from a quote attributed to Mr. Phil Karlton.

Naming is elusive. It has a lot to do with gut, feel, style and even aesthetics. It is, in my humble opinion, the most non-technical of technical subjects. Naming is an art. Good naming has a whole lot of subjectivity.

Naming deserves as much attention as all the other skills we preach in programming---like good architecture, writing "clean code", or rigid testing. While all the other best practices are critical, they share the common drawback that you cannot see these things right away. It's only after digesting the codebase and working with it for awhile that you reap its benefits. 

On the other hand, there's an immediate payoff to a codebase with well-intentioned names. They are the first things the programmer sees when they dig in, perhaps on their way to fixing a bug. They make code instantly more approachable. The best part is you can reap these benefits without a substantial amount of work.

After all, isn't that what we're all striving for?
