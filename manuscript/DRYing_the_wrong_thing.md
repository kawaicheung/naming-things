# DRYing the wrong thing

Sandi Metz has a wonderful discussion of the perils of making [the wrong abstraction](https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction). She argues that duplication isn't necsesarily a bad thing--it's far better than trying to prematurely abstract code some code that _happens_ to be the same right now.

I was reminded of her discussion recently when I was trying to clean up some code around a set of kludgy email methods in DoneDone. Let me explain.

In DoneDone, we use a few different email services for different reasons.
