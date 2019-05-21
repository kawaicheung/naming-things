# Hunting for Names

[The idea of lines of code is no longer as important when every char counted -- readability is far more important]

The first time I ever played around with code was as a 7-year old. My brother and I had convinced our parents to get us our first family computer -- an Apple IIc. As part of the purchase, the local computer store also subscribed us to a year's subscription to _Nibble_, a magazine for Apple II computer users focusing on hobbyist programming. I started reading it because it had a funny name.

My favorite section of the magazine was on the back page of each issue, called the "One- And Two-Liners". Readers would submit programs that could be written in just one or two lines of AppleSoft BASIC. You would then type these lines of code meticulously into the console, hit ENTER, and pray for the thing to work. Usually, the thing was some sort of graphical rendering.

Here's an example of an actual one-liner from an edition of _Nibble_ https://csc.lsu.edu/~kooima/misc/nibble/nibble2.jpg.

![A one-line program that produces a 3D-rendering of a missile's nose cone](images/nibble.png)

[Need to add: Collab/outreach example - simply putting names to conditionals]

[Git before: https://github.com/GetDoneDone/donedone-2.0/blob/ccc52e4c45e95bff06df2c863d10dec8e75858a9/Services/Accounts.cs]

[Git after: https://github.com/GetDoneDone/donedone-2.0/blob/892c7d69ac5ff9aa98d3c85df3f009408eeb8504/Services/Accounts.cs]

Making your codebase clearer isn't just about naming things better--it's also about finding things to name that weren't being named before. 

A common place I hunt for is anywhere I see bits of business logic using only the properties of an object in a place _other_ than inside the object itself. There's usually an easy way to name that logic and push it back into the object. The object, in turn, gets more powerful and self-sufficient. It can _do_ a lot more than it once did, and that helps any other code that interacts with instances of that class in the future.

In this example, I have a `Person` class that houses some basic information used throughout my codebase.

```C#
public class Person
{
    public string FirstName;
    public string LastName;
    public DateTime LastAccessTimestamp;
    public AccountRoleType Role;
    
    ...
}
```
Instances of `Person` naturally spring up all over the place--appearing within various methods and other objects or surfacing in the application's view layer. 

For example, on a person's profile page, I use this object to display a person's name and show a few links to other sections of the application they have access to.

```HTML
<div>
  <h2>@Model.LoggedInPerson.FirstName @Model.LoggedInPerson.LastName</h2>
  @if (Model.LoggedInPerson.Role == AccountRoleType.ADMIN || 
       Model.LoggedInPerson.Role == AccountRoleType.OWNER)
  {
      <a href="...">Edit</a> | <a href="...">Cancel</a>
  }
</div>
```

In another part of the application, I use the person's first and last name to prep notification messages when they update their profile.

```C#
// e.g. "Mary D. updated their profile"
var subject = person.FirstName + " " + person.LastName.Substring(0,1) + "." + " updated their profile.";
```

I also have a method inside of a security class that checks if the person has accessed the application within an hour. If not, I require them to log in again.

```C#
if ((person.LastAccessTimestamp - DateTime.Now).TotalMinutes > 60)
{
    // Automatically log this person out and require them to login again.
}
```

Throughout the application, whereever the `Person` object is used, little bits of business logic against its properties are sprinkled about. Most of these bits feel so inconsequentually minor--simple, one-line constructions and statements--you might not even consider them to be real business logic at all.

I write and see code like this all the time. 

When we write code against an object's properties, it makes sense to do so where it's currently needed. For instance, it was _when_ I was writing the security class method that I needed to find out how long the person has been idle. So, it made sense to write the simple math inside the security method and move on with things.

But, leaving the code as is costs you an opportunity to better define these manipulations and to organize them in a place more conducive to reuse. Namely, back inside the class where all the properties are already defined.

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

Back to the view example, I can push the check for whether this person is an admin or owner as a property of the object itself, and give it a meaningful name. In this case, the check is answering the question, "Does this person have administrative access?". `HasAdminAccess` is a sound name.

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

I could take this entire statement and turn it into a boolean property off the `Person` like so:

```C#
public bool HasPersonBeenIdleForMoreThan60Minutes
{
  get
  {
    return person.LastLoggedIn - DateTime.Now).TotalMinutes > 60;
  }
}  
```

Or, I could push just the calculation of the total days into the object:

```C#
public int MinutesIdle
{
  get
  {
    return (person.LastAccessTimestamp - DateTime.Now).TotalMinutes;
  }
}  
```
The first option cleans up _all_ of the logic from the security method. But, the name feels too particular to me. If someone were just inspecting the `Person` class, they might ask why such a specific property exists. In addition, if I change the requirements around the idle time, I might easily forget to change the name of the property. I don't like this trade off.

Instead, I prefer the latter option. The security method decides what the idle time limit should be (60 minutes) while the `Person` class can tell us how long they've been idle. We leave the appropriate bits of logic in the appropriate places in code.

With these updates, our `Person` class now evolves to something a lot more powerful:

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

And besides reuse, the biggest gain comes from the improved readability of my code. Here's how the improved implementations look like:

```
<div>
  <h2>@Model.LoggedInPerson.FullName</h2>
  @if (Model.LoggedInPerson.HasAdminAccess)
  {
      <a href="...">Edit</a> | <a href="...">Cancel</a>
  }
</div>
```

First, in my HTML markup, the business logic below competes far less with the HTML around it.

```C#
var subject = person.AbbreviatedName + " added a comment";
```

The subject of the email notification can also be interpreted with one glance. You don't spend time focusing on the details of how the person's name is being displayed anymore.

```C#
if (person.MinutesIdle > 60)
{
    // Sign out and require this person to login again.
}
```

Finally, the conditional check on the person's login date can be understood instantly, instead of having to parse (even if for a brief moment) through the date logic.

Keep hunting for places where you can corral bits of logic back into the objects they're derived from. It will do wonders to the clarity of your code.

_By the way, if you are familiar with common anti-patterns in object-oriented programming, what I've described in this chapter is a flavor of Martin Fowler's anemic domain model. If you've never read his take on this anti-pattern, I recommend taking a few minutes to read his take and enjoy his one-of-a-kind writing style:_ 

https://www.martinfowler.com/bliki/AnemicDomainModel.html
