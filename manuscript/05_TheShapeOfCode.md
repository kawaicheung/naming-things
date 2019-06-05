# The Shape of Code

Code has a certain _shape_ to it. It's spacing and indentation. Where line breaks are made. How it flows down a page. These are important to how we consume the code.

Sometimes names can disturb that shape dramatically. Consider this simple `for` loop.

```C#
for (int i=0; i < tokens.length; i++)
{
  if (tokens[i].Used)
  {
    usedTokens.Add(tokens[i]);
  }
}
```

To me, this code has good shape. Yes, the indents and spacing are consistent.  But there's something else I notice--the importance of the variables are roughly equal to their size. When I read this code, it doesn't take me long to figure out that there is some `tokens` array, and elements in that array are added to a `usedTokens` list if they are used. 

Here's that same code block again with a few name changes. In each case, I substitute a name with an even more precise name.

```C#
for (int curIndexOfTokenArray=0; 
	curIndexOfTokenArray < tokensArray.length; 
	curIndexOfTokenArray++)
{
   if (tokensArray[curIndexOfTokenArray].Used)
   {
     usedTokensList.Add(tokensArray[curIndexOfTokenArray]);
   }
}
```
If we think only in terms of clarity, then the code should be an improvement. Certainly, `curIndexOfTokenArray` is clearer than `i`. The names `tokensArray` and `usedTokensList` now tell me the object's type without having to dig into them. The names are far more clear. 

But, now we face a more onerous problem. The whole thing just _looks_ substantially more complex. We need more time to digest what it's saying. The code has lost its shape.

While the current index is a critical anchor of a `for` loop, representing it in a verbally meaningful way isn’t. For one thing, an index is common to _every_ kind of `for` loop. In addition, it's scope is small -- it only exists for the duration of a few lines inside the loop. If someone were really confused about the variable name, they only need to look around a small visual radius to get refamiliarized.

The stars of the show here ought to be the `tokens` array and `usedTokens` list. Adding more description to `i` brings a peripheral stage crew member into the spotlight. Couple this with the additional suffix `Array` to `tokens` and the line `if (tokensArray[curIndexOfTokenArray].Used)` takes much longer to parse through it.

The original version values the shape of the entire statement more the level of description each variable brings. There are certain times where this is a better trade-off. This example is one of them.

```C#
for (int i=0; i < tokens.length; i++)
{
   if (tokens[i].Activated)
   {
     usedTokens.Add(tokens);
   }
}
```
A> Sandi Metz has championed the idea of a "squint test" to evaluate how readable your code is. She's even developed a [neat package](https://atom.io/packages/squint-test) for the Atom editor that helps squint for you.

As I look through my own code, I hunt for places where a variable name is getting in the way of the shape of the rest of the code, and then change the name to something smaller. For instance, I recently stumbled across a method in a text search catalog I wrote for DoneDone that leverages a library called Lucene. 

In one method, I defined how new searchable documents are created. The main player in this method is an instance of Lucene's `Document` object that I name `lucene_doc`.
```C#
Document createDocument(ItemEventForSearchIndex item_event)
{
  var lucene_doc = new Document();

  lucene_doc.Add(new Field(_FIELD_item_event_id,...
  lucene_doc.Add(new Field(_FIELD_item_id, ...
  lucene_doc.Add(new Field(_FIELD_project_id, ...
  lucene_doc.Add(new Field(_FIELD_creator_id, ...
  lucene_doc.Add(new Field(_FIELD_created_on, ...
  lucene_doc.Add(new Field(_FIELD_is_convo_thread, ...
  lucene_doc.Add(new Field(_FIELD_is_non_convo_creation,...
  ...
}
```
When I ran across this method again, I felt I could improve its shape by shortening the name of the variable from `lucene_doc` to, simply, `doc`. Not only does it shorten the lines, but getting rid of the underscore precents that variable from visually competing with the other snake-cased names in the code. Overall, it reads smoother without giving up much:
```C#
Document createDocument(ItemEventForSearchIndex item_event)
{
  var doc = new Document();

  doc.Add(new Field(_FIELD_item_event_id,...
  doc.Add(new Field(_FIELD_item_id, ...
  doc.Add(new Field(_FIELD_project_id, ...
  doc.Add(new Field(_FIELD_creator_id, ...
  doc.Add(new Field(_FIELD_created_on, ...
  doc.Add(new Field(_FIELD_is_convo_thread, ...
  doc.Add(new Field(_FIELD_is_non_convo_creation,...
  ...
}
```
Here’s a final example. In this method, `systemTimeZones` represents a collection of objects each describing a time zone. The method loops through this collection, then extracts time zone information to build a list of `DropDownComponents` while marking the passed-in time zone as `Selected`:
```C#
public List<DropDownComponent> BuildTimeZonesDropDownList(string selectedTimeZone)
{
  var result  = new List<DropDownComponent>();
  var systemTimeZones = TimeZoneInfo.GetSystemTimeZones();

  foreach (var systemTimeZone in systemTimeZones)
  {
    var comp = new DropDownComponent(systemTimeZone.Id, systemTimeZone.DisplayName);

    if (systemTimeZone.Id == selectedTimeZone)
    {
      comp.Selected = true;
    }

    result.Add(comp);
  }

  return result;
}
```
The issue with the shape of this code is how repetitive some of the variable names look. There are several names in this method that all look similar at a glance:

