You've probably noticed by now that a large number of examples I discuss in this book started from refactorings. It makes sense. Refactorings aren't just about shifting code around and removing duplicate bits of logic, they can also fundamentally change the meaning of changed objects and methods. That's why the names have to change.

We have to be careful though. Sometimes a refactoring makes technical sense but not logical sense. I want to talk about a refactoring I did once that, honestly, bothered me for _years_. It always felt off to me but I couldn't articulate why. I couldn't easily spot the problem because the technical rework was completely correct.

We send out a handful of different kinds of emails in DoneDone. 

Some of them are, what I like to call, "in app" emails: Email updates when someone comments or edits an issue, assigns something to you, and so forth. Another set are what I like to call "system" emails: Emails for password resets or completing your account registration. Yet another kind of email is an "external customer" email: You can use DoneDone as a customer support tool and forward your incoming support emails right to it. This means you can also reply back to your customers through DoneDone. Those emails are this third category.

Each of these email groups are treated differently for a few tactical reasons.

One reason is the order of the services we try to send out these emails. There are three different ways we attempt to send out emails. If, on the very rare occasion, an email service is down, the system will try the next service, and the following service. 

Specifically, we can send email through a service called Postmark. The beauty of Postmark is these emails are logged for a certain period of time, so we can see exactly that the email has been sent and the content of those emails. The other benefit is Postmark will handle all the deliverability issues involved with sending emails from a custom domain that doesn't belong to us. They take care of all the icky DNS setup through a simple verification process. The downside is it's more expensive per send than our other options.

The second service is through AWS's simple email service (SES). It's cheaper per email than Postmark and gives us basic global metrics (like bounce and deliverability rates), but it doesn't log emails so we can't review the contents nor can we 100% guarantee the emails were ultimately delivered.

The final approach is through a local SMTP server sitting on our own internal network. It's the worst of all options because there's no out-of-the-box logging of anything. That's why it's a last resort option for us--and the odds of the first two services both failing are exceedingly rare.

Given our three use cases, I apply a unique order to the email services we try when sending email.

With in-app emails, we try AWS first, then try Postmark, then resort to our local SMTP. Why? Well, All in-app emails are sent from donedone.com, so there's no need to allow for custom domains. Secondly, in-app emails are sent very often -- we're saving money by using the AWS solution first.

With system emails, we try Postmark first, then AWS, then resort to our local SMTP. Why?

1) Try Postmark, then try AWS, then resort to our local SMTP.
2) 




https://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction

ONT: Email methods -- sendExternalUserEmail DRY up vs. split out to a postmark_ses_thing() because relationship to loggability is different than 'from' address thing.
