---
title: "Part 3: Displaying Tweets"
date: 2021-02-26T18:08:01+01:00
draft: false
_build:
  list: true
  render: true
  publishResources: true
tags: []
tags: ["twitter", "custom_feed", "projects"]
description: "Project journal about making custom twitter feed"
toc: true
---


## Displaying and reviewing tweets.
I query tweets, I filter them out. And how to "serve" them?
Best solution would be [this](/projects/custom-twitter-feed-part1/#visualize-an-ideal-solution)
![twitter mockup 2](/img/20210125113932.png)

In theory this could happen, as Jack Dorsey seem to be "for it".
{{< tweet 1204766082885537792 >}}
But I had to look at other options in the meantime.
## Streamlit
Streamlit look like really great solution.
I could grab bigger batch, let's say 1000 of tweets, put some filters on the left sidebar, and user could decide in "real-time" what content he is interested in (and in what his not).
Or how many tweets to display:
![[Pasted image 20210307145849.png]]
I builded a prototype early.

### Why not streamlit?
#### fixed hight
You embeded-tweet-element needs to have a fixed height. 
What?
It looks like this:
```py
components.html(et, height=600, scrolling=True)
```
So there is an issue with this - tweets varies in content and lenght. If you have a media (image), or replying to someone, this mean you will have "tall" element.
And you would have to have addtional scrolling.

At the same time, if you post short, one sentence tweet, there will be big gap.

#### hard to provide feedback
You can't interact with tweets - if you want to reply or like, you need to open new window in browser.
It's a little annoying, but I can live with it.

But what if I want to label tweet as unrelevant? or put some user on "muted" list?

There is too much of a friction.

## Tweetdeck


### setting up Tweetdeck

Tweetdeck has fixed width for columns, so for better UI I use stylish plugin for firefox:

https://userstyles.org/styles/70450/tweetdeck-wide-columns?utm_campaign=stylish_stylepage

Changed code to suit me better:
```css
.column{width:450px !important}
```

In Tweetdeck I have 2 most important collections:
- `custom_feed` - where I upload tweets after filtering them
- `not_relevant` - so I can quickly drag tweets from one column to other, to mark them as not relevant.
![[Pasted image 20210307151005.png]]

### uploading data to collections
I was planning to use `tweepy` for uploading - but it doesn't have this functionality.
But twitter API is quite friendly for noobs, and there is no need for using a dedicated library.





## Reviewing proccess

While writing a function that uploads tweets to a collection I noticed that sometimes upload fails.
After inspection there were two reasons:
- tweet is from a protected account
- tweet is deleted.

Almost 20% of tweets were deleted! - I guess December 2020 was a time when people regretted posting things.
It changed over time - right now it's less than 10% deleted tweets. It seems high, but I don't think it's surprising 

- I often delete tweets, within 10 minutes of posting (noticed spelling error or something like that; retweet something, then cancel retweet and do a quote tweet)
- there are accounts, that have automatic purge script setup, and they delete tweets older than a week or month.


This is how output of running 
```bash
972 tweets in a batch
Adding 30 tweets to collection custom-1351555076024893440
no_errors    29
not_found     1
Name: err_reason, dtype: int64
```

## NEXT:
### Labeling training data

![[attn dataset 1]]
![[attn feed creation]]
[[blog process]]

### [[dealing with short urls]]

https://github.com/gsajko/tweetfeed/blob/master/notebooks/create_news_domains_list.ipynb