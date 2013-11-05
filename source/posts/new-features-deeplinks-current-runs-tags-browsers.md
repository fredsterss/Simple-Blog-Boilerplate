---
title: New Features Current runs, tags autocomplete, run deeplinks and consistent browser rendering
date: 2013-10-30
author: fred
category: news
tags:
  - features
---

From now on we're going to be much more visible about the work we're doing inside Rainforest. Over the past year we've been grinding away building an accurate and fast system for getting test results, and we've been so heads down that we have been pretty bad at letting the world know what we're working on.

No more! From now on we'll be posting weekly with everything that we deployed in the past week. This is the first such post. Without further adieu:

## New: Current Runs Displayed

![The current runs interface](http://f.cl.ly/items/1Z1X2q3c2W1N401P2D2J/Screen%20Shot%202013-10-30%20at%2015.50.28.png)

You guys really wanted this. In fact, *we* really wanted this! So we built it. Now whenever you run a test, one of these progress bars pops up in the bottom right of the dashboard, and auto-updates until the run is complete. It gives you a real-time view of how your run is progressing.

You can also abort the run by clicking the 'x'. We'll be improving this in upcoming releases by showing you what is passing and failing as it happens, and giving more insight into exactly what the backend is doing as the run progresses.

## New: Tags Autocomplete

![Autocompleteing Tags](http://f.cl.ly/items/2E0F301j2S3t033A2h1T/Screen%20Shot%202013-10-30%20at%2016.54.12.png)

Since we added a powerful filtering mechanism to Rainforest, Tags have been the best way to organize your tests. It has been hard to use tags consistently however because they have not autocompleted. We fixed that in this release. Tags now autocomplete and are searcheable.

## New: Run Deeplinks

![Run deeplinks](http://f.cl.ly/items/0T1V0Z3w1C3u33003S3r/Screen%20Shot%202013-10-30%20at%2017.01.46.png)

It's now possible to deeplink to test results from a specific run. You get these links in emails from us, and whenever it makes sense to show a specific run within Rainforest, for instance on the test history page:

![Run deeplinked from History](http://f.cl.ly/items/3R0w0f3D2E1X2P1b171N/screen%20copy.png)

We are trying to keep the references to runs within Rainforest to a minimum, because extensive user testing has shown us that the 'run' concept is quite confusing to non-technical customers. So this is a first step towards trying to find the right balance of power and simplicity.

## New: More Consistent Browser Rendering

![Consistent browser rendering](http://f.cl.ly/items/473R1X2K18283k26200F/Screen%20Shot%202013-10-30%20at%2017.24.08.png)

The small details make all the difference! Results are now shown for all possible browsers, with the browsers that were not tested shown as light grey. This makes it much easier to tell your cross-browser coverage at a glance.