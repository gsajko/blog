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
I collect twitter data using library made by Simon Willison - [twitter-to-sql](https://github.com/dogsheep/twitter-to-sqlite).
It's part of 
My first approach was to do it by myself: twitter API is very clear, so it shouldn't be too hard to just grab tweets with API and dump them into SQL? Right?
It wouldn't be a problem as a one-time-thing, as I usually had done in the past in jupyter notebooks. 
But if this should be more robust - with proper testing etc it wouldn't be a "custom twitter feed" project but "twitter API to SQL" project.

So I decided to use Simon's library.
It's awesome, I have it "git-cloned" in a folder, so from time to time I take a look at the code. 
It's a part of bigger project by Simon called [dogsheep](https://dogsheep.github.io/). "Tools for personal analytics using SQLite and Datasette"
<details><summary>CLICK for more Datasette and Dogsheep</summary>
	<p>
		<iframe width="560" height="315" src="https://www.youtube.com/embed/l1EFThsAFgs" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
		<br>
		 detailed notes by Simon: 
		<a href = "https://simonwillison.net/2020/Nov/14/personal-data-warehouses/">LINK</a>
	</p>
	</details>

### Setting up cron jobs
This was new to me. 

Here are some resource I found useful for setting up cron jobs on Ubuntu:
https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-ubuntu-1804
https://www.howtogeek.com/101288/how-to-schedule-tasks-on-linux-an-introduction-to-crontab-files/

small things I learned:

how to check if  cron is working?
`ps -ef | grep crond`

how to look at current crontab (without going into editor)?
`crontab -l`

```bash
gsajko   16198 16000  0 10:56 pts/2    00:00:00 grep --color=auto crond
```

I tried this one, but without success.
```bash
*/1 * * * * cd /home/gsajko/work/custom_twitter_feed run-one twitter-to-sqlite home-timeline home.db --since -a auth/auth.json
```
I spent way to much time debugging this, looked at many `log/syslog` files.

Until I cound [this gem](https://serverfault.com/questions/449651/why-is-my-crontab-not-working-and-how-can-i-troubleshoot-it):

>**The vast majority of "my cron script doesn't work" problems are caused by this restrictive path.**
>you can solve this in a couple of ways:
>1. Provide the full path to your command.
> `1 2 * * * /path/to/your/command`

So I quickly remade it (removed `cd ` part, added full paths for twitter-to-sqlite, to `home.db`, and to `auth.json` file)

```bash
2,7,12,17,22,27,32,37,42,47,52,57 * * * * run-one /home/gsajko/miniconda3/bin/twitter-to-sqlite home-timeline /home/gsajko/work/custom_twitter_feed/home.db -a /home/gsajko/work/custom_twitter_feed/auth/auth.json --since
```

Simon also shared his crontab for his personal data werehouse here:
https://gist.github.com/simonw/1299d61d17637d1145955ebc019ea3c4
I shamelessly copied his "time" settings. 

What is the `run-one`? It check is previous run was completed, before starting new one. Given possible timeouts on API side, it's great to have it in place.
https://blog.dustinkirkland.com/2011/02/introducing-run-one-and-run-this-one.html

### note about collecting data
I'm using `home-timeline` option:

[Get Tweet timelines](https://developer.twitter.com/en/docs/twitter-api/v1/tweets/timelines/api-reference/get-statuses-home_timeline)

> Up to 800 Tweets are obtainable on the home timeline. It is more volatile for users that follow many users or follow users who Tweet frequently.

If you have muted accounts, using this endpoint WON'T get their tweets. I some way, I'm giving some of the control to twitter when using this option.
But I wanted to have wide range of tweets, and other option would be to download ALL tweets by ALL users I follow.

This would baloon the database waay more then I would like too - this option would download 3200 tweets of each user. But it would download another 3200 with next use - until it would grab all possible tweets.

I did this as a test for a list of 60 users, and `db` was about 200mb in size.
Compared to `home-timeline` `db` grabing tweets from more than 600 users I follow - 160mb (about 2,5 months of cronjobs).

I was fun looking at tweets posted in 2008, though. People were using twitter in tottaly different way then today - there were mostly micro-blogs.


## Displaying and reviewing tweets.
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
It changed over time - right now it's less than 10% deleted tweets. It seem high but I don't think it's surprising 

- I delete often, within 10 minutes of posting (noticed spelling error or something like that; reweet something, then cancel reweet and do a quote tweet)
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

Trough uploading batches of tweets to collection and reviewing them, I make some changes to that list: removed some domains (youtube.com, linkedin.com), noticed that some of domains weren't in the dataset (for example: techmeme.com, buzzfeednews.com), so I added them to the list.

## NEXT:
### Labeling training data

![[attn dataset 1]]
![[attn feed creation]]
[[blog process]]

### [[dealing with short urls]]

https://github.com/gsajko/tweetfeed/blob/master/notebooks/create_news_domains_list.ipynb

### most important client - myself
I'm doing this project 

