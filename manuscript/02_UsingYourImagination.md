# Naming Objects with Imagination

I think the term _object-oriented programming_ is a bit of a misnomer.

Most "objects" in code don't have a direct representation in the physical world. Introductory textbooks on the subject all seem to use examples involving dogs, cats, and birds (or cars, boats, and planes) as a way of describing how to code classes and interfaces. But, unless you're a programmer at PetSmart, these aren't realistic examples.

We should call it _noun-oriented_ programming because a noun can be a person, place, thing, or idea. And most classes really are ideas...with functionality. 

Classes manifest because a set of functions and properties all have a common purpose and we want to wrap them up in a neat little bundle. But, these classes don't always have an obvious physical translation. This is where wishy-washy names like `UserManager`, `MessagingHelper`, and `AppHandler` are born. 

Working through a codebase littered with class names is like working in a bloated organization where everyone is some form of middle-management. What exactly do you do, _Regional Account Manager Vice Principal_? When we're reading code, we have to dig more to figure out what these things actually mean. When we know there's some property or method out there we want to leverage, we have a harder time remembering where it lives. _Was it in the helper doohickey or in the other manager thingy?_

There are ways around this. Generic names might be a sign that the guts of the class belong elsewhere. For instance, maybe the methods inside that `UserManager` can be moved into the `User` class itself. It might also be a sign that the class does too many things and needs to be split up into smaller pieces. Perhaps there are natural groupings inside that `AppHandler` class--one that handles initialization, one that handles routing, one that deals with exception handling, and so forth. More specific names can be derived from there.

If it's neither of those cases, sometimes we just have to face the reality: A class can be hard to name because it does something that doesn't easily translate in the real world. That's when a little imagination helps. Even when a class is responsible for something that only makes sense in your own code, there's usually some metaphorical noun we can apply to it to make it memorable, and make it easier to recall when you need to revisit the "object" again.

Here are three personal examples.

## "May I see your ticket please?"

The other day, I was looking at some code I wrote awhile back that allows users to reset their password. As was usually the case, I grimaced at the look of the code in its current state. It wasn't horrendous, but there were clear signs a little rethinking could make things better.

The way the reset password function works is not unlike most other apps: A user enters their email address in a "forgot password" form from the app. Then, they receive a link with an encrypted token in the querystring. The information inside the token determines whether the reset link is still valid and which user originally requested the reset.

The token was encrypted on one layer of the stack and decrypted on a completely different layer. Even further, the same token strategy was also used for completing a user's initial registration, but the encryption and decryption logic had been written twice. The code for encrypting, decrypting, and evaluating the token was spread in a few too many places. 

After a bit of chin-scratching and experimentation, I determined the best refactoring was to wrap up this "token" concept in one object. Let the object handle both the encryption and decryption of the token. I got to a place I was very happy with--the guts of the object looked something like this. 

_I've omitted the detail around how the encryption and decryption work since they are unimportant here._

```C#
public sealed class AuthToken
{
  private readonly DateTime _utc_date_issued;

  public readonly int UserID;
  public readonly string EmailAddress;
  public readonly string EncryptedToken;

  public AuthToken(int user_id, string email)
  {
    UserID = user_id;
    EmailAddress = email.ToLower().Trim();
    _utc_date_issued = DateTime.UtcNow;
    EncryptedToken = // Omitted for simplicity...
  }

  public AuthToken(string encrypted_token)
  {
    try
    {
       // Omitted to save trees and unnecessary scrolling...
       UserID = // Deduced from the token...
       EmailAddress = // Deduced from the token...
       _utc_date_issued = // Deduced from the token...
    }
    catch
    {
      throw new InvalidInput("This token could not be decrypted.");
    }
  }

  public bool IssuedWithinMinutes(int minutes)
  {
    if (_utc_date_issued.AddMinutes(minutes) < DateTime.UtcNow)
    {
      return false;
    }
    
    return true;
  }
}
```

