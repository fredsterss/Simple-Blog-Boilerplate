---
author: russ
title: A rough guide to keeping your website up through catastrophic events (aka. hosting in a single zone)
hackernews: http://news.ycombinator.com/item?id=4182271
categories: [guides, ops]
date: 2012-06-30
---

So, you're hosted in a single zone, and if you're in US-East presumably [went down last night](http://news.ycombinator.com/item?id=4180339). Stop it. AWS has seven regions across the world, three of which are in the US. Each region is split in up to five availability zones. You need to use more than just multiple zones if you want to stay up during one of these outages. Cross region is the answer.

The following is a rough guide to making your website available regardless of 'catastrophic' events.

## Static sites

Simple static sites and assets that don't change often have an easy option: just use CloudFront. You can do this by setting up a new CloudFront distribution. There are two common ways to use CloudFront for 'download' content - the first is S3 backed. This is great if your static content is already on S3.

Assuming so, use the [AWS Console](https://console.aws.amazon.com/), click CloudFront, click create distribution, select download, and select your bucket from the dropdown list. If you've not set your [http caching headers](http://www.mnot.net/cache_docs/#CACHE-CONTROL) then you should set the Object Caching to Custom and enter a TTL - aka how long to cache things for.

The second and less oft-used way is to use CloudFront as a forward-proxy. [Custom origins](http://aws.typepad.com/aws/2010/11/amazon-cloudfront-support-for-custom-origins.html) have been supported for some time now. It's pretty much the same procedure are above, but you point it at your webserver.

The process for this is more complex as you'll need to alter your webserver config, DNS and setup the distribution.

1. Setup a subdomain called origin.yourdomain.com,
2. Alter your webserver to serve your website from origin.yourdomain.com (you can also leave it serving from the old domain)
3. Set your CloudFront distribution's origin domain name to origin.yourdomain.com
4. As you are probably not setting your cache headers, you should set your object caching to custom and enter a [TTL](http://en.wikipedia.org/wiki/Time_to_live).
5. If you use [query strings](http://en.wikipedia.org/wiki/Query_string), make sure to enable them.
6. Check origin.yourdomain.com works
7. Add your old domain name(s) in the the Alternate Domain Names(CNAMEs).
8. Set a default root object - this is the default file to be served.
9. Remove the A record for your domain and add a CNAME to your distribution

Your website will now be significantly faster.

## Dynamic sites

For most people reading this, those of you serving dynamic content, you'll need to do something more complex. Migrating to a multi-region setup isn't the simplest so I'll outline it - the major parts are:

1. DNS
2. Auto Scaling
3. Automated configuration & Deployment / AMIs
4. Using a database or store which copes with network partitions in a sane way and coding your application to take advantage of this

## 1. DNS

You want to move to [Route53](http://aws.amazon.com/route53/).

Route53 is AWS's DNS offering. It's awesome. It uses anycast. It has a swanky API. There are libraries for [most](http://aws.amazon.com/sdkforphp/) [commonly](http://aws.amazon.com/sdkforruby/) [used](http://aws.amazon.com/sdkfornet/) [langauges](http://aws.amazon.com/sdkforjava/) and [more on github](https://github.com/search?q=route53&repo=&langOverride=&start_value=1&type=Repositories&language=). The most important feature for us is going to be LBR, or Latency Based Routing. This allows you to have DNS records which change depending on the latency in a sepcific AWS region. Think of it as a CDN for DNS: it serves the lowest latency record for each user.

Side note: if you're not already, you should be using DNS for internal communication. Machines fail, using DNS means you can easy replace them without having to redeploy configuration. Route53 rocks at this due to its API. This is a separate discussion though.

## 2. Auto Scaling

So, you want to be able to cope with a fixed (lol, jk - I mean you've guessed your capacity) amount of traffic, but without spending too much monies. You're engineering for minimal downtime but also want to minimise cost. I.e. you don't want to run two or more identical copies of your environment (If you do, you are either over provisioned 98% of the time by 2x or you've engineered something that will get overloaded for the 2%, which makes it all rather pointless).

The solution is Auto Scaling. Auto Scaling lets you scale the number of machines up or down based on metrics of your choosing. These can be application-specific or the standard metrics from Cloud Watch.

__tl:dr; When a region fails, traffic will mount up on the other one(s) and you'll need to scale up. Auto Scaling will do this.__

However, it relies on you doing #3.

## 3. Automated Configuration & Deployment and or AMIs

Automation is key for Auto Scaling, otherwise new machines start unconfigured. The easiest and fastest way to make your app work nicely with Auto Scaling is to use AMIs. However, this has one major issue: deployment. Releasing new code means rebuilding an AMI. If your release cycle is quick and your automation bad, this might suck more time than it's worth.

Using something like Puppet, Chef or CFEngine to manage configuration of your newly started machines is good - the main drawback over baking AMIs is speed - It can take up to 40 minutes to go from boot to usable. This might be too late if your application needs to deal with 100% extra traffic all of a sudden.

Automated deployment is essential if you're not baking AMIs.

## 4. The Crux: Your database

I'm probably biased, but my current database of choice is [MongoDB](http://www.mongodb.org/), so I'll assume it's yours too. If you're using MySQL or PostgreSQL I'll do a separate article in the future.

If you're using a single instance of Mongo, shame on you - you should migrate to at least two full machines + arbiter in a replica set. [Replica sets](http://www.mongodb.org/display/DOCS/Replica+Sets) allow for your app to keep running almost seemlessly to your application should your primary machine fail. They've also got pretty neat ways of going read only if nodes can't see the [majority of the set](http://www.mongodb.org/display/DOCS/Replica+Sets+-+Voting).

Minimum sane setup:

Primary in one Region, Secondary in another and an Arbiter in a third region. This will allow for your application to keep writing if either of the primary or secondary regions' fail.

A better setup:

Replace the Arbiter with a full replica set member. This will allow for your application to keep writing if any one region fails. If you have a sharded setup, the key is to make sure your shards are distributed as well as config servers. Should you be super-paranoid, I'd suggest using a mix of EBS backed and Instance Storage, this will help when EBS [goes](http://storagemojo.com/2011/04/29/amazons-ebs-outage/) [south](https://forums.aws.amazon.com/thread.jspa?threadID=46277), however it can be more of a pain to setup.

One downside to all this is that you will have to pay for regional data transfer (but luckily [not twice](http://aws.amazon.com/ec2/faqs/#If_I_transfer_data_between_availability_zones_using_public_IP_addresses_will_I_be_charged_twice_for_Regional_Data_Transfer_once_because_its_across_zones_and_a_second_time_because_Im_using_public_IP_addresses) and sort out your security (AWS doesn't currently support cross region Secuirty Groups).

In conclusion, you probably used a single zone because it's easy (hey - so do we for now!). There will come a point where the pain of getting shouted at by your boss, client or customers outweighs learning how to get your app setup properly yourself.

You can get your app on AWS to stay up even with a region wide failure. Why don't you?