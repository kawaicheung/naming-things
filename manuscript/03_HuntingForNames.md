# What can I name?

One of the things I like to do routinely is clean up the code I'm working on. I know, I know--there are many of you out there who would love to devote a day or two just to clean up code at your company. But, that wouldn't sit well with management or your clients.

The good news is, you don't need a day or two to make positive impacts on your code. You can get a lot done even in a thirty minute session. One of my favorite ways to do this is with an exercise I'll call "the name hunt". 

The way this works is pretty simple. I look for bits of business logic using only the properties of a single object in a place _other_ than inside the object itself. 

There's usually an easy way to name that logic and push it back into the class definition. The object, in turn, gets more powerful and self-sufficient. It can _do_ a lot more than it once did, and that helps any other code that interacts with instances of that class in the future.

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
var subject = person.FirstName + " " + person.LastName.Substring(0,1) + "." + " updated the issue.";
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
Or, I could push just the calculation of the idle minutes into the object:

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
The first option (`HasPersonBeenIdleForMoreThan60Minutes`) cleans up _all_ of the logic from the security method. But, the name feels too specific. If someone were just inspecting the `Person` class, they might ask why such a specific property exists. In addition, if I change the requirements around the idle time, I might easily forget to change the name of the property. I don't like this trade off.

The second and third examples are both decent options. But, let's go with the second option for now.

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
It's not a bad option--and it removes the extra bit


Keep hunting for places where you can corral bits of logic back into the objects they're derived from. It will do wonders to the clarity of your code.

_By the way, if you are familiar with common anti-patterns in object-oriented programming, what I've described in this chapter is a flavor of Martin Fowler's anemic domain model. If you've never read his take on this anti-pattern, I recommend taking a few minutes to read his take and enjoy his one-of-a-kind writing style:_ 

https://www.martinfowler.com/bliki/AnemicDomainModel.html
