# Introduction

Here's a little insight into what I'm thinking about when I write code.

So, I'm working on--what I'll call for now--a data migration feature. It's for a new version of an issue tracking tool I helped create ten years ago called DoneDone. For the past handful of years, I've been its sole developer. Being the lone developer on a product for that long has some drawbacks, but for someone like me, I crave this kind of quiet, sustained work--like a scupltor chiseling away endlessly at this large piece of stone.

The goal of this feature is to give our customers an easy way to bring their existing data over from the old version of DoneDone (which we call _Classic_) over to this new version. I wish I could tell you the process is a simple mapping of database tables and columns from Classic over to the new version, but it's not. The new DoneDone is markedly different than its predecessor: Some data maps simply, other data requires some massaging, and some stuff simply can't be mapped at all.

For the next six days, I work away at this feature--the whole bit. I develop a screen to sign in to the old system, one to let users choose the projects they want to move over, and one to see the progress of their migration request. On the backend, I work on a number of database updates to store these requests. I then write an out-of-band service that picks up these requests to perform the dirty work of moving this data over cleanly. Then, there are other tangential pieces like emailing the requester when the migration is complete or notifying us of any errors.

It's intense work but I get it all done and tested. Things go suprisingly smoothly for such a large addition.

> "One of the biggest sins you can commit is to stop programming when it works." -Brandon Rhodes

When the dust settles, I give my code another onceover--it's like re-reading a manuscript from the beginning again with a fresh set of eyes. You tend to pick out things you don't like about your code best that way.

I always like to use the same words we use to talk about the product in the codebase itself. This avoids any unnecessary mental mapping between talking about the system and actually writing code for the system. So naturally, my codebase is littered with the word _migration_ now. There's a `ClassicMigrator` project in my solution, methods named `QueueMigrationRequest()` and `MigrateClassicProjects()`, object properties like `EligibleForMigration` and `HasMigratableProjects`. There are models, views, and controllers with the derivatives of `Migrate` sprinkled around. The copy on the application uses the words _migrate_ and _migration_ too.

But, this word is really eating at me. Mike (my business partner) and I have been using the word "migration" in reference to this feature the whole time. Sometimes you use a term so much that you no longer think about what it actually means; You just know what it's _supposed_ to mean. It's an important word to get right.

But, in reality, we aren't migrating data. Migrate has this connotation that something is leaving one place to go to another, like a flock of birds migrating south for the winter. In our case, data isn't leaving the Classic version. That data is still there, untouched, after a migration. This way, Classic users can play around with the new system using their existing data but don't have to leave Classic if they don't like the new system.

Migrate is actually a very misleading word. Using that word in the application copy might make customers apprehensive about their existing data. Using that word in code might confuse future developers.

So, next, I think about replacing migration with _copy_. But, copy isn't right either. As I mentioned, this data transfer isn't a literal copy. There are some things that don't translate perfectly, or at all. Copying also seems like a fast, mindless operation--a simple `CTRL+C`, `CTRL+V` exercise. That's not what this is.

I told Mike about my conundrum. Immediately, he suggested using the word _import_ instead. Ah hah!

There's a heftiness to the word _import_. Whenever I think about importing data, I think metal gear icons spinning slowly and CPU graphs spiking. Even when you import things in the real world, it has that same feeling of heft--huge cargo ships coming to port across the ocean lugging a large amount of goods.

And unlike migrate, the word import doesn't feel like data is leaving one place and going to another. I typically think of importing data from a file I've uploaded, where I know that data isn't being removed from the file. I also don't necessarily expect this one-to-one mapping between my data and the imported data. Import seems like the perfect word to use.

So, I end up substituting _migrate_ (and all its various derivatives) with _import_.

* * *

If you haven't figured it out already, I'm the kind of programmer that obsesses over these seemingly inconsequential verbal details. Finding that just-right name gives me the same kind of adrenaline boost I get after I've solved a difficult problem or figured out a much cleaner approach to an ugly solution. 

Naming things in code deserves as much attention as all the other skills we preach in programming---like good architecture, continuous refactoring, or testing. While all the other best practices are critical, they share the common drawback that you cannot see them right away. It's only after a bit of digestion of a codebase that you reap its benefits. 

On the other hand, there's an immediate payoff to a codebase with well-intentioned names. They are the first things the programmer sees when opening up the codebase. They make code instantly more approachable. The best part is you can reap these benefits without a substantial amount of work.

After all, isn't that what we're all striving for?
