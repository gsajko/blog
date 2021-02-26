---
title: "Part3"
date: 2021-02-26T18:08:01+01:00
draft: true
_build:
  list: false
  render: true
  publishResources: true
tags: []
tags: ["twitter", "custom_feed", "projects"]
description: "Project journal about making custom twitter feed"
toc: true
---


## Displaying and reviewing tweets.
### uploading data to collections
two collections - `not_relevant`, and `custom_feed`.
`custom_feed` is where tweets are heading. 
`not_relevant` - for an easy way to label tweets.

### 
While writing a function that uploads tweets to a collection I noticed that sometimes upload fails.
After inspection there were two reasons:
- tweet is from a protected account
- tweet is deleted.

Almost 20% of tweets were deleted! - I guess December 2020 was a time when people regretted posting things.
It changed over time - right now it's less than 10% deleted tweets. It seems high, but I don't think it's surprising 

- I often delete tweets, within 10 minutes of posting (noticed spelling error or something like that; retweet something, then cancel retweet and do a quote tweet)
- there are accounts, that have automatic purge script setup, and they delete tweets older than a week or month.



## NEXT:
### Labeling training data

![[attn dataset 1]]
![[attn feed creation]]
[[blog process]]

### [[dealing with short urls]]

https://github.com/gsajko/tweetfeed/blob/master/notebooks/create_news_domains_list.ipynb