Taking a quick walkthrough of this object, you'll notice that there are two constructors. One hydrates the properties of the object with a `user_id` and `email` of the user when a password reset (or registration completion request) is initiated. The timestamp of the token is the moment the object instance is created. 

The other hydrates the same object after a user clicks on the emailed link. The token is decrypted and the same properties are deduced from the decryption process.

The `IssueWithinMinutes` public method allows code elsewhere to decide whether to honor the request. For instance, we might set a password reset link to be valid for only ten minutes, whereas a user registration link could be valid for a few hours.

I'm giddy with the promises of such an object. I'm able to cleanup some duplicate logic used by both the password reset and registration completion functionality. And, whereas the encryption and decryption process once lived in random helper methods on different layers of the stack, they now have a comfortable home.

The last hurdle, however, is a big one--what do I name this thing? My first attempt of `AuthToken` was a half-hearted one to get something down just so I can finish the implementation. But, reading this back again, brings up all sorts of questions and lackluster answers.

* **Does "Auth" mean "Authorization" or "Authentication"?** In this case, it kind of means _both_. That doesn't really help.
* **Does this object really represent the encrypted token?** Kind of. It really represents the encrypted token _in addition to_ the data the token represents. Calling it an `AuthToken` while also having a property with the name `EncryptedToken` is confusing. It's more than just the token. It's easy to get into the trap of naming an object on only part of its reason for being.
* **What is it supposed to be used for?** In the lexicon of object naming, `AuthToken` is about as generic as `UserManager`.

This isn't the right name. What comparable thing possibly exists in the real world like this? 

I think about something that an authority creates for someone, and that person can later exchange for access to do whatever the authority agreed they can do. There's a quality of _redeeming_ this thing. 

A _ticket_ comes to mind. But, that conjurs up thoughts of going to a sporting event or movie, as if there's a specific time and place to redeem it. Not quite right.

A _permit_? Now that feels promising. A permit is valid for a set period of time and it lets someone do something agreed to by an authority until it expires. Plus, a _permit_ is something given to you usually by a governing body, not your local movie theater or sporting venue. 

It doesn't evoke the excitement of having a "ticket". And, let's be honest, password resets and user registrations feel more like the DMV than a rock concert. If you're going to name something unconventional, it's better to have the name provoke similar feelings.

I end up with the name `PermitForUserUpdate`. Implementing the class with this name feels right. It's memorable and even the method `IssuedWithinMinutes()` feels like such a natural method name. Permits in the real world are normally issued. Here's how I can validate a password reset permit hasn't expired yet.

```C#

var passwordResetPermit = new PermitForUserUpdate(encrypted_token);

if (!passwordResetPermit.IssuedWithinMinutes(10))
{
  throw new Exception("This permit is expired!");
}
```

## The pigeon express

When you add a comment in DoneDone, we may send out several email notifications, add alerts to user notification lists, and post a Slack message. But, none of that stuff has to be done the moment you press "Add". To keep the app as scalable as possible, we use a queue service to push off the work that doesn't have to be done in sync with a request. This keeps things speedy on the app-side. 

I have some code that pushes and pulls objects off of these various queues. The app might push a `Messaging` object in response to a comment, and a few moments later, a worker process somewhere else is pulling that object off the queue and handing it off to another process to ship emails or send Slack messages.

Up until now, the queue code was baked into the same namespace as the email and notification shipping code because it didn't need to be anywhere else. But, recently, I added a new and unrelated feature that seemed like a perfect fit to re-introduce the queue concept.

[Continue]

## Let me...

In a DoneDone project, whenever issues are added or updated, the app assembles email objects and pushes them to a queue, each object representing a single email that should be sent to a user. A separate messaging service takes care of popping these objects off of the queue and delivering the messages.

There's a slightly challenging bit when determining who should actually receive these emails. 

You see, by default, anyone assigned to or mentioned on the issue receives an email notification. But, this rule can be usurped by a user's notification settings for that project. A user can opt to:

