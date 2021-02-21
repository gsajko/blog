---
title: "Part 2: Collecting and displaying data"
date: 2021-02-18T16:43:08+01:00
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
## Collecting training data
### collecting data
I collect twitter data using library made by @simon twitter-to-sql

cron jobs


## Displaying 
### uploading data to collections
two collections - `not_relevant`, and `custom_feed`.
`custom_feed` is were tweets are heading. 
`not_relevant` - for easy way to label tweet.

### 
While writing function that uploads tweets to collection I noticed that sometimes upload fails.
After inspection there were 2 reasons:
- tweet is from a protected account
- tweet is deleted.

Almost 20% tweets were deleted! - I guess December 2020 was time, when people regretted posting things.
It changed over time - right now it's about 10% deleted tweets. It seem high but I don't think it's surprising 

- I delete often, within 10 minutes of posting (noticed spelling error or something like that)
- there are accounts, that have automatic purge script setup, and they delete tweets older than week / month.



## Filtering out
### finding news
As a part of design, my goal was to remove news from my feed.
So first "filter" should remove tweets related to news.
After pondering for some time ("Should I do ML based news detection model?") I arrived at very simple solution:
- removing tweets containing links to news domains.

Luckily for me, there is such a dataset.
https://www.kaggle.com/uciml/news-aggregator-dataset
In that dataset we have column named `HOSTNAME`, so I could take unique values from that dataset 

This got me over 9000 unique news domain names.
`news_domain_list`



---
#### get domain
so let's move to tweets.

In order to find news in tweet urls I needed to clean them up.
- removed links to other tweets
- removed short links, like "bit.ly", "buff.ly".

Then I needed a function to find url in tweet (simple regex `re.findall(r"http\\S+", tweet)`)

to get domain I used  `urllib` library:

```python
def get_domain(url: str) -> str:
    """extracts domain from url, returns it"""
    domain = urlparse(url).netloc.replace("www.", "")
    dot_split = domain.split(".")
    if (len(dot_split) > 2) & (
        dot_split[-1] == "com"
    ):  # for links like "edition.cnn.com", but not like "site.co.nz"
        return ".".join(dot_split[1:])
    else:
        return domain
```

After reviewing some batches I noticed that some domains weren't detected properly, so there was a need for some splitting and joining - result were transformed to match the `news_domain_list`

#### is it news?
The result of this was a single pandas cell with a list.

I expanded pd.DataFrame, so each domain has it own column.
On each of those columns (max links in tweet was 6!) I used simple `.isin`
```python
    for col in columns:
        df[col] = df[col].isin(news_domains_list)
```

Then I summed it up in one column `contains_news` and made values `0` or `1`.



Trough uploading batches of tweets to collection and reviewing them, I make some changes to that list: removed some domains (youtube.com, linkedin.com), noticed that some of domains weren't in the dataset (for example: techmeme.com, buzzfeednews.com)

After another iteration some news still were getting through, and not some small news site, but sites like cnn.com.

I discovered that sometimes news were posted in short url format.

#### dealing with short urls


####





https://github.com/gsajko/tweetfeed/blob/master/notebooks/create_news_domains_list.ipynb





Oo
### most important client - myself
I'm doing this project 


## NEXT: Labeling training data


![[attn dataset 1]]
![[attn feed creation]]
[[blog process]]