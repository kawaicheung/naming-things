# Naming Completely (Using names as a guide)

There was a major change I had to make when I was building the new version of DoneDone. Whereas I was previously storing all user-generated content in Markdown format, I now realized that it would be far more beneficial to store content as HTML. I didn't make this change lightly--in fact, doing so impacted things up and down the layers of the codebase.

The new DoneDone wasn't a complete rewrite--it was iterative update after iterative update from the existing codebase. So, I wasn't starting from scratch. But, this meant that any fundamental technical change (like swapping out Markdown with HTML) required tedious updates around the entire codebase.
