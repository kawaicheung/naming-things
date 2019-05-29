# Naming Opposites

Usually, the English language has enough breadth that most concepts have a meaningful opposite. For a piece of functionality that allows us to move a file from the trash, we don't have to say _undelete_; _Restore_ makes perfect sense. It's clear that a file that can be restored is already in the state of being deleted.

But, this isn't always the case. Take this example.

There's this concept in DoneDone called _Workflows_. A workflow defines a series of statuses available for an issue. A typical workflow might have states like "Open", "In Progress", "Not Reproducible", "Closed" and so forth. We let users create workflows to tailor them to their particular business processes. 

A workflow can have its own status too. It starts as unpublished, and when a user is ready to use the workflow, it becomes published. That's simple enough.

But, once a workflow is published, a user can still _unpublish_ the workflow. The requirements for a workflow to be unpublishable are more than just that it's already been published--there are a few other caveats as well. So, I decide to wrap this logic inside of a convenient little method. My first attempt at a method name is:


[IsWorkflowPublishable...simple... but then...]

```
public bool IsWorkflowUnpublishable();
```

Quickly, I realize there's something unsavory about this name. Someone could easily mistake this as a check that a workflow *cannot be published* rather than a check that a workflow *can be unpublished*. Those are, indeed, two very different things.

The problem is that there just isn't a good adjective that means the opposite of published. _Draft_ might be the best way to describe this concept. It's something I've seen before in blogging tools for instance. But a method name like `IsWorkflowDraftable()` or `IsWorkflowAbleToBeInDraftMode()` feels like we're headed in the wrong direction fast.

In cases like these, I try to look for a completely different angle to the name. After staring at my options for a few minutes, I realize the obstacle lies with the initial word _is_. It corners us into having to come up with an adjective to describe the state of the workflow we want to achieve--and there aren't any good ones that make the method name read clearly.

Instead of starting the name with the adjective-requiring "is", I start with the verb-requiring "can."

```
[and CanPublishWorkflow()]
public bool CanUnpublishWorkflow();
```

Now I might be onto something! Notice I still use the word _unpublish_, but as a verb the intent is no longer ambiguous. It's clear we are checking whether this _workflow can be unpublished_ as opposed to checking whether the _workflow can't be published_.

If the opposite version of a name makes your method ambiguous, instead of pushing harder on finding a different name, see if you can reword the method altogether. You might already have all the pieces you need without knowing it.
