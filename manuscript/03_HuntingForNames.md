# Hunting For Names

I'm sure you've had dreams of devoting an entire week to just cleaning up code for a work project. But, that wouldn't sit well with exactly 99% of managers and 100% of clients out there. The fact is, we have to clean up--surreptitiously--as we go. 

The good news is, you don't need a long stretch of dedicated time to make positive impacts on your code. You can get a lot done in small spurts. One of my favorite exercises is one of the simplest: _Find_ things to name. Look for bits of exposed logic you can define with a name, then replace the logic with the name. 

On the new DoneDone, I notice a conditional on a Vue element that looks like this:
```HTML
<div v-if="!['xs', 'sm', 'md'].includes($mq)">
    ...
</div>
```
If you're unfamiliar with Vue's syntax, no big deal. the `v-if` attribute is Vue syntax that works just like a normal `if` statement, with a Typescript expression inside of it. On an HTML element, it determines if that element should be rendered. In this case, I have a `<div>` element that I only want to show if the Typescript statement `!['xs', 'sm', 'md'].includes($mq)` is true.

But, when you scan this statement, it looks complicated. 

What does this mess mean? Well, it evaluates to `true` if the "extra small", "small", or "medium" media query breakpoints are not hit based on the size of the browser. Put more simply, it checks that the current browser width is sized at least to the width of a normal desktop screen.

I can quickly fix this line by defining the mess with a meaningful name.
```TypeScript
isDesktopWidth(): boolean {
  return !['xs', 'sm', 'md'].includes(this.$mq)
}
```
And, now I get the satisfaction of cleaning up my original code with something so much more approachable.
```HTML
<div v-if="isDesktopWidth">
    ...
</div>
```
Not only does this read better, but I have a property I can potentially reuse again in other parts of my code.

I can spot a line of code like this from a mile away--a messy, overly technical bit of code without a proper home. I often write code like this on the first pass, when I'm just trying to get a feature to work right. But, if I never make that second pass, after testing the code works, then code quickly gets ugly.

It's that second pass where I get to put a name to a piece of logic. Where I get to say _what_ rather than _how_. Where the readability of code improves dramatically. This is among the most basic ways we can improve code.

Take a moment to look at your own code--especially in the "highest" areas of your stack where the semantics really _should_ be more human-readable. You might be surprised how many bits of logic are sprinkled about that could be wrapped up into a meaningful name.

* * *

There are situations where logic that's seemingly full of good names can also be improved by taking things a step further. 

I also look for bits of business logic using only the properties of a single object outside of its class definition. There's usually an easy way to name that piece of logic and push it _back_ into the class definition. 

The object, in turn, gets more powerful and self-sufficient. Done repeatedly, your objects start to _do_ a lot more than they once did. It helps prevent repeated code, and makes those objects more useful to any other code interacting with them down the road.

In this example, I have a `Person` class that houses some basic information used throughout my codebase.

```C#
public class Person
{
    public string FirstName;
    public string LastName;
    public DateTime LastAccessTimestamp;
    public AccountRole Role;
    
    ...
}
```
Instances of `Person` naturally spring up all over the place.

For example, on a person's profile page, I have an instance named `AuthedPerson` used to display an authenticated user's name and a few links to other sections of the application they have access to, if they are an admin or owner in the account.

```HTML
<div>
  <h2>@AuthedPerson.FirstName @AuthedPerson.LastName</h2>
  @if (AuthedPerson.Role == AccountRole.ADMIN || 
       AuthedPerson.Role == AccountRole.OWNER)
  {
      <a href="...">Edit</a> | <a href="...">Cancel</a>
  }
</div>
```
In another part of the application, I use a person's first and last name to prep notification messages when they update an issue.
```C#
// e.g. "Mary D. updated the issue."
var subject = person.FirstName + " " + 
  person.LastName.Substring(0,1) + "." + " updated the issue.";
```
I also have a method inside of a security class that checks if a person has accessed the application within an hour. If not, I require them to log in again.
```C#
if ((person.LastAccessTimestamp - DateTime.UtcNow).TotalMinutes > 60)
{
    // Log out and send to the login screen.
}
```
Throughout the application, whereever the `Person` object is used, little bits of business logic against its properties are sprinkled about. Most of these bits feel so inconsequentually minor--simple, one-line constructions and statements--you might not even consider them to be real business logic at all.

I see code like this all the time. I write code like this all the time. 

When we write code against an object's properties, it makes sense to do so where it's currently needed. For instance, it was _when_ I was writing the security class method that I needed to find out how long the person has been idle. So, it made sense to code their idle time inside the security method and move onto other things.

But, leaving the code that way costs you an opportunity to better name these bits of logic. You also can't organize them in a place more conducive to reuse: Namely, back inside the class where all the properties are already defined.

For instance, in the view example, displaying a person's first and last name might not seem like _logic_, but it is--it represents a person's _full name_. I can easily push this bit of logic back to the `Person` class itself. This also gives me the opportunity to name it something meaningful. 

