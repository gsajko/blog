---
title: "How to research stuff using Twitter"
date: 2020-02-01 19:24:01+02:00
draft: False
_build:
  list: true
  render: true
  publishResources: true
tags: ["twitter", "researching", "learning"]
description: "How to get useful informations out of Twitter"
toc: false
---

## Key to this method - follow lots of opinionated people. 

I follow many people - more than 600. There are many ML experts. So if I'm researching what methods to use, what algorithms, or just want to learn about a subject I'll use Twitter first for that.


## Using search
Let's say I want to learn how to do "topic modeling" on a dataset made of tweets.
I just use search, remembering to check "**People you follow**"

![search](/img/20210201183635.png)

I especially liked this tweet:
{{< tweet 1153485963794046981 >}}

Without any prior knowledge, just from that one tweet by [@pmbaumgartner](https://twitter.com/pmbaumgartner/), you can learn that there is a traditional topic modeling method called "LDA", and it doesn't provide good results for tweets. "BERT embeddings/Universal Sentence Encoder → UMAP → HDBSCAN" seem to work better.

But let's look at the whole discussion.

## Treeverse
I go to the "main" tweet and use a browser plugin called [Treeverse](https://treeverse.app/)

This is the tweet that started the whole discussion.
{{< tweet 1153456432122978304 >}}
It gives me a good overview - Twitter sometimes hides some tweets, so I prefer to use Treeverse.
![Treeverse](/img/20210201184238.png)

One other tweet contained tricks on how to improve the performance of modeling:

{{< tweet 1153464471857377288 >}}

There was also a response by this guy:

{{< tweet 1153488195432132614 >}}

Remember "*BERT embeddings/Universal Sentence Encoder → UMAP → HDBSCAN*" from the previous tweet?

![Leland](/img/20210201184524.png)

Leland happens to be the researcher behind UMAP and HDBSCAN. 

Instant "follow" for me.


Ok, so what about LDA? What do other people think about it?
I repeated the search, remembering to check "People you follow".

This time I needed to dig a little deeper, focused on this tweet:

{{< tweet 1133806246182604800 >}}

The **tree** got way bigger (Rachael has 22k followers)

![](/img/20210201185406.png)

But if you zoom in:

![](/img/20210201185506.png)

We see familiar faces ([@pmbaumgartner](https://twitter.com/pmbaumgartner/), [leland_mcinnes](https://twitter.com/leland_mcinnes)) 

Leland even provides code examples for his approach.

{{< tweet 1133908055689781249 >}}

https://gist.github.com/lmcinnes/fbb63592b3225678390f08e50eda2b61

Twitter is good for finding different approaches/methods, way better than any other forum-like solution.

But the key is following correct people. 

And **do bias to the side of more opinionated folks**.