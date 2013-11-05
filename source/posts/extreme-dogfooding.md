---
author: simon
title: Extreme dogfooding
categories: [guides, workflow]
date: 2013-11-04
---

We believe dogfooding our own product has been crucial in helping us decide when to open up Rainforest to public beta. Confident that our product is fast and accurate enough to solve real-world QA needs, this has been a long journey and many things helped us get to this point.

This blog post will explain how critical dogfooding was for the development of Rainforest and how we used it to decide when to launch our product.

*Rainforest is a super easy way to do QA for web apps. I am the first engineer hired at Rainforest.*

## How we Dogfood

We are great believers in [Continuous Integration](http://en.wikipedia.org/wiki/Continuous_integration) and deployment, so one of the first things that was set up was a CI server. This automatically runs our unit tests ([rspec](http://rspec.info/) / [jasmine](http://pivotal.github.io/jasmine/)) and deploys to our staging, QA and production environments.

On top of our comprehensive unit test suite we have (of course!) a Rainforest test suite. In the early days that suite was both run manually whenever we needed, as well as 4 times a day by a [cronjob](http://en.wikipedia.org/wiki/Cron). Back then our Rainforest suite would almost always fail, because the product was so full of bugs.

After several months of hard work - adding more filtering / training / checking (more in another post) - Rainforest was in a 'pretty good' state. By that I mean [Rainforest](https://www.rainforestqa.com/) was in a state where we could clearly see its potential in terms of speed and quality. However, it still wasn't good enough to expect people to use it, much less pay for it.

It became clear then that we needed our first serious user, and that this user could and should only be us. By serious user I mean someone who uses Rainforest as we intended: full CI integration where Rainforest is a first class citizen, equal in standing to rspec and jasmine. **This made it impossible for us to deploy without a passing test suite!**

Talk about disruptive.

At some point I started referring to this process as 'extreme' dogfooding, and it stuck.

## Why we did it

You might be thinking that this all seems like a major PITA. You'd be right. But the upside of doing this has vastly outweighed the downside. 

We knew that if we didn't have confidence in our product it was unfair to expect our customers to. It is far better for us to deal with these issues and feel the pain directly than to let our beta users report them to us. The obvious reason is that we have the power to fix those issues and our customers do not. This creates a very tight feedback loop, where we notice problems immediately because we're such heavy users of the product. 

*It was also the most important launch indicator*. After several months of finding bugs and being annoyed at our deploy process, we got to the point where this process was no longer painful - in fact, Rainforest was finding real bugs in our application, before we could deploy it to production. Because we were so close to our users' experience of the product we knew when it was worth paying for.

## Things we found through dogfooding

Our product has totally changed since we started this process, but here are four of the most important features that resulted out of this experiment. It would have taken us much longer to find these and been much more painful from our customers' perspective if we hadn't been doing extreme dogfooding.

### Ease of integration is super important
The first thing that came out of this experiment was that integrating Rainforest and a CI server was pretty painful. To make it easier, we wrote a [command line tool](https://github.com/rainforestapp/rainforest-cli) to run our browser tests.

### We didn't want to run everything
We then realized that some of our tests should not be run as they are not actually tests, but [sub-tests or components of different tests](http://docs.rainforestqa.com/pages/example-test-suite.html#test_steps). Tagging already existed, but critically there was no way of specifying this when starting a run. Fail. We added this to our API, and now it's easy to run a subset of our tests.

### Pushing too fast can be painful
Sometimes we would push to our staging branch faster than we could run the Rainforest suite. This would trigger two simultaneous Rainforest runs. This was wasteful and caused various race conditions in our suite.

To solve this, we added an extra option to our API that aborts other in-progress runs whenever you trigger a new one.

### We needed to support multiple sites

This one is a little embarrassing; Our app is spread across two subdomains (www.  and app.). We wanted to add an integration test that convered signing up from our marketing site and then subscribe to a pricing plan from our main app. At that time, we realized that our UI did not allow up to test multiple subdomains or domains. Oops. That was promptly fixed.

### The little things
In addition to the big things, there were tons of little bugs that affected our speed and accuracy that we were forced to fix. These details are easy to overlook while building a product, but the details matter; the last 20% of execution makes up 80% of the experience. 

If our accuracy was not 100%, we could not deploy, because a failure in Rainforest kills our build.

## Conclusion

Dogfooding has been both a great way of building a product and given us confidence that Rainforest is ready to solve real problems, and hence worth paying for.

Building a product that solves a problem for yourself is always easier that one that solves someone else's problem; you understand it, you're close to it and it affects you when it's broken.

This tight feedback loop enabled by this extreme dogfooding means you are forced to quickly fix the things that are broken.

Maybe your team is not lucky enough to be building dev tools they need to use daily, but you should still find a way of having the product you build solve a problem that you have.