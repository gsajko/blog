---
title: "Project update"
date: 2021-03-14T13:38:23+01:00
draft: true
_build:
  list: false
  render: true
  publishResources: true
tags: []
description: ""
toc: true
---

# Meta - notes

It's hard for me to do blogpost about project, while developing it. So I fallen behind with "reporting".

In hindsight, it's better (for me) to focus on working on project and then blog about my progress in a monthly cadence. But then again, it's hard to write in retrospect, if the time gap is too big.

Once the project is more established, I can do weekly blog about iterative improvements.

# What I did 
I had many eye opening moments since last post.
I won't write much about "how to", but rather "what" and "why".
## CLI
CLI stands for **C**ommand **L**ine **I**nterface.
I think it was the biggest improvement for me. CLI at first felt like magic ✨.

My thinking about running code until then was very Jupyter Notebook centric. 

You stay in notebook interface, you run cell after cell.

With CLI I started run code out of my terminal. I can't describe how gratifing it was. After some time developing, I finaly felt that I created **something**. 

Here are option used by me currently:
```py
  age: int = typer.Option(21, "--age", "-a"),
  reverse_age: bool = typer.Option(False, "--reverse", "-r"),
  nr_tweets: int = typer.Option(30, "--tweets", "-t"),
  ignore_lists: bool = typer.Option(False, "--ignore_lists", "-il"),
  users_from_list: str = typer.Option(None, "--users_from_list", "-fl"),
  friends: bool = typer.Option(False, "--friends_only", "-fo"),
  notfriends: bool = typer.Option(False, "--not_friends_only", "-nfo"),
  min_likes: int = typer.Option(0, "--min_likes", "-l"),
```

I used it to grab tweets from my timeline, that are stored in SQL.

`tweetfeed to-collection -a 60 -r -fl Q1 -l 10` - this returns tweets that are "younger" than 60 days (`-a 60 -r`), only from users that belong to private list `Q1` that I created (`-fl Q1`).
To avoid noise, I want to see only tweets that have more than 10 likes `-l 10`.

Those option are ones that I use the most - I change only values - how old tweets to get, from which list, and should I filter them out by number of likes.




## Linting + styling
As a self-learner it's hard to go on, if I don't get feedback. 


## Testing code
testing_code


## Model
Finally! After that much coding at last I could do some Machine Learning.

I used very simple linear model `LogisticRegression`. It was (stil is) very basic. It's by design - I want to iterate and learn/drill in each steps of ML process.

I only cleaned texts.
To get labels I used my Twitter activity.

```py
def create_neg_list_idx(path_to_db, owner_id, auth, muted_path):
    df_tweets = load_tweets(path_to_db, days=0)
    muted_acc_list = get_muted_acc(
        owner_id, auth, muted_lists=["nytblock", "muted"]
    )
    neg_list_idx = list(
        set(
            idx_contain_muted_words(df_tweets, muted_path)
            + from_muted_users_idx(df_tweets, muted_acc_list)
            + with_news_idx(df_tweets, muted_path)
            + get_not_rel_idx(owner_id, auth)
        )
    )
    return neg_list_idx

# create "negative" tweets list (tweets I don't want to see):
# - tweets that contain words I buted
# - tweets from users I muted
# - tweets that contain links to news
# - tweets that I labeled as not relevant
```

```py
def get_engagement(path_to_fav, path_to_timeline):
    favorite_idx = load_favorites(path_to_fav).tweet.tolist()
    df_timeline = load_tweets(path_to_timeline, days=0)
    quoted = df_timeline[df_timeline.quoted_status == "N/A"].id.tolist()
    retweeted = df_timeline[df_timeline.retweeted_status == "N/A"].id.tolist()
    return list(set(quoted + retweeted + favorite_idx))

# create "positive" tweets list (tweets I want to see more, based on my twitter interactions):
# - tweets that I liked
# - tweets that I quote tweeted
# - tweets that I retweeted
```

After creating dataset, I split the data and used `CountVectorizer` to transform "text" into "numbers".

Based on my list `LogisticRegression` tried to classify, If tweet belong to one ("negative") or the other ("positive") list. So by defalut it predicts `0` or `1`. 
But if we use `model.predict_proba(X)` it will return a number between 0 and 1. This number will show, how confident the model was at predicting class.

For example `0.95` whould mean, that model is fairly confident, that this tweet would be interesting to me.

On the other hand, tweet with score `0.51` also would be classified as interesting, but with much lower confidance.

I run the prediction on all tweets in my database - it scored all existing tweets.

So now, when I run CLI to grab tweets, they will be sorted by the score - first, those with highest score will be returned.

**I framed the ranking problem as a binary classification problem.**
### CLI + model scores
Let's go back to some CLI options.

```py
  age: int = typer.Option(21, "--age", "-a"),
  reverse_age: bool = typer.Option(False, "--reverse", "-r"),
  nr_tweets: int = typer.Option(30, "--tweets", "-t"),
  ignore_lists: bool = typer.Option(False, "--ignore_lists", "-il"),
  users_from_list: str = typer.Option(None, "--users_from_list", "-fl"),
  friends: bool = typer.Option(False, "--friends_only", "-fo"),
  notfriends: bool = typer.Option(False, "--not_friends_only", "-nfo"),
  min_likes: int = typer.Option(0, "--min_likes", "-l"),
```

Knowing, that the tweets will be sorted by being interesting for me, I can try and discover content, that otherwise I would be ignoring:

`tweetfeed to-collection -a 30 -nfo` will return tweets that are older than 30 days (`-a 30`): so that I'll avoid recent tweets, and option `-nfo` will ensure that those tweets are only from people I don't follow.

For example I interact and like content of users, that belong to `Q1` list.
I can specify, to ignore those users, to enable more content outside of my usual content-sources: `tweetfeed to-collection -a 30 -il Q1`