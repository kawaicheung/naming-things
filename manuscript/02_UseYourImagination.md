# Use your imagination

If you write in any one of the common software programming languages today, you probably do some amount of object-oriented programming. But, I think the term _object-oriented_ is a misnomer.

Most classes we create in code don't have a direct representation in the physical world. Introductory textbooks on the subject all seem to use examples involving dogs, cats, and birds (or cars, boats, and planes) as a way of describing how to code objects and interfaces. Unless you're a programmer at PetSmart, those aren't realistic examples.

Really, we should call it _noun-oriented_ programming. Because a noun can be a person, place, thing, or idea. Most classes really are an idea...with functionality. 

Objects manifest because a set of functionality and properties all have a common purpose. But, these objects don't always have an obvious physical translation. This is where wishy-washy class names like `UserManager`, `MessagingHelper`, and `AppHandler` are born. Looking through a codebase littered with names like these is a helpless feeling. We have to do so much more mental stretching to figure out where things are and what things mean.

There are certainly ways around this. Sometimes these generic names are a sign that the guts of the object belong elsewhere. Maybe the methods inside of a `UserManager` can be moved to the `User` class itself. Other times, the situation isn't that clear cut.

Even when an object doesn't have an obvious physical representation, a bit of imagination can bridge that gap and make an object's name memorable. 

* * *

In DoneDone, whenever issues are added or updated, the app assembles email objects that are pushed to a queue--each object representing a single email that the messaging service will send to a user. A separate messaging service is responsible for popping these objects off of the queue and sending out the messages.

The slightly challenging bit is who should receive emails. 

By default, anyone assigned who's assigned to or mentioned on the issue receives an email notification. But, this rule can be usurped by each user's own email notification settings. A user can decide to:

* Never be bothered. 
* Receive emails on any new issues.
* Receive emails on anything within the project, regardless of whether they are assigned or mentioned.

The other part of this determination depends on what each person has opted into. As a DoneDone user, you can decide to be notified on every update (even on issues you're not directly added to), on every created issue, or just the issues you're added to. You can also decide to not be emailed on anything, ever. It's the messaging service's job to figure out who should receive an email notification and package up a collection of email objects to ship to the email service.

I ended up extracting out a class to handle this extra determination. The class holds various methods helpful to the messaging service. 

Before these emails are shipped, 

For instance, the `AllowEmail()` method accepts a user's ID and checks whether the user has opted out of all emails. This method is used while looping through all the folks on a newly updated issue to sift out the ones that don't want to be bothered at all. Another property of this class is a `UsersThatAlwaysWantEmail` list. The messaging service grabs this list to ship out emails to anyone else not directly on the issue.

So, what should we name an object like this?

A name like `EmailNotificationManager` is tempting. But, looking at this name in isolation, I might assume this class also manages the storing of a user's email preferences. In addition, it doesn't convey the idea that the object helps determine who should get emailed. Scratch that. How about `EmailNotificationHelper`? Still too ambiguous. What, exactly, is it trying to help with?

In scenarios like these, I imagine what the best physical parallel would be. Close your eyes and visualize the messaging service as a _real thing_. What would this object look like?

Here's what I imagined.

The messaging service is a dimly lit shipping center. Once an issue is created or updated, the messaging team receives word from the intercom system. One employee looks up the issue to see who's been added to it. He notes these folks on a clipboard and runs down to an assembly line of workers ready to assemble messages and haul them to the back of a truck for delivery. 

But, before he gives the assembly line his list, another worker stops him dead in his tracks, with the palm of her hand. She asks to see the clipboard. One by one, she makes sure none of the folks on the list have opted-out of these message deliveries. She also adds anyone else that should be on this list but isn't. Then, she hands the clipboard back to the worker and, with a nod, tells him to move along.

I imagine this person with a stern face dressed in a police uniform. And suddenly, the name `EmailNotificationOfficer` comes to mind. `Officer` conveys a kind of policing of emails -- exactly what this object's main responsibility is. It's certainly more memorable and visual than either a manager or helper. 

At first, the name feels strange. "Officer" is not a very programmatic term. But, that's _just the point_.

The more I've let the name sit, the more correct it feels. If you asked me a month later what the class `EmailNotificationOfficer` does in the DoneDone codebase, I can tell you in great detail. I can also tell you I wouldn't have had the same luck had I stuck with a name like `EmailNotificationManager`.
 
