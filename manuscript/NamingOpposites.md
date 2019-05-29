# Naming Opposites

A lot of code is just about tracking states. An account might be active or inactive. A purchase order could be pending or processed. 

Code tends to be more readable if concepts are stated positively. Anytime I see a variable with a not operator (`!`), I see if there's another variable I can create that expresses the same concept without the `!`. For instance, a conditional statement like `if (!won)` can be improved by replacing it with `if (lost)`.

Usually, the English language has enough breadth in vocabulary that most states have a meaningful opposite. For a piece of functionality that allows us to move a file from the trash, we don't have to say _undelete_. _Restore_ makes perfect sense. It's clear that a file that can be restored is already in the state of being deleted.

But, this isn't always the case. That makes naming a state's opposite succinctly a bit of a challenge. Take this example.

There's a concept in DoneDone called _Workflows_. A workflow defines a series of statuses that an item can be in. A typical issue tracking workflow might have states like "Open", "In Progress", and "Closed". We let users create workflows to tailor them to their own business processes. When a user is ready to use the workflow, it has to be _published_.

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
