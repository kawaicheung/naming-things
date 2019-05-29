# Naming States

[This one is a little weird because my example isn't really about state naming -- Published and Draft are perfectly good states. It's about the action of "Unpublishing" - maybe I can just state that "IsWorkflowUnpublishable()" is weird. Or maybe I can just talk abotu this more as a cautionary tale]

I can't think of a place where naming is more essential than when dealing with states. An account might be _active_ or _inactive_; A purchase order could be _pending_, _processed_, or _canceled_. Finding that perfect name for each state is hard--especially when an entity can have a lot of states that all have roughly the same meaning.

Billing systems are a great example of this. I've been working with Stripe's API for awhile now and I still have to remind myself what the difference is between subscriptions that are _past_due_, _unpaid_, _incomplete_, _expired_, or _canceled_. They all kind of mean similar things (there's something wrong with the subscription), but understanding their subtle differences are the key to a successful integration and basically having nothing work correctly. 

This isn't a problem with Stripe as much as it is that billing is a complex domain.

Code tends to be more readable if states are stated positively. Anytime I see a state variable with a not operator (`!`), I see if there's another variable I can create that expresses the same concept without the `!`. For instance, a conditional statement like `if (!won)` might be better off if we replaced it with `if (lost)`.

Usually, the English language has enough breadth that most states have a meaningful opposite. For a piece of functionality that allows us to move a file from the trash, we don't have to say _undelete_. _Restore_ makes perfect sense. It's clear that a file that can be restored is already in the state of being deleted.

But, this isn't always the case. That makes naming a state's opposite succinctly a bit of a challenge. Take this example.

There's a concept in DoneDone called _Workflows_. A workflow defines a series of states that an item can be in. A typical issue tracking workflow might have states like "Open", "In Progress", and "Closed". We let users create workflows to tailor them to their own business processes. 

But, a workflow can have its own state too. When a user is ready to use the workflow, it must be _published_.

A user can also--for lack of a better verb--_unpublish_ a workflow. The requirements for a workflow to be "unpublishable" are more than just that it's already been published; There are a few other caveats as well. So, I decide to wrap this logic inside of a convenient little method. My first attempt at a method name is:

```
public bool IsWorkflowUnpublishable();
```

However, there's something unsavory about this name. We could easily mistake this as a check that a workflow *cannot be published* rather than a check that a workflow *can be unpublished* -- two very different things.

The problem is that there just isn't a good adjective that means the opposite of "published". There aren't even a good few words for it. "In draft mode" might be the best way to describe the opposite of published, but a method name like `IsWorkflowAbleToBeInDraftMode()` or `IsWorkflowDraftModable()` feels like we're headed in the wrong direction fast.

In cases like these, it helps to look for a completely different angle to the name. The obstacle with this method name is with the initial word "is". It corners us into having to come up with an adjective to describe the state of the workflow we want to achieve -- and there aren't any good ones.

Instead, we can get the same clarity across by changing the angle of the method name. Instead of starting the name with the adjective-requiring "is", I start with the verb-requiring "can."

```
public bool CanUnpublishWorkflow();
```

Now we might be onto something! Notice I still use the word "unpublish", but as a verb it's no longer ambiguous. It's clear we are checking whether this _workflow can be unpublished_ as opposed to checking whether the _workflow can't be published_.

If the opposite version of a name makes your method ambiguous, instead of pushing harder on finding a different name, see if you can reword the method altogether. You might already have all the pieces you need without knowing it.
