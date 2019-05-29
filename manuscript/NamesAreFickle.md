# Names are Fickle

> _One of the biggest sins you can commit is to stop programming when it works._

Keeping a codebase with well-intentioned names is often just about _remembering_ to do so. Everytime we make a change to a codebase, we have to consider the names of things over again. It's easy to leave working code as-is, without considering the debt you've just handed over the next reader. [Change to more about Brandon's idea of pushing back the information everywhere]

Awhile back, I had a method that updated various pieces of account data:

```
public void UpdateAccountInfo(string account_name, int account_owner_id, byte[] logo_image);
```
As you can probably tell by the signature, this method lets you change the name of the account, the owner of the account, and the logo tied to the account.

At some point, it became advantageous to handle the uploading of the logo somewhere else. As part of this update, I pulled the `logo_image` parameter out of this method.

```
public void UpdateAccountInfo(string account_name, int account_owner_id);
```
Down the road, we also decided that an account could have multiple owners. Since the change was fairly large, we decided it was best to manage owners in an entirely separate part of the application. Part of this update naturally required removing the `account_owner_id` from this method.

```
public void UpdateAccountInfo(string account_name);
```
In the flurry of updating code, it's easy to leave the `UpdateAccountInfo` method named as is. But, stripped of most of its original responsibilities, this method is much better named `UpdateAccountName`. That's all it's doing anymore. It also doesn't hurt to shorten the parameter `account_name` to just `name`, since it's obvious at this point what the parameter refers to.

```
public void UpdateAccountName(string name);
```
This change sounds obvious to make, but it's only because I've isolated the discussion of what changed to just this lone method--not the other myriad of method additions, refactorings, and adjustments that come as a natural part of every kind of software change.

When you’ve moved code around your application to get the pieces fitting just right, revisit how you’ve named the methods, properties, and classes that have undergone the facelift. Do these names still make sense? Do the comments around these methods still apply?

When you’re in the same code daily, you might not even notice that the name of a variable or method is misleading because you’re so familiar with it. But, to someone coming into the codebase fresh (or, if you happen to take a few weeks off and come back later), misleading names will be detrimental to their understanding of the system.

There won't be a failed unit test or compiler warning telling us that a construct's name is no longer relevant. That's why they're so often left unchanged. So, remember to look for opportunties to tighten a construct's name everytime something changes about that construct. Nothing else will automatically remind you to do so.
