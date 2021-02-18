---
title: "Part 2: Getting Data and Filtering Out"
date: 2021-02-18T16:43:08+01:00
draft: true
_build:
  list: false
  render: true
  publishResources: true
tags: []
tags: ["twitter", "custom_feed", "projects"]
description: "Project journal about making custom twitter feed using Machine Learning"
toc: true
---
## Filtering out
## Describe process so far.
### collecting data
cron jobs


## Displaying 
### uploading data to collections
two collections - `not_relevant`, and `custom_feed`.
`custom_feed` is were tweets are heading. 
`not_relevant` - for easy way to label tweet.

## Removing news
### filtering out
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
so let's move to tweets.

In order to find news in tweet I needed to clean them up.
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

there was a need for some splitting and joining - result were transformed to match the `news_domain_list`

The result of this was a single pandas cell with a list.
I expanded pd.DataFrame, so each domain has it own column.
On each of those columns (max links in tweet was 6!) I used simple `.isin`
```python
    for col in columns:
        df[col] = df[col].isin(news_domains_list)
```

Then I summed it up in one column `contains_news` and made values `0` or `1`.