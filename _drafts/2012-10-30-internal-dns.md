---

layout: post
title: "internal dns"
date: 2012-10-30 16:18:42
category:
tags:
  -

---

# Easy application configuration with internal DNS

Many people have different application configurations for their production, staging, test and development environments: we don't. Using internal DNS and search paths allows us to separate the configuration from the infrastructure.

This has two advantages;

- No change of config between environments - this reduces those odd errors when pushing that many of you have experienced
- When replacing servers there is no need to alter the configuration and re-deploy the application

Implementing internal DNS is easy, especially if you're using Amazon AWS. Amazon offer a service called Route53, which allows for programatic configuration and setup of DNS servers. The only caveat is that all your records are publicly resolvable if known - this isn't likely to be an issue for most people.

Thinking about implementing internal DNS will force you to think about you infrastructure layout in general; how you group your machines, how they are named, where they are located, etc.

How to group your machines in AWS could be a whole separate blog post, but the default option for most people is to group machines using security groups by role and environment.

Once you've decided this, then you can ded