```C#
public string FullName
{
  get
  {
    return FirstName + " " + LastName;
  }
}  
```
Likewise, the same can be done for the person's name in the subject line of the notification message:
```C#
public string AbbreviatedName
{
  get
  {
    return FirstName + " " + LastName.Substring(0,1) + ".";
  }
}
```
Back to the view example, I can move the check for whether this person is an admin or owner as a property of the object itself, and give it a meaningful name. In this case, the check is answering the question, "Does this person have administrative access?". `HasAdminAccess` is a sound name.

```C#
public bool HasAdminAccess
{
  get
  {
    return Role == AccountRoleType.ADMIN || Role == AccountRoleType.OWNER;
  }
}   
```
The logic against the `Person` object within the security class can be pushed back to the class in a couple of ways. Let's take a closer look at the conditional statement again:
```C#
if ((person.LastAccessTimestamp - DateTime.Now).TotalMinutes > 60)...
```
There are a few ways I could go about moving this logic. I could take this entire statement and turn it into a boolean property off the `Person` like so:
```C#
public bool HasPersonBeenIdleForMoreThan60Minutes
{
  get
  {
    return person.LastLoggedIn - DateTime.Now).TotalMinutes > 60;
  }
}  
```
This cleans up _all_ of the logic from the security method. But, the name feels way too specific. If someone were just inspecting the `Person` class, they might ask why such a specific property exists. In addition, if I change the requirements around the idle time, I might easily forget to change the name of the property. I don't like this trade off.

I could push just the calculation of the idle minutes into the object:

```C#
public int MinutesIdle
{
  get
  {
    return (person.LastAccessTimestamp - DateTime.Now).TotalMinutes;
  }
}  
```
Or, I could convert the logic to a method and allow the caller to pass in the time:
```C#
public bool IdleLongerThanMinutes(int minutes)
{
  return (person.LastAccessTimestamp - DateTime.Now).TotalMinutes > minutes;
}  
```
These examples are both decent options. But, let's go with the former option for now.

With these updates, our `Person` class now evolves into something a lot more powerful:

```C#
public class Person
{
    public string FirstName;
    public string LastName;
    public DateTime LastLoggedIn;
    public AccountRoleType Role;
    
    public string FullName
    {
      get
      {
        return FirstName + " " + LastName;
      }
    }  
    
    public string AbbreviatedName
    {
      get
      {
        return FirstName + " " + LastName.Substring(0,1) + ".";
      }
    } 
    
    public bool HasAdminAccess
    {
      get
      {
        return Role == AccountRoleType.ADMIN || 
               Role == AccountRoleType.OWNER;
      }
    } 
    
    public int MinutesIdle
    {
      get
      {
        return (person.LastAccessTimestamp - DateTime.Now).TotalMinutes;
      }
    } 
    ...
}
```
By moving this logic into the `Person` class, it's now easier to DRY up my codebase. There will likely be other places that require displaying a person's full name or knowing whether they have administrative privileges. Those answers are already baked into the object itself.

And besides reuse, the biggest gain comes from the improved readability of my code. Here's how the improved implementations look like. In my HTML markup, the business logic below competes far less with the HTML around it.
```
<div>
  <h2>@AuthedPerson.FullName</h2>
  @if (AuthedPerson.HasAdminAccess)
  {
      <a href="...">Edit</a> | <a href="...">Cancel</a>
  }
</div>
```
The subject of the email notification can also be interpreted with one glance. You don't spend time focusing on the details of how the person's name is being displayed anymore.
```C#
var subject = person.AbbreviatedName + " added a comment";
```
Finally, the conditional check on the person's login date can be understood instantly, instead of having to parse (even if for a brief moment) through the date math.
```C#
if (person.MinutesIdle > 60)
{
  // Log out and send to the login screen.
}
```
If instead, we went with the third option (creating a method that accepted an idle time), the security logic would look like this:
```C#
if (person.IdleLongerThanMinutes(60))
{
  // Log out and send to the login screen.
}
```
It's not a bad option--it replaces the business logic (`MinutesIdle > 60`) with a name (`IdleLongerThanMinutes(60)`)--but for my tastes, it reads a little awkwardly. I prefer the initial switch.

The best part of this work is that, once you get accustomed to the game, it's not a heavy effort. You can make these simple refactorings quickly and you can stop whenever the time you've devoted is up (or your manager's come back from lunch).

Keep hunting for places where you can corral bits of logic back into the objects they're derived from. It will do wonders to the clarity of your code.

_By the way, if you are familiar with the "anemic domain model" anti-pattern, what I've described in this chapter is essentially an approach to circumventing this. This is a phrase first introduced by Martin Fowler, one of my favorite technical authors. If you've never read his take, [I recommend taking a few minutes to read it](https://www.martinfowler.com/bliki/AnemicDomainModel.html)._ 
