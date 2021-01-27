---
title: "Custom twitter feed project - Part 1"
date: 2021-01-26T10:15:57+01:00
draft: False
_build:
    list: true
tags: ["twitter", "custom_feed", "projects"]
description: "Project journal about making custom twitter feed using Machine Learning"
toc: true
---


## Introduction
This is an introductory post about the project I'm making.
I'm in process of reading [Machine Learning Bookcamp Book](https://www.manning.com/books/machine-learning-bookcamp) by [Alexey Grigorev](https://twitter.com/Al_Grigor). The main "meta" concept behind this book is the idea, that the best way to learn Machine Learning is by _doing projects_.

In parallel, I follow [Applied ML](https://madewithml.com/#applied-ml) course by [Goku Mohandas](https://twitter.com/GokuMohandas). 

I will try to incorporate things I learn from both of those sources while doing this project.

For this post, I'll be loosely using the framework proposed in "Applied ML" course.

## The Problem: Attention is scarce and Twitter feed sucks.

### Attention
There are many ways to think about attention.
For example:

[Attention is consuming information](https://twitter.com/balajis/status/1339214847909609473)

[Attention is like a water](https://www.newyorker.com/magazine/2013/01/07/a-pickpockets-tale)


I like how Herbert A. Simon in [Designing organizations for an information-rich world](https://digitalcollections.library.cmu.edu/awweb/awarchive?type=file&item=33748) reverses "Attention is consuming information".

We're not consuming information, **information is consuming our attention**.

>Hence a wealth of information creates a poverty of attention and a need to allocate that attention efficiently among the overabundance of information sources that might consume it.

That means that **attention is scarce**. We should be mindful about how we allocate our attention. At the same time, Internet media are overflowing with information.

### How to deal with information overload?
#### Reduce by filtering
We can reduce information overload, by filtering out information that is not relevant to us.

#### Focus on what matters most.
This can be different from person to person. But this need to comes from that personal deep values.

I like Cal Newport's "The Minimalist Technology Screen" approach (from his book [Digital Minimalism](https://www.calnewport.com/books/digital-minimalism/)) 

>To allow an optional technology back into your life at the end of the digital declutter, it must:
>
>Serve something you deeply value (offering some benefit is not enough).
>
>Be the best way to use technology to serve this value (if it’s not, replace it with something better).
>
>Have a role in your life that is constrained with a standard operating procedure that specifies when and how you use it.


### Twitter feed sucks

I use Twitter a lot, so based on the premise that "attention is scarce", I don't find it useful.

I have to admit, that Twitter newsfeed is good at recommencing to me tweets (and putting them high on my newsfeed), that are interesting for me. This is even more visible when I don't visit "Home" tab for a long time. If you refresh your newsfeed every 5 minutes it becomes worse immediately. But at the same time, it floods me with unrelated stuff trying to drive my engagement even more - suggest me "topics to follow", and also recommends tweets related to outrage-of-the-day, suggest viral tweets. It just adds to the noise. Muted words don't work for me as intended. Twitter is good as entertainment. Not as a source of information.


## Other reasons for doing this project
Aside from "solving" this problem, there are several other reasons, why I'm interested in doing this.

### personal computation
I'm partial to ideas around "personal computation" where users control their own data and algorithms. With this project, I wanted to learn more about this, about challenges that arise when trying to serve models locally, and how to train them with a small about of data.

### learning about RecSys
At the same time, I want to learn more about using RecSys in "real life" settings and learn how to improve them with time.

### software engineering
This project is also my first attempt to have a project existing outside of Jupyter Notebooks - with linting, Unit Tests, Makefile, writing documentation, etc.

As show here 👉 [How to Set Up a Python Project For Automation and Collaboration](https://eugeneyan.com/writing/setting-up-python-project-for-automation-and-collaboration/)

## Solution
Based [Applied ML](https://madewithml.com/#applied-ml) course - we need to first **visualize** solution, **understand** how the problem is currently being solved, and **design** a baseline.

### Visualize an ideal solution
#### Custom twitter feed

You get your custom newsfeed. 

It works in a two prongs approach: filtering and ranking.

It **filters out** irrelevant stuff based on:
- is it news?
- user preferences - mute words, mute accounts

It **ranks tweets** and puts high on the list tweets that
- have a low noise-to-signal ratio
- are evergreen


And how to "serve" them?

An ideal solution would be if twitter would enable API for loading custom tweets in their "Home" tab.
this is my mockup of how would that look:

You click on the ✨ icon
![twitter mockup 1](/img/20210125113745.png)

and can choose newsfeed generated by 3rd party like you can choose between "Latest" and "Home".
![twitter mockup 2](/img/20210125113932.png)


### Understand
#### Why twitter solution is bad?

Twitter has other incentives them me: The current twitter solution maximizes user engagement, not maximizing signal to noise.

And I want to filter out things that decay quickly with time. 
#### the relation between relevant and recent

The chart used in this [tweet](https://mobile.twitter.com/GruPiotr/status/1330507931549306882) is great at showcasing how the relevance of information can differ, and its relation to time since published.

{{< tweet 1330507931549306882 >}}

![relevence chart](/img/20210125121217.png)

I want to get rid of all information represented by "blue" curves, and most represented by "green" curves.


### Design
#### baseline:


- getting data: using cron jobs, grab tweets from a `home` endpoint to SQL, using [twitter-to-sqlite](https://github.com/dogsheep/twitter-to-sqlite)
- using the rule-based approach to **filter out** tweets
- re-using this data, to **classify** other tweets, that didn't get filter out (`is tweet not-relevant`)
- send them to a collection inside of [Tweetdeck](https://tweetdeck.twitter.com/).
	- I have thought about different approaches how to serve those tweets.
		- serving web app with Flask and tweets embedded.
		- serving using streamlit.
    - Using Tweetdeck is the best, because it allows me to interact with tweets with ease - I can _like_ and _retweet_ from there. 
    - Additionally, I can create more than one collection - I can easily drag/copy tweets from one collection to another - this enables me for easy labeling of tweets - just drag tweet to `not_relevant` collection.

#### next:
- **classify** remaining tweets based on engagement - (`tweet is engaging`, `tweet is not engaging` )
	- ❗️ this requires me to be more conscious and mindful about which tweets I interact with ❗️ 
		- I need to show to "algorithm", that I prefer a particular type of tweet.
