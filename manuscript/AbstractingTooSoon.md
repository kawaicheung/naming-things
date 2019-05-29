# Abstracting Too Soon

As I write code, I usually start with names that describe their constructs concretely before gradually iterating toward something more abstract. Proper abstractions need to be made at the right pace. Abstract too slowly and it will be the feature creep that steers the ship. Abstract too soon and the boat will soon be sinking under its own weight.

[Maybe Sandi Metz bit here about championing the idea of duplicate code better than the wrong abstraction?]

Naming should also undergo the same kind of scrutiny. The gradual reshaping of our codebase also warrants constant re-evaluation of the names we give to the constructs in our code. 

It's taxing work. And I think it leads programmers (including myself) to abstract the meaning of a name too early in the process - a desparate attempt to get a name right, once and forever. But I've never found prematurely abstracting a name helpful. In fact, it often inflicts more pain than comfort, just like premature code abstraction does.

I'm going through a refactoring in DoneDone's codebase to consolidate a few data objects that hold very similar kinds of information in slightly different ways. They all revolve around the history of actions applied to an issue.

Ultimately, the properties of these objects manifest in a few different places. On an issue's detail page, the collection of history for a single issue is written out from an `IssueDetailHistory` collection. On the activity dashboard, a list of history on all issues in a given day is extracted from a list of `ActivityHistory` items. When activity occurs on an issue, yet another "history" object is packaged up in a class with a list of emails to send to specific users. A mailing service unpacks the data and formats the right bits of information into emails.

Each of these objects look nearly identical. They all could derive from a single class, but they've all been written separately from each other over time. It's time to consolidate them.

After a fair amount of hair-pulling, I've reduced these classes into a single `IssueHistory` class that can support the needs of each of these three distinct use cases, and more going forward. This also gives me an opportunity to consolidate similar names in the old objects like `CreatedOn`, `CreatedDate`, or `CreateDate` into one consistent name (I like `CreatedOn`, for what it's worth).

In the process of the consolidation, I discover one particular string off of the old history object used by the mailing service. It's a terse description used as the beginning of the email subject, like "New fixer" or "Priority Update" or "Closed". (The rest of the subject is composed of the issue number and title). 

The dilemma I face here is what exactly to call this string. At present moment, it's _only_ used by the mailing service and not by either the detail page or activity dashboard. But, since I've done all the pruning to get toward a single object, I feel compelled to come up with a name that can satisfy all future implementers.

My natural inclination is to call this something like `AbbreviatedAction`, especially because it juxtaposes nicely with the `Action` attribute that stores a more verbose description of the action ("John Doe was assigned to fix the issue" or "The priority was updated to Critical") I go with this name.

After a few days of refactoring the rest of the code, I find myself going back to this name a couple of times not entirely remembering what it's being used for. I even decide to comment the name to remind me it's only being used by the mailing service as part of the subject of the email. 

```C#
// Note: Only being used by the mailing service right now as part of the email subject, but I'm sure it will have other uses one day.
public string AbbreviatedAction
{
  get { … }
}
```

Whenever I have to remind myself what a name means (and eventually surrender to the commenting devils) there's a good chance I've prematurely abstracted the name. 

Instead of finding a name that simultaneously fits anything but nothing in particular, it's best I reduce the scope of the name's meaning to fit how it's being used right now. A name like `EmailSubjectAction` ends up working a lot better.

```C#
public string EmailSubjectAction
{
	get { … }
}
```

Some might worry that a name that's too specific like this might hide the full capabilities of that construct. That, one day, I'll need the very thing that `EmailSubjectAction` brings to the table (a terse description of an issue action) for something entirely unrelated to email subjects and I will never realize it existed. Instead, I'll create a near-identical version of that construct and name it something else.

In my experience, I rarely find this to be the case. If I'm familiar enough with the codebase, I recall some part of it that replicates (or nearly replicates) what I'm trying to implement by thinking about the particular feature ("What I need is just like that first part of that subject line of the emails that are sent when issues are updated!") rather than the particular name of the relevant construct (`EmailSubjectAction`). The more specific name just helps me find where that construct lives, faster.

Others might argue that I really should move this particular property to its own class and inherit the `IssueHistory` class. This way, I can use this class specifically for the emailing case. But, to me, the additional overhead of managing another class just for the sanctity of keeping the name exposed only to those who need it isn't worth the tradeoff. I'm OK with exposing this property to the other use cases especially if it's the only one of its kind. If this starts becoming a theme with other properties, then I might sway the other way. But, it's too early now.

Once that construct is being used in multiple different ways, then I can justifies the broader name change. In my case, `AbbreviatedAction` may eventually be the best name one day, but only when its set of use cases warrants it.
