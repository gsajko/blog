---
title: "Part 2: Collecting and displaying data"
date: 2021-02-18T16:43:08+01:00
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
## Collecting training data
### Collecting data
I collect Twitter data using a library made by Simon Willison - [twitter-to-sql](https://github.com/dogsheep/twitter-to-sqlite).

My first approach was to do it by myself: Twitter API is very clear, so it shouldn't be too hard to just grab tweets with API and dump them into SQL? Right?
It wouldn't be a problem as a one-time-thing, as I usually had done in the past in jupyter notebooks. 
But if I would want to make something more robust - with proper testing etc it wouldn't be a "custom twitter feed" project but a "Twitter API to SQL" project.

So I decided to use Simon's library.
It's awesome, I have it "git-cloned" in a folder, so from time to time I take a look at the code. 
It's a part of a bigger project by Simon called [dogsheep](https://dogsheep.github.io/). "Tools for personal analytics using SQLite and Datasette"

<br>
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
So what are cron jobs? [Wikipedia](https://en.wikipedia.org/wiki/Cron): "is a time-based job scheduler". 
It means, that you shedule when you want a script to run.
For me, the script was `twitter-to-sqlite` command downloading my Twitter timeline, and `time` was "every 5 minutes".
You put this information into a file called `crontab` and it executes tasks in the background.

Here are some resources I found useful for setting up cron jobs on Ubuntu:
- https://www.digitalocean.com/community/tutorials/how-to-use-cron-to-automate-tasks-ubuntu-1804
- https://www.howtogeek.com/101288/how-to-schedule-tasks-on-linux-an-introduction-to-crontab-files/



First I tried this command below, but without success.
```bash
*/1 * * * * cd /home/gsajko/work/custom_twitter_feed run-one twitter-to-sqlite home-timeline home.db --since -a auth/auth.json
```
I spent way too much time debugging this, looked at many `log/syslog` files.

Small things I have learned:

How to check if cron is working?
`ps -ef | grep crond`

How to look at the current crontab (without going into the editor)?
`crontab -l`

```bash
gsajko   16198 16000  0 10:56 pts/2    00:00:00 grep --color=auto crond
```

My script still didn't run.
But I found [this gem](https://serverfault.com/questions/449651/why-is-my-crontab-not-working-and-how-can-i-troubleshoot-it):

>**The vast majority of "my cron script doesn't work" problems are caused by this restrictive path.** 
> You can solve this in a couple of ways:
>
>1. Provide the full path to your command.
>
> `1 2 * * * /path/to/your/command`

So I quickly remade it (removed `cd` part, added full paths for `twitter-to-sqlite`, to `home.db`, and to `auth.json` file)

```bash
2,7,12,17,22,27,32,37,42,47,52,57 * * * * run-one /home/gsajko/miniconda3/bin/twitter-to-sqlite home-timeline /home/gsajko/work/custom_twitter_feed/home.db -a /home/gsajko/work/custom_twitter_feed/auth/auth.json --since
```

Simon also shared his crontab for his personal data warehouse here:
https://gist.github.com/simonw/1299d61d17637d1145955ebc019ea3c4.

I shamelessly copied his "time" settings. 

What is the `run-one`? It checks if the previous run was completed, before starting a new one. Given possible timeouts on API side, it's great to have it in place.
https://blog.dustinkirkland.com/2011/02/introducing-run-one-and-run-this-one.html

### Note about collecting data
I'm using `home-timeline` command. Here is info from Twitter API docs:

[Get Tweet timelines](https://developer.twitter.com/en/docs/twitter-api/v1/tweets/timelines/api-reference/get-statuses-home_timeline)

> Up to 800 Tweets are obtainable on the home timeline. It is more volatile for users that follow many users or follow users who Tweet frequently.

If you have muted accounts, using this endpoint WON'T get their tweets. I some way, I'm giving some of the control to twitter when using this option.
But I wanted to have a wide range of tweets, and another option would be to download ALL tweets by ALL users I follow.

This would balloon the database waaay more than I would like to - this option would download 3200 tweets of each user. But it would download another 3200 with next use - until it would grab all possible tweets.

I did this as a test for a list of only 60 users, and `db` file was about 200mb in size.
Compared to `home-timeline` `db` file from grabbing tweets from more than 600 users I follow - 160mb (about 2,5 months of cronjobs).

As I mentioned earlier - script downloaded 3200 each time per user - going back in time to 2008 for some users.
It was fun looking at tweets posted in 2008, though. People were using Twitter in a totally different way than today - there were mostly micro-bloging ("I ate lemon pie today", "watching the game" - that kind of stuff).

## Filtering out

### Finding news
As a part of the design, my goal was to remove news from my feed.
**The first "filter" should remove tweets related to the news.**

After pondering for some time ("Should I do ML-based news detection model?"), I arrived at a very simple solution:
- removing tweets containing links to news domains

All I needed, is a collection of news domains.
Luckily for me, there is such a dataset: [News Aggregator Dataset](https://www.kaggle.com/uciml/news-aggregator-dataset)
It's a collection of headlines - over 400k news stories.

In that dataset, there is a column named `HOSTNAME`, so I could take unique values from that column and ignore the rest of the dataset.

This got me over 9000 unique news domain names.


#### Get the domain from the tweets
so let's move back to tweets.

In order to find news in tweet's urls I needed to clean them up.
- remove links to other tweets (they are urls - but not )
- remove short links, like "bit.ly", "buff.ly". *(in hindsight - I'll leave them for extracting domains)*

Then I needed a function to find url in tweet (simple regex `re.findall(r"http\\S+", tweet)`)

to get the domain I used  `urllib` library:

After reviewing some batches, I noticed that some domains weren't detected properly, so there was a need for some splitting and joining - results were transformed to match the `news_domain_list`


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

#### Is it news?
The result of this was a single pandas cell with a list.

Current record holder (with 5 links):
{{<tweet 1336857186350354432>}}

I expanded `pd.DataFrame`, so each domain has its own column.
On each of those columns I used simple `.isin`
```python
    for col in columns:
        df[col] = df[col].isin(news_domains_list)
```
![](/img/20210226180547.png)

Then I summed it up in one column `contains_news` and made values `0` or `1`.

![](/img/20210226181734.png)

Through uploading batches of tweets to the collection and reviewing them, I make some changes to that list: 
- removed some domains (youtube.com, linkedin.com), 
- noticed that some of the domains weren't in the dataset (for example techmeme.com, buzzfeednews.com), so I added them to the list.

On top of that - I saw that often quote tweets were containing news, or someone was replying to the news posted by another person.
To deal with that, I decided to add  tweet + quoted_tweet + in_reply_to and use `find_news` function on that text.

### Muted words
### Muted accounts
