# Surfacing Method Parameters

[Needs an intro]

I'll start with an example of a feature addition I worked on for account cancellations. 

For years, we only allowed our customers the option to cancel an account immediately -- it was instantaneous and irreversible. That worked for the vast majority of customers.

I have a method off of a billing repository class who's responsible for invoking the cancellation when requested. It's about as simple a read as you can imagine.

```C#
public class BillingRepository
{
  ...
  public void CancelAccount(int account_id) { ... };
  ...
}
```
And here's how I call it in an account service class.
```C#
_billing_repository.CancelAccount(account_id);
```

However, we'd get the occasional request from a customer that wanted to cancel their account at the end of their term, which could be several months out. Rather than require them to remember to cancel the account in a few months, they'd like the account to _automatically_ cancel on the last day of the term. 

I started implementing by tacking on a parameter to the current `CancelAccount()` repository method. Since there were now two cancellation options, cancelling immediately or at the end of the billing period, I chose the simplest parameter type that fills the need, a boolean. 

I decided to name it `cancel_at_period_end`. Now, I can pass in `true` to handle this new special case, and `false` to handle the original case.

```C#
public void CancelAccount(int account_id, bool cancel_at_period_end);
```

After I've implemented the new code to handle the update, I now update all existing references to this method that handle immediate cancellations. 

```C#
_billing_repository.CancelAccount(account_id, false);
```

However, something feels awkward to me about this line of code.

At a glance, it's hard to tell what the passed-in `false` parameter means. Having just written the updates, it makes sense to me now, but it won't to someone else. They'll have to look at the method signature and perhaps even drill into the method to be sure. 

I feel compelled to add the comment above each call to `CancelAccount()` for clarity.  It also helps differentiate between the new code I'll be adding later to handle the additional option of canceling at the end of the period.

```C#
// Cancel the account immediately...
_billing_repository.CancelAccount(account_id, false);
```

Better. But the method call still feels strange. The standard cancellation case (canceling immediately) accepts the `false` parameter. Passing in `false` as the default just feels odd -- I have to suppress something to perform the default action.

I can get around this pretty quickly though. Since I'm working with a boolean parameter, I can simply flip the name of the option around so that the standard case passes in `true` and update my code accordingly. I swap the `cancel_at_period_end` parameter, with `cancel_now`. And voilà!

```C#
public void CancelAccount(int account_id, bool cancel_now);
```

```C#
// Cancel the account immediately...
_billing_repository.CancelAccount(account_id, true);
```

Now, the default case passes in `true`. But, I've introduced a more onerous problem. By simply reading the `CancelAccount()` method signature, I can't quite tell what passing in `false` would do. What would it mean for `cancel_now` to be `false`? Would it cancel in a day? In a month? At the end of the period? 

At this point, I've exhausted my options with the boolean parameter. While it allows for both options, the options aren't clear from the method signature.

I often find this is the case with booleans when the concept it describes is binary but the options aren't strict _opposites_. That's the case here -- the natural opposite of `cancel_now` isn't `cancel_at_period_end` in the way the natural opposite of `open` is `closed`.

To improve, I could try an `enum` instead.

```C#
enum CancelationType 
{
  NOW,
  AT_PERIOD_END
}

...

public void CancelAccount(int account_id, CancelationType cancel_type);

...

_billing_repository.CancelAccount(account_id, CancelationType.NOW);

```

This is an improvement. I've gotten rid of the ambiguity issues we had with the boolean parameter. It also leaves us better positioned to introduce additional cancelation types in the future.

But, having worked with this software since its beginning, I'd bet there won't be any new cancelation types added soon. It just doesn't feel like one of those features we'd continually augment in the near future. 

Weighing this factor in, an even more readable approach is to directly convey the type of cancelation in the method name. I ultimately decide to create two distinct cancelation methods.

```C#
public void CancelAccountNow(int account_id);
public void CancelAccountAtPeriodEnd(int account_id);
```

With this update, now both the standard and unique cancelation implementations read as informatively:

```C#
_billing_repository.CancelAccountNow(account_id);
_billing_repository.CancelAccountAtPeriodEnd(account_id);
```

I also get an additional benefit from this approach. Breaking the method out into two methods allows me to separate the implementations of each. In the original approach, I'd have to do something like this:

