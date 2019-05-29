# The Tautologous Name Trap

It's good habit to look for business logic that we can extract into a method or variable. Consistently doing this infuses better meaning into our code. It makes it easier to understand our intentions later on. But, naming these extractions appropriately can be more elusive than it initially appears.

I'm working on a piece of code that processes incoming emails for DoneDone. One of its responsibilities is to send an auto-response email back to the original sender if certain conditions are met.

In the first iteration of this feature, we decide to initiate the auto-response only if the original email was received outside of a company's office hours. 

I've written a method responsible for this work, querying the company's work hours and comparing these date ranges to the current `DateTime` to answer the question. 

```C#
bool isCurrentlyOutsideOfficeHours();
```

In my incoming email handler, I call this method to determine if an auto-response is necessary.

```
if (isCurrentlyOutsideOfficeHours()) 
  sendAutoResponse();
```
The method name `isCurrentlyOutsideOfficeHours()` makes the conditional statement read coherently. Even a non-programmer can read the statement above and deduce it's saying the following:

> If we are currently outside the office hours, then send an auto-response.

A few weeks later, we receive some requests from our customers to allow them to configure company holidays as well as office hours. This way, the auto-response will always be triggered during a company holiday regardless of the time of day.

Back under the hood, we add the new functionality. I then update the auto-response logic to check whether today is a holiday, and adjust my auto-response logic accordingly. This requires another method that handles the dirty work. I call it `isCompanyHoliday()`.

```C#
bool isCurrentlyOutsideOfficeHours();
bool isCompanyHoliday();

...

if (isCurrentlyOutsideOfficeHours() || isCompanyHoliday()) 
{ 
  sendAutoResponse(); 
}
```
Now, I've added more complexity to the auto-response logic -- enough to make me consider consolidating the conditional expression into its own method, then calling the new method in place of the expression. At first, a name like `shouldSendAutoResponse()` sounds sensible. Here's what that would look like:

```C#
bool shouldSendAutoResponse()
{
  return isOutsideOfficeHours() || isCompanyHoliday();
}

...

if (shouldSendAutoResponse()) 
{ 
  sendAutoResponse(); 
}

```
With the new method extracted, the conditional, once again, feels tidy. But, now we have a new problem. The conditional statement is tautologous: 

> If we should send an auto-response, then we should send an auto-response. 

That's not a particularly helpful statement for the code reader. 

When we read the conditional in isolation, we don't know _why_ we're sending the auto-response. We need to trickle into the `shouldSendAutoResponse()` method to find out. The extraction doesn't give us better comprehension; It merely tucks away some logic that we likely will have to drill into later anyways.

I find this happens a lot with these quick extraction exercises. Our immediate inclination is to name the newly extracted method (or variable) after the outcome of the conditions being met rather than the meaning of the conditions. We name the method after the _effect_ rather than the _cause_.

Not only does this create tautalogous statements, but it's less likely we'll reuse the new construct somewhere else. At a glance, we wouldn't think to employ `shouldSendAutoResponse()` anywhere else besides the place in code where we want to send auto-responses. If we named the method after what's _causing_ the sending of the auto-response, we better our chances of reuse later.

So, why are we sending the auto-response? In this case, the cause of sending an auto-response message is because, quite simply, the office is closed. Let's try that.

```C#
bool isOfficeClosed()
{
  return isOutsideOfficeHours() || isCompanyHoliday();
}

...

if (isOfficeClosed()) 
{ 
  sendAutoResponse(); 
}
```
The conditional now reads more meaningfully. In addition, `isOfficeClosed()` is a method that has far more obvious applications than `shouldSendAutoReponse()` does. For instance, I could call it to determine if a chat application should be disabled on the application's help site. This has nothing to do with the auto-response feature.

Tautologous conditionals aren't necessarily bad, though. There are times where describing the _effect_ is the cleanest option. We'll continue with this example. 

Suppose that we introduce a few more conditions to determine whether to send an auto-response. Namely, we let a user toggle auto-responses altogether and we also want to exclude auto-responses to emails that are flagged as spam. My first step is to augment the conditional one more time.

```C#

if (isOfficeClosed() && autoResponseEnabled && !_email.IsSpam) 
{ 
  sendAutoResponse(); 
}

```

The conditional expression bloats up again and it seems ripe for packaging things up. But, I have trouble finding an elegant solution. Is there a meaningful name that could consolidate all or some of the expression? `isOfficeClosed()`, `autoResponseEnabled`, and `!_email.IsSpam` don't appear to have any relationship to one another other than determining whether we should send an auto-response.

In this case, I can decide to either leave things as is, or name the entire expression for what it affects. I choose the latter.

```C#

bool isOfficeClosed()
{
  return isOutsideOfficeHours() || isCompanyHoliday();
}

bool shouldSendAutoResponse()
{
  return isOfficeClosed() && autoResponseEnabled && !_email.IsSpam;
}

...

if (shouldSendAutoResponse()) 
{ 
  sendAutoResponse(); 
}
```
In this case, there's far less chance there would be any another meaning given to `isOfficeClosed() && autoResponseEnabled && !_email.IsSpam` being true. We live with the tautologous argument to keep the statement succinct, and we're also unlikely to miss an opportunity to reuse this method elsewhere.

This brings up an important general idea we see throughout all programming concepts, particularly with naming. We make tradeoffs depending on how a section of code evolves. When I can find an apt name for the _cause_, I choose it. When I can't, I can convince myself that naming an extraction for its _effect_ is still better than not extracting it at all. In the end, the goal is to continually rename things to produce the most coherency given each situation.