* Never receive any emails.
* Always receive emails on any new issues, even if they aren't assigned or mentioned.
* Always receive emails on every update within the project no matter what.

So whenever an issue is added or updated, these extra notification settings need to be accounted for before the email objects are pushed to the queue.

You can imagine coding a straightforward implementation of this: You query the database for all custom notification settings for users in this project and store the information in some lists. For instance, you could have three lists that each represent one of the options above, containing the user IDs that opted into each one. Then, at the point in code where these issue emails are assembled, you would reference these lists to decide which users to suppress and which additional users to add, just before they're sent to the email queue.

This is precisely the approach I took. 

Since I like working with objects, I make a class out of this. It takes three integer lists (representing the user IDs opting in to each notification choice) in its constructor and stores them as `private` properties. I then create a few methods used to determine who should receive (or not receive) emails. 

For instance, there's an `AllowEmail()` method which accepts a user's ID and checks whether we should avoid emailing them by referencing that internal list of user IDs for those that never want emails. Similarly, there are a couple of methods to get all the users who want to receive emails even if, by default, they wouldn't. 

I then create an instance of this object to assist in weeding out any assigned or mentioned folks as well as adding any other folks that want to be notified.

The class is small and its work doesn't belong anywhere else. By these measures, it's a really good class. To save on paper (or screen space), I won't even show you the mundane details--what I'm struggling with is what to call this thing.

When in the midst of writing code, the last thing you want is to be hung up on a name. I want to just name this thing quickly and move on. The first thing that comes into my head is, of course, `EmailManager`. 

What does an email manager do? Does it manage the quantity of emails? Does it create them? Does it send them out? 

There's that famous scene in _Office Space_ where the contemptuous consultant asks the agitated employee a simple--yet piercing--question.

> What would you say...ya do here?

That's how I feel about this name. I can't move on just yet. How about `EmailHelper`? Still too ambiguous. What, exactly, is it trying to help with? `EmailAssistant`? Slightly more descriptive, but no more helpful.

In a scenario like this, I try to build a story in my head. I visualize what this object would look like as a _real thing_ in the _real world_. 

* * *

This whole scene resembles something I might imagine in an old-school shipping center. A new shipment of boxes arrive in a large crate, ready to be delivered. These boxes have labels on them indicating who they need to be sent to. One of the workers starts to unload the boxes on the floor, ready to lift them up on a conveyer belt sitting behind a delivery truck.

But, before he lifts the first one, the boss stops him with the palm of her hand. She asks to see each box. One by one, she reads the label on each box, making sure none of the folks on her list have opted-out of these deliveries. She also demands two more boxes be created for a few other folks on her list. The worker scurries off and calls for more.

When he comes back, she nods in approval. She's someone you don't want to mess with. 

What would I call this person? Maybe an officer. And suddenly, the name `EmailOfficer` comes to mind. _Officer_ conveys a kind of _policing_ of emails--exactly what this object's main responsibility is. It's certainly more memorable and visual than either a manager or helper. 

At first, the name feels strange. _Officer_ is not a very programmatic term. But, that's just the point. There are certainly other names that could work just as well. Maybe `EmailCop` or `EmailGateKeeper`. All of these evoke an image in your head far more precise than what the names manager, helper, or handler evoke. 

Ask me today what the class `EmailOfficer` does in the DoneDone codebase, and I can tell you in great detail. I can also tell you I wouldn't have had the same luck had I stuck with a name like `EmailManager`.

* * *

_sqs_courier? wrapper for pulling and adding things to the queue, better than 'helper' or 'manager' as it implies someone transporting things to and from.

_David Heinemeier Hannson goes through a similar exercise in his blog post "[Hunting for Great Names In Programming](https://medium.com/signal-v-noise/hunting-for-great-names-in-programming-16f624c8fc03)", where he discusses his naming choices for a module used to prevent specific IP addresses as user inputs. It's a great read._ 