```C#
void CancelAccount(int account_id, bool cancel_at_period_end)
{
  if (cancel_at_period_end)
  {
    // Implementation for canceling at period end
  }
  else
  {
    // Implementation for canceling immediately
  }
}
```

The method body would not only be much longer, but it would have more than one responsibility. Breaking the methods apart not only clarify their use, but will make finding and updating their implementations easier down the road.

----

Another place you'll come across a golden opportunity to create a new method is whenever `null` values are passed to your method. Usually, it's a sign that a new method name ought to be created to handle the passing of the `null` to the original method.

In DoneDone, I have a series of bulk editing methods that live in a services layer. They accept a list of `item_ids` and perform some action on each of them. One of these is a function to bulk update item due dates. Here's the signature:

```C#
public void UpdateDueDates(List<long> item_ids, DateTime? due_date, string comment, User requester);
```

Crawling up to the application layer, here's a snippet of the controller method that calls the bulk edit methods depending on the user's input:

```C#
switch (input.ActionChangeType)
{
    ...
        
    case BulkActions.UPDATE_PRIORITY_LEVEL:
        _items_service.UpdatePriorityLevels(input.ItemIDs, short.Parse(input.Value), input.comment, _LoggedInUser);
        success_alert = "The priority level has been updated for these items.";
        break;   
        
    case BulkActions.UPDATE_DUE_DATE:
        _items_service.UpdateDueDates(input.ItemIDs, DateTime.Parse(input.Value), input.comment, _LoggedInUser);
        success_alert = "The due date has been updated for these items.";
        break;
    
    ... 
}
```

Due dates are optional--hence the nullable `DateTime?` object representing the due date in the parameter list. In a recent feature update, we wanted to explicitly add an option to remove due dates from all items. Because the `UpdateDueDates()` method already gives the option to pass in a `null` value, the update was easy:

```C#
switch (input.ActionChangeType)
{
    ...
        
    case BulkActions.UPDATE_PRIORITY_LEVEL:
        _items_service.UpdatePriorityLevels(input.ItemIDs, short.Parse(input.Value), input.comment, _LoggedInUser);
        success_alert = "The priority level has been updated for these items.";
        break;   
        
    case BulkActions.UPDATE_DUE_DATE:
        _items_service.UpdateDueDates(input.ItemIDs, DateTime.Parse(input.Value), input.comment, _LoggedInUser);
        success_alert = "The due date has been updated for these items.";
        break;
        
    case BulkActions.REMOVE_DUE_DATE:
        _items_service.UpdateDueDates(input.ItemIDs, null, input.comment, _LoggedInUser);
        success_alert = "The due date has been removed for these items.";
        break;
        
    ... 
}
```

But, explicitly passing `null` makes the method invocation less readable. When I inevitably revisit this line of code down the road, I have to trickle into the method to be certain of what it means. Also, in this case, there would never be a reason to pass anything other than a hard-coded `null` -- so why not eliminate the need altogether?

Back on the service layer, I create a new method. It's nothing more than a wrapper to the original `UpdateDueDates()` method:

```C#
public void RemoveDueDates(List<long> item_ids, string comment, User requester)
{
  UpdateDueDates(item_ids, null, comment, requester);
}
```

But with this small addition, I now get to tidy up where I invoke the method on the application layer. Not only do I dissolve the `null` parameter, I also benefit from the name of the new method. In context, the two actions around due dates are much more distinct and clearer to parse.

```C#
case BulkActions.UPDATE_DUE_DATE:
  _items_service.UpdateDueDates(input.ItemIDs, DateTime.Parse(input.Value), input.comment, _LoggedInUser);
  success_alert = "The due date has been updated for these items.";
  break;
        
case BulkActions.REMOVE_DUE_DATE:
  _items_service.RemoveDueDates(input.ItemIDs, input.comment, _LoggedInUser);
  success_alert = "The due date has been removed for these items.";
  break;
```

---

In both examples, adding a _new_ method rather than relying on the parameters of an existing method were fairly easy decisions. In both cases, there was only one variant to the parameters -- requiring one additional method. If the amount of variations are small (say, 3 or less), and these variations are unlikely to change over time, surfacing parameters into method names makes sense.

However, you may find yourself in situations where there are dozens or more variations to a method, and new variations are added to relatively frequently. In those cases, it might be best to lay off creating new methods to avoid such a large increase in methods and adding the burden of making updates to them too often.