* The `systemTimeZones` collection
* The `systemTimeZone` object scoped in the `foreach` loop
* The `selectedTimeZone` parameter
* The `GetSystemTimeZones()` method call. 

Scenarios like these are tricky because each name, in isolation, is appropriately descriptive. No single name is egregiously lengthy, misleading, or overly detailed. 

The problem, however, is when we step back and read the method as a whole. It reminds me a lot of the lines of a Dr. Suess story. One of my two-year-old son’s favorites is _Hop On Pop_, which begins:

> UP PUP - Pup is up. 
> CUP PUP - Pup in cup. 
> PUP CUP - Cup on pup.

The lines of Dr. Seuss, of course, are intentionally dizzying. My son loves to get lost in the pattern of the words and giggles at the absurdity of its rhythm. But, reading dizzying code at 5pm isn't that fun. For this, we need to make some name improvements.

The most immediate, impactful name change I can make is with the name of the `systemTimeZone` object. It appears four times within the method. No other similarly named construct appears more than twice. Also, because its reach is small (only scoped to the `foreach` loop), we can get away with a less descriptive name without doing much harm to the reader’s understanding. 

Let’s try reducing `systemTimeZone` to something shorter, like `tz`:

```C#
public List<DropDownComponent> BuildTimeZonesDropDownList(string selectedTimeZone)
{
  var result  = new List<DropDownComponent>();
  var systemTimeZones = TimeZoneInfo.GetSystemTimeZones();

  foreach (var tz in systemTimeZones)
  {
    var component = new DropDownComponent(tz.Id, tz.DisplayName);

    if (tz.Id == selectedTimeZone)
    {
      component.Selected = true;
    }

    result.Add(component);
  }

  return result;
}
```
I like this change already. Now, the details of the `foreach` loop are much easier to scan. In addition, all of the other similarly-named constructs benefit. They're given more room so that the similarity in their names aren’t as distracting on the eyes as they were before.

Just like our prior example, the variable `tz` feels sized appropriately. Conceptually, a small name like `tz` feels like just one element in a longer-named collection like `systemTimeZones`. These are the kinds of subtle visual cues that all lend themselves to good code shape.

Next, I decide to pare down the variable `systemTimeZones` to just `timeZones`. I originally named this list `systemTimeZones` to follow how the .NET framework named the get method I'm using (`GetSystemTimeZones()`). But, the word _system_ doesn't add any helpful meaning here. This change also helps better differentiate from the `selectedTimeZone` variable used inside the `foreach` loop.

```C#
public List<DropDownComponent> BuildTimeZonesDropDownList(string selectedTimeZone)
{
  var result  = new List<DropDownComponent>();
  var timeZones = TimeZoneInfo.GetSystemTimeZones();

  foreach (var tz in timeZones)
  {
    var component = new DropDownComponent(tz.Id, tz.DisplayName);

    if (tz.Id == selectedTimeZone)
    {
      component.Selected = true;
    }

    result.Add(component);
  }

  return result;
}
```
I could go further and rename the other variables more uniquely, but I don't feel it's necessary. When I read the updated method, I don't feel that dizzying effect of all those similarly named variables that I did earlier.

When we edit names, we shouldn't look at them in isolation. That habit can drive naming decisions that don’t actually benefit the overall readability of the surrounding code. 

Instead, look at the context in which these names live. Find out what’s making a section of code difficult to read and solve the larger problem. It might be a more descriptive variable name, but it might also be a terse one. Let the full context drive those decisions. In the previous examples, I didn't value the precision of certain variable names as much as I did the shape of the lines around them, particularly because the scope of those variables was small.

When editing prose, we read whole sentences and paragraphs to get a sense of readability and style. It’s the same in code-writing.
