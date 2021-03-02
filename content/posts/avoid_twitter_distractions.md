---
title: "How to avoid distractions while using Twitter"
date: 2021-02-01 18:24:01+02:00
draft: False
_build:
  list: true
  render: true
  publishResources: true
tags: ["twitter", "distraction", "focus"]
description: "How to get the most information out of Twitter while avoiding distractions"
toc: true
---

**TLDR**: use [uBlock Origin](https://en.wikipedia.org/wiki/UBlock_Origin)


## block stuff with uBlock Origin
uBlock Origin is a browser plugin, which I use mainly as an adblocker.
But you can also use its "picker mode" to block custom elements from a website. 

I used it for example to block out "Trends for you"

![trends for you](/img/20210108131748.png)

But it was not working correctly every time.

I tried this:
https://schleiss.io/fixing-twitter-design-with-extension/
but it didn't work.

So I found an even better solution:

Go to the uBlockOrigin dashboard, you should see something like this:

![dashboard](/img/20210201182747.png)

This is what you want to paste in there (you can change the date)

```
! 2021-01-08 https://twitter.com
twitter.com/i/api/2/timeline/home.json
twitter.com/i/api/2/guide.json
twitter.com/i/api/1.1/users/recommendations.json
```

It will prevent feed from loading (`home.json`), will block "people to follow" (`recommendations.json`) and trends(`guide.json`)

Then you can use uBlockOrigin to block out "Try again" parts for a cleaner look.

![s](/img/20210108160521.png)

This is the final effect:

![](/img/20210201180348.png)

So ... how to view tweets?
## Lists

Lists! I have a tonne of them.

![](/img/20210201181049.png)

I group people into lists and use them as needed.
You can even use lists created by other people. 
If I look for great curated lists, [@michael_nielsen](https://mobile.twitter.com/michael_nielsen) lists are my go-to.

https://mobile.twitter.com/michael_nielsen/lists

Instead of using "Home", I'm using one list, here called *rev_list*, which consist of 50-60 people. People whose tweets I don't want to miss. But I think about 20-30 of them are active, so there isn't a lot of tweets.

Visiting it once per day, it takes about 15 min to scroll through not-seen tweets.

If you MUST use "Home", just disable uBlockOrigin.

I found, that using "Home" rarely gives good results! Twitter algo really flexes its recommendation muscles 💪.

**BONUS** - [How to research stuff using Twitter](../use_twitter_for_research)