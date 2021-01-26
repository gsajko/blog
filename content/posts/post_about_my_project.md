---
title: "Begining of a custom-twitter-feed project"
date: 2021-01-26T10:15:57+01:00
draft: True
tags: ["twitter", "custom_feed", "projects"]
description: "Project journal about making custom twitter feed using Machine Learning"
toc: true
---


## Intro
This is an introductory post about the project I'm making.
I'm in process of reading [Machine Learning Bookcamp Book](https://www.manning.com/books/machine-learning-bookcamp) by [Alexey Grigorev](https://twitter.com/Al_Grigor). The main "meta" concept behind this book is the idea, that the best way to learn Machine Learning is by doing projects.

In parallel, I follow [Applied ML](https://madewithml.com/#applied-ml) course by [Goku Mohandas](https://twitter.com/GokuMohandas). 

I will try to incorporate things I learn from both of those sources while doing this project.

For this post I'll be loosely using framework proposed [here](https://madewithml.com/courses/applied-ml/solution/)

## The Problem: Attention is scarce and Twitter feed sucks.

### Attention is scarce 
Common thinking about attention is, that we use our attention to consume information.

I like how Herbert A. Simon in [Designing organizations for an information-rich world](https://digitalcollections.library.cmu.edu/awweb/awarchive?type=file&item=33748) reverses this:

We're not consuming information, **information is consuming our attention**.

>Hence a wealth of information creates a poverty of attention and a need to allocate that attention efficiently among the overabundance of information sources that might consume it.

That means, that **attention is scarce**. We should be really mindful about where we allocate our attention. At the sane time Internet media are overflowing with information.

### How to deal with this?
#### Reduce information overload
We can reduce information overload, by filtering out information that is not  relevant for us.

#### Focus on what matters most.
This can be different from person to person. But this need to comes from that personal deep values. 

I like Cal Newport's "The Minimalist Technology Screen" approach (from his book [Digital Minimalism](https://www.calnewport.com/books/digital-minimalism/)) 

>To allow an optional technology back into your life at the end of the digital declutter, it must:
>1. Serve something you deeply value (offering some benefit is not enough).
>2. Be the best way to use technology to serve this value (if it’s not, replace it with something better).
>3. Have a role in your life that is constrained with a standard operating procedure that specifies when and how you use it.


### Twitter feed sucks
I use Twitter a lot, so based on the premise that "attention is scarce", I don't find it useful.

I have to admit, that twitter newsfeed is good at recommencing me tweets (and putting them high on my newsfeed), that are interesting for me. This is even more visible, when I don't visit "Home" tab for a long time. If you refresh you newsfeed every 5 minutes it becomes worse immidetly. 
But at the same time, it floods me with unrelated stuff trying to drive my engagement even more - suggest me "topics to follow", and also recommends tweets related to outrage-of-the-day, suggest viral tweets. It just adds to the noise.
Muted words doesn't work for me as intended.
Twitter is good as a entertainment. Not as a source of information. 


##  Personal curiosity
Aside of "solving" this problem, there are several other reasons, why I'm interested in doing this.

### personal computation
I'm fascinated about ideas around "personal computation" where users control they own data, and algorithms that they use everyday. With this project I wanted to learn more about this, about challenges that are to be faced when trying to serve models locally, and how to train them with small about of data.

### learning about recsys
At the same time, I want to learn more about using RecSys in "real life" settings, and learn how to improve them with time.

### software engineering
This project is also my first attempt to have project existing outside of Jupyter Notebooks - with linting, Unit Tests, Makefile etc 

As show here 👉 [How to Set Up a Python Project For Automation and Collaboration](https://eugeneyan.com/writing/setting-up-python-project-for-automation-and-collaboration/)

## Solution
Based [Applied ML](https://madewithml.com/#applied-ml) course - we need to first visualize solution, understand the problem, and design a baseline.

### Visualize an **ideal solution**
#### Custom twitter feed
You get your custom newsfeed. 

It filters out irrelevant stuff based on:
- is it news?
- is 
- user preferences - mute words, mute accounts

It puts high on the list tweets that
- have low noise-to-signal ratio
- are evergreen


And how to "serve" them?

Ideal solution would be if twitter would enable API for loading custom tweets in there "Home" tab.
this is my mockup how would that look:

You click on the ✨ icon
![twitter mockup 1](/img/20210125113745.png)

and can choose newsfeed generated by 3th party like you can choose between "Latest" and "Home".
![twitter mockup 2](/img/20210125113932.png)


### Understand
#### Why twitter solution is bad?

Twitter has other incentives them me:Current twitter solution maximize user engagement, not maximizing signal to noise.

Filter out things that decay quickly with time.
#### relation between relevant and recent

Chart used in this [tweet](https://mobile.twitter.com/GruPiotr/status/1330507931549306882) below is great at showcasing how the relevance of information can differ, and it relation to time since published.

{{< tweet 1330507931549306882 >}}

![relevence chart](/img/20210125121217.png)

I want to get rid of all  information represented by "blue" curves, and most represented by "green" curves.


### Design
#### baseline:

- using cron jobs, grab tweets from `home` endpoint using `twitter-to-sqlite`
- using rule-base approach filter out tweets
- Classify remaining tweets based on engagement - (`tweet is engaging`, `tweet is not engaging` )
	- ❗️ this requires for me to be more conscious about what tweets are interact with ❗️ 
		- I need to show "algorithm" that I prefer particular type of tweets.
- send them to collection inside of [Tweetdeck](https://tweetdeck.twitter.com/).
	- I have though about different approaches how to serve those tweets.
		- serving web app with Flask and tweets embedded.
		- serving using streamlit.
- Using Tweetdeck is the best, because it allows me to interact with tweets with ease - I can like and retweet from there. 
- Additionally, I can create more than one collection - I can easily drag/copy tweets from one collection to another - this enables me for easy labeling of tweets - just drag tweet to `not_relevant` collection.
