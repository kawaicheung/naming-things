# Introduction

Here's a little insight into what I'm thinking about when I write code.

So, I'm working on--what I'll call for now--a data migration feature. It's for a brand new version of an issue tracking and customer support tool called DoneDone. For the past handful of years, I've been its sole developer. Being the lone developer on a product for this long has some drawbacks, but for someone like me, I crave this kind of quiet, sustained work--like a scupltor chiseling away endlessly at a large piece of stone. 

The goal of this feature is to give our customers an easy way to bring their existing data over from the old version of DoneDone (which we call _Classic_) over to this new version. I wish I could tell you this is a simple mapping of database tables and columns from Classic over to the new version, but it's not. The new DoneDone is markedly different than its predecessor: Some data maps simply, other data requires some massaging, and some stuff simply can't be mapped at all.

For the next six days, I work away at this feature--the whole bit. I develop a screen to sign in to the old system, one to let users choose the projects they want to move over, and one to see the progress of their migration request. On the backend, I work on a number of database updates to store these requests. I then write an out-of-band service that picks up these requests to perform the dirty work of moving this data over cleanly. There are other tangential pieces I build along the way, like emailing the requester when the migration is complete and broadcasting error notifications.

It's intense work but I get it all done and tested. Things go suprisingly smoothly for such a large addition.

When the dust settles, I give my code another onceover--like re-reading a manuscript from the beginning again with a fresh set of eyes. You tend to pick out things you don't like about your code best that way.

I always try to use the same terms when I write code as when I talk about a feature. This avoids any unnecessary mental mapping when I transition between the screen and the rest of the world. So naturally, my codebase is littered with the word _migration_ now. There's a `ClassicMigrator` project in my solution, methods named `QueueMigrationRequest()` and `MigrateClassicProjects()`, object properties like `EligibleForMigration` and `HasMigratableProjects`. There are models, views, and controllers with the derivatives of `Migrate` sprinkled around. The copy on the application uses the words _migrate_ and _migration_ too.

On this re-read, the word is really eating at me. Mike (my business partner) and I have been using the word "migration" in reference to this feature the whole time. It's an important word to get right. 

Sometimes you use a word so much that you no longer think about what it actually means; You just know what it's _supposed_ to mean. Here's the problem: We aren't actually migrating data. 

Migrate has this connotation that something is leaving one place to go to another, like a flock of birds migrating south for the winter. In our case, data isn't leaving the Classic version. That data is still there--untouched--after the migration. I wrote it this way so existing customers can try the new system using their existing data, but if they don't like it, they can just go back to Classic.

Migrate is misleading. Using that word in the application copy might make customers apprehensive about their existing data. Using that word in code might confuse future developers.

I contemplate replacing _migration_ with _copy_. It's clear that copying doesn't mean removing the original. But, this isn't quite right either. As I mentioned earlier, this data transfer isn't a literal copy. There are some things that don't translate perfectly, or at all. Copying also seems like a fast, mindless operation--a simple `CTRL+C` `CTRL+V` exercise. That's not what this is.

I tell Mike about my conundrum. Immediately, he suggests using the word _import_ instead. Ah hah!

There's a heftiness to the word _import_, isn't there? Whenever I think about importing data, I think metal gear icons spinning and CPU graphs spiking. Even when you import things in the real world, it has that same feeling of heft--huge cargo ships meandering across the ocean lugging thousands of tons of goods.

And unlike migrate, the word import doesn't feel like data is leaving one place and going to another. In the technical sense, I think of importing data from a file I've uploaded, where I know that data isn't being removed from the file. I also don't necessarily expect this one-to-one mapping between my data and the imported data. Import seems like the perfect word to use.

So, I end up substituting _migrate_ (and all its various derivatives) with _import_.

* * *

I'm the kind of programmer that obsesses over these seemingly inconsequential verbal details. Finding that just-right name gives me the same kind of adrenaline boost I get after I've solved a difficult problem or figured out a much cleaner approach to an ugly solution. 

My obsession with naming started from a quote I read some time ago. If you've written code long enough, there's a good chance you've heard it as well.

> There are only two hard things in Computer Science: cache invalidation and naming things. - Phil Karlton

Karlton was a developer at Netscape back in its heyday. I'm sure he did great work. But, it's this quote that he will be forever remembered by in the programming industry.

Exactly when I first heard it, I do not recall--I just remember chuckling to myself. First, because I can think of many things that are difficult for me in programming. Second, because naming wasn't initially among those things; I had never thought about naming as a difficult exercise.

Yet, we all have written and read names that confuse, misdirect, conflate, or otherwise mistify us. So, how do you name things well? Unlike so many other things in programming, a bad name won't be caught by the compiler. There are no metrics for naming. A bad name won't break your code. A good name won't speed up your build.

Naming is elusive. It has a lot to do with gut, feel, style and even aesthetics. It is, in my humble opinion, the most non-technical of technical subjects. Naming is an art. Good naming has a whole lot of subjectivity.

And though there are no metrics for good names, it deserves as much attention as all the other skills we preach in programming---like good architecture, writing "clean code", or rigid testing. While all the other best practices are critical, they share the common drawback that you cannot see these things right away. It's only after digesting the codebase and working with it for awhile that you reap its benefits. 

On the other hand, there's an immediate payoff to a codebase with good names. They are the first things a programmer sees when digging into new code. They make your code more approachable. You can change the name of something and instantly improve your codebase.

This is a book about how names can impact how well our code reads. This isn't a "styleguide". For instance, I'm not going to tell you that all strings have to be prefaced by `str` or camelCase is better than kebab-case. This is about adding meaning into the code we write, regardless of language or development environment.


[Is this necessary?]

* * *

Mike asked me why I cared so much about naming when I happen to be working on code that only I'm going to see (as I mentioned earlier, I've been working on DoneDone as a solo developer for quite some time). Most codebases are touched by a multitude of people---where establishing good names is arguably more important than on a codebase managed by just one person.

My answer is simple: Naming better makes me want to code better. Being the lone programmer on an app motivates me even more because it's a reflection of the quality I like to put into my work. When I revisit code I worked on a long time ago, I get the satisfaction of seeing that quality still present. And when the day comes when someone else contributes to the codebase, they'll benefit from the care I've already put into it.
