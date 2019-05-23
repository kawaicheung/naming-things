# Use your imagination

I think the term _object-oriented programming_ is a bit of a misnomer.

Most classes we create in code don't have a direct representation in the physical world. Introductory textbooks on the subject all seem to use examples involving dogs, cats, and birds (or cars, boats, and planes) as a way of describing how to code objects and interfaces. But, unless you're a programmer at PetSmart, those aren't realistic examples.

Really, we should call it _noun-oriented_ programming. Because a noun can be a person, place, thing, or idea. Most classes really are an idea...with functionality. 

Objects manifest because a set of functions and properties all have a common purpose and we want to wrap them up in a neat little bundle. But, these objects don't always have an obvious physical translation. This is where wishy-washy class names like `UserManager`, `MessagingHelper`, and `AppHandler` are born. 

Working through a codebase littered with class names like these is a helpless feeling. When we're reading code, we have to do more digging to figure out what these things mean. When we know there's a function out there that does what we want, we have to ask ourselves where it lives. _Was it in that helper doohickey or in this other manager thingy?_

There are certainly ways around this. Generic names might be a sign that the guts of the object belong elsewhere. For instance, maybe the methods inside that `UserManager` can be moved into that `User` class itself. It might also be a sign that the class does too many things and needs to be split up into smaller pieces. 

If it's neither of those cases, sometimes we just have to face the reality: A class can be hard to name because it does something that doesn't really exist in the real world.

That's when a little imagination helps.

* * *

In a DoneDone project, whenever issues are added or updated, the app assembles email objects and pushes them to a queue, each object representing a single email that should be sent to a user. There's a separate messaging service responsible for popping these objects off of a queue and sending out the messages.

The slightly challenging bit is determining who should receive these emails. 

You see, by default, anyone assigned to or mentioned on the issue receives an email notification. But, this rule can be usurped by a user's notification settings for that project. A user can opt to:

* Never receive any emails.
* Always receive emails on any new issues, even if they aren't assigned or mentioned.
* Always receive emails on every update within the project no matter what.

So, whenever an issue is added or updated, these extra notification settings need to be accounted for before the email objects are pushed to the queue.

You can imagine coding a straightforward implementation of this: You grab all the custom notification settings for users in this project from the database and store the information in some lists. For instance, you could have three lists that each represent one of the options above, containing the user IDs that opted into each one.

Then, at the point in code where these issue emails are assembled, you would reference these lists to decide which users to suppress and which additional users to add.

This is precisely the approach I took. However, I like working with objects. 

So, I packaged these lists up inside a new class and made them `private`. I then created a few friendlier named methods used to determine who should receive (or not receive) emails. 

For instance, there's an `AllowEmail` method which accepts a user's ID and checks whether we should avoid emailing them. This way, I can loop through each person that's assigned or mentioned and confirm they haven't opted out of all emails.

Similarly, there are a couple of methods to get all the users who want to receive emails even if, by default, they wouldn't. 

The class is small and it's work doesn't belong anywhere else. By these measures, it's a really good class. But, what do you call this thing?

A name like `EmailManager` is tempting. But, looking at this name in isolation, I might assume this class also manages the storing of a user's email preferences. In addition, it doesn't convey the idea that the object helps determine who should get emailed. Scratch that. How about `EmailHelper`? Still too ambiguous. What, exactly, is it trying to help with?

In scenarios like these, I imagine what the best physical parallel would be. Close your eyes and visualize it as a _real thing_. What would this object look like?

Here's what I imagine.

We're in a dimly lit shipping center. Once an issue is created or updated, the messaging team receives word from the intercom system. One employee looks up the issue to see who's been assigned or mentioned on it. He notes these folks on a clipboard and runs down to an assembly line of workers ready to assemble these small message boxes to put on a conveyer belt.

But, before he gives the assembly line his list, another worker stops him dead in his tracks, with the palm of her hand. She asks to see the clipboard. One by one, she makes sure none of the folks on the list have opted-out of these deliveries. She also adds anyone else that should be on this list but isn't. Then, she hands the clipboard back to the worker and, with a nod, tells him to move along.

I imagine this person with a stern face dressed in a police uniform. And suddenly, the name `EmailOfficer` comes to mind. `Officer` conveys a kind of policing of emails--exactly what this object's main responsibility is. It's certainly more memorable and visual than either a manager or helper. 

At first, the name feels strange. "Officer" is not a very programmatic term. But, that's _just the point_.

The more I've let the name sit, the more correct it feels. If you asked me a month later what the class `EmailOfficer` does in the DoneDone codebase, I can tell you in great detail. I can also tell you I wouldn't have had the same luck had I stuck with a name like `EmailManager`.

* * *

[Thing about DHH and PrivateNetworkGuard]
 
