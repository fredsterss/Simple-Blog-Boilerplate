---
layout: post
author: russ
title: "Rainforest QA + Continous Integration = love"
date: 2013-08-29 12:57:04
category: 
tags:
  - ci
  - cli

---


I hate doing manual QA. Being a web developer it's part of my life, which is why Fred and I started working on [Rainforest QA](https://www.rainforestqa.com/). We've been practicing extreme-dogfooding (explaination in an upcoming post) - we've recently got bored of having to manually start or schedule new QA runs, so today we're announcing decent (alpha) continuous integration support.

# Setting up rainforest-cli with a Ruby project

Add it to your Gemfile;

```
gem "rainforest-cli", require: false
```

We use Circle CI (as they're really fast and we have lots of tests) - so for them you can just add;

```
bundle exec rainforest run --token YOUR_API_TOKEN --tag run-me --conflict abort --fg --fail-fast
```

to the commands section of your circle.yml file

```
deployment:
  staging:
    branch: develop
    commands:
      - bundle exec rainforest run --token YOUR_API_TOKEN --tag run-me --conflict abort --fg --fail-fast
```