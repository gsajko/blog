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

![](/img/20210307145849.png)
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

![](/img/20210307151005.png)

### uploading data to collections
I was planning to use `tweepy` for uploading - but it doesn't have this functionality.
But twitter API is quite friendly for noobs, and there is no need for using a dedicated library.



## Reviewing proccess
Regardless of the methods of displaying tweets, I think it is crucial to have a way of looking at the results.

It allowed me to think about new features, new ways of filtering out tweets, small fixes etc. in an iterative matter.

The downside - this method relies on *eyeballing*. This could lead to silent errors. One of those examples I can think of is removing tweets that could be important for me, without me noticing. 

But as my main goal is improving signal-noise ratio, I'm willing to "loss" those tweets, if this means that my feed would be low on noise.

In [precision and recall](https://en.wikipedia.org/wiki/Precision_and_recall) terms, I consider this a problem good for "recall" metric.

Some of the cases:
### Deleted tweets
While writing a function that uploads tweets to a collection I noticed that sometimes upload fails.
After inspection there were two reasons:
- tweet is from a protected account
- tweet is deleted.

Almost 20% of tweets were deleted! - I guess December 2020 was a time when people regretted posting things.
It changed over time - right now it's less than 10% deleted tweets. It seems high, but I don't think it's surprising 

- I often delete tweets, within 10 minutes of posting (noticed spelling error or something like that; retweet something, then cancel retweet and do a quote tweet)
- there are accounts, that have automatic purge script setup, and they delete tweets older than a week or month.


This is how output of running looks like
```bash
972 tweets in a batch
Adding 30 tweets to collection custom-1351555076024893440
no_errors    29
not_found     1
Name: err_reason, dtype: int64
```
### Shortened URLs
I noticed, that despite being on the list of domains to be filtered out, some websites where still appearing in my custom feed.
Upon investication I found the culprit: shotrned url. 

Here are examples:
```py
[
'1843m.ag','4NN.cx','53eig.ht','808ne.ws','abc30.tv','abc7.la','abc7ne.ws',
'apne.ws','b-gat.es','bbc.in','bloom.bg','bos.gl','cbsn.ws','cityjourn.al',
'cnb.cx','cnet.co','cnn.it','cntrvlr.co','cos.lv','dailym.ai','econ.st',
'engt.co','hill.cm','ja.ma','jtim.es','mol.im','mtr.cool','natgeo.com',
'nyti.ms' ,'nzzl.us','on.mktw.net','on.theatln.tc','p4k.in','pewrsr.ch',
'propub.li','rdcu.be','reut.rs','rol.st','sabahdai.ly','sc.mp','slate.trib.al',
'str.sg','tcrn.ch','tdrt.io','thr.cm','ti.me','tmz.me','to.pbs.org','trib.al',
'vult.re','wapo.st','washex.am','wdrb.news','wired.trib.al','yhoo.it','zd.net'
]

```
Getting them was quite time consuming. I had to 
- iterate over all links in my data (15k url), 
- *expand* URL using `requests_html` and `response = HTMLSession().get(url)` - this can take about 1 second to process
  - I'm  using `domain = urlparse(url).netloc` when generating custom feed, but it doesn't handles shortened urls. 
- there were other options for doing that but all were time consuming. I decided on `requests_html`, because it provided easy method of getting the title of webpage. This could be very useful later for building models - I could `concat` tweet text with webpage title to provide more context for model.

As results, I have `.json` file with 15k link and titles. Each item looks like this:

```py
  {
      "url": "https://nyti.ms/3c4yrSX",
      "title": "Could a Small Test Screen People for Covid-19? - The New York Times",
      "short_url": "nyti.ms",
      "long_url": "nytimes.com",
      "is_news": true
  }
```
`short_url` - domain extracted using `urlparse(url).netloc` from shortened url directly
`long_url` - domain extracted from url expanded by `HTMLSession().get(url)`

This allowed me to quickly idwntify additional urls I should add to my `news_domains.txt` file.

But there were cases, when shortened url *sometimes* led to news site, and sometime not. I decided to have a treshold of 50% - if shortened url led to news site 50% of times, it should be counted as news domain.

Here are url that were *spared*:

![](/img/20210314124305.png)

### Labeling training data

## NEXT:


![[attn dataset 1]]
![[attn feed creation]]
[[blog process]]

### [[dealing with short urls]]

https://github.com/gsajko/tweetfeed/blob/master/notebooks/create_news_domains_list.ipynb