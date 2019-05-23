# Use your imagination

I think the term _object-oriented programming_ is a bit of a misnomer.

Most classes we create in code don't have a direct representation in the physical world. Introductory textbooks on the subject all seem to use examples involving dogs, cats, and birds (or cars, boats, and planes) as a way of describing how to code objects and interfaces. But, unless you're a programmer at PetSmart, those aren't realistic examples.

Really, we should call it _noun-oriented_ programming. Because a noun can be a person, place, thing, or idea. Most classes really are an idea...with functionality. 

Objects manifest because a set of functions and properties all have a common purpose and we want to wrap them up. But, these objects don't always have an obvious physical translation. This is where wishy-washy class names like `UserManager`, `MessagingHelper`, and `AppHandler` are born. 

Working through a codebase littered with class names like these is a helpless feeling. When we're reading code, we have to do more digging to figure out what these things mean. When we know there's a function out there that does what we want, we have to ask ourselves where it lives. _Was it in that helper doohickey or in this other manager thingy?_

There are certainly ways around this. Generic names might be a sign that the guts of the object belong elsewhere. Maybe the methods inside of a `UserManager` can be moved to the `User` class itself. It might also be a sign that the class does too many things and needs to be split up into smaller pieces. 

But if it's neither of those cases, sometimes the class is just hard to name, period. 

Even when an object doesn't have an obvious physical representation, a bit of imagination can bridge that gap and make an object's name memorable. 

* * *

In a DoneDone project, whenever issues are added or updated, the app assembles email objects that are pushed to a queue--each object representing a single email that should be sent to a user. There's a separate messaging service responsible for popping these objects off of the queue and sending out the messages.

The slightly challenging bit is determining who should receive these emails. 

You see, by default, anyone assigned to or mentioned on the issue receives an email notification. But, this rule can be usurped by each user's notification settings for that project. A user can decide to:

* Never receive any email.
* Also receive emails on any newly created issues, regardless of whether they are assigned or mentioned.
* Receive emails on every update within the project no matter what.

So, whenever an issue is added or updated, we need to lookup these extra notification settings for all the other users in the project.


My approach is to create a class that holds this extra notification information. An object is instantiated with three integer lists--each representing a list of user IDs that fall into one of the three special notification cases. Those are tucked away as private lists inside the method. The object is used any time the system is packaging up these email objects to ship to the queue. 

I then expose a few methods used to determine who should receive emails. For instance, there's an `AllowEmail` method which accepts a user's ID and checks whether we should avoid emailing them. Similarly, there are methods to get all the users who want to receive emails even if, by default, they wouldn't. 


So, what should we name an object like this?

A name like `EmailNotificationManager` is tempting. But, looking at this name in isolation, I might assume this class also manages the storing of a user's email preferences. In addition, it doesn't convey the idea that the object helps determine who should get emailed. Scratch that. How about `EmailNotificationHelper`? Still too ambiguous. What, exactly, is it trying to help with?

In scenarios like these, I imagine what the best physical parallel would be. Close your eyes and visualize the messaging service as a _real thing_. What would this object look like?

Here's what I imagined.

The messaging service is a dimly lit shipping center. Once an issue is created or updated, the messaging team receives word from the intercom system. One employee looks up the issue to see who's been added to it. He notes these folks on a clipboard and runs down to an assembly line of workers ready to assemble messages and haul them to the back of a truck for delivery. 

But, before he gives the assembly line his list, another worker stops him dead in his tracks, with the palm of her hand. She asks to see the clipboard. One by one, she makes sure none of the folks on the list have opted-out of these message deliveries. She also adds anyone else that should be on this list but isn't. Then, she hands the clipboard back to the worker and, with a nod, tells him to move along.

I imagine this person with a stern face dressed in a police uniform. And suddenly, the name `EmailNotificationOfficer` comes to mind. `Officer` conveys a kind of policing of emails -- exactly what this object's main responsibility is. It's certainly more memorable and visual than either a manager or helper. 

At first, the name feels strange. "Officer" is not a very programmatic term. But, that's _just the point_.

The more I've let the name sit, the more correct it feels. If you asked me a month later what the class `EmailNotificationOfficer` does in the DoneDone codebase, I can tell you in great detail. I can also tell you I wouldn't have had the same luck had I stuck with a name like `EmailNotificationManager`.
 
