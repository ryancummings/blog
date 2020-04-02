+++
title = "Reddit marketplace notifications withg Python and PRAW"
author = ["Ryan Cummings"]
date = 2020-03-31T17:51:00-04:00
lastmod = 2020-04-02T09:20:04-04:00
tags = ["python"]
categories = ["technical"]
draft = false
+++

A few weeks ago, BestBuy mistakenly listed a high quality 2tb SSD for more than $200 under retail price. The mistake only lasted a few minutes, but the lucky few who saw it in time got an amazing deal. I found out about the deal through [the BuildAPcSales subreddit](https://www.reddit.com/r/buildapcsales/). But I have better things to do than prowl reddit for deals on tech. My secret weapon? A python script that searches Reddit for things I want to buy, and notifies me via phone if a deal pops up.

In this post, I'll describe my Reddit market watcher script and help you build your own. Let's getT started!


## First steps - setting up the environment {#first-steps-setting-up-the-environment}

It's always a good idea to write a Python program using a virtual environment. To do so, make a new folder where your script will live and execute the following from your shell.

```bash
python3 -m venv reddit_venv ./reddit_venv
```

Now that you have a virtual environment sitting inside your project folder, activate the venv using the command:

```bash
source ./reddit_venv/bin/activate
```

Depending on your shell, you may find that the name of the virtual environment is now listed in the shell prompt. Now that you are using your virtual environment, you need to run the `praw` python library to use the Reddit API:

```bash
pip install praw
```

The `praw` library documentation is available online [here](https://praw.readthedocs.io/en/latest/). This library has a lot to offer outside the scope of this tutorial, so check it out!


## Reddit setup {#reddit-setup}

To use the `praw` library, you need to register a Reddit API app. Praw offers [quick start instructions](https://praw.readthedocs.io/en/latest/getting%5Fstarted/quick%5Fstart.html) to help you out. [Go here](https://github.com/reddit-archive/reddit/wiki/OAuth2-Quick-Start-Example#first-steps) for instructions from Reddit on making and registering a Reddit app. You will need the following to make full use of the Reddit API:

-   client id
-   client secret key
-   user agent (app name)
-   username (your reddit username)
-   password (your reddit password)


## Database setup {#database-setup}

Once you have the login information for your reddit app, you are ready to start coding! First we will import our libraries:

```python
import praw
import datetime as dt
import requests
import json

import sqlite3
import os
```

Next, let's create a database to store Reddit posts that we download with the API:

```python
if not os.path.exists('/home/pi/python/reddit.db'):
    conn = sqlite3.connect('/home/pi/python/reddit.db')
    c = conn.cursor()
    with conn:
        c.execute("""CREATE TABLE reddit (
        id text,
        subreddit text,
        title text,
        timestamp text,
        url text
        )""")
else:
    conn = sqlite3.connect('/home/pi/python/reddit.db')
    c = conn.cursor()
```

You will have to customize the paths to point to the project folder you created earlier.

Let's add a few functions that let us work with the database:

```python
def addEntry(post): # {id, subreddit, title, timestamp, url}
    with conn:
        c.execute("INSERT INTO reddit VALUES (:id, :subreddit, :title, :timestamp, :url)",
                  post)

def isNew(post):
    c.execute("SELECT * FROM reddit WHERE id=:id", {'id': post['id']})
    if c.fetchone():
        return(False)
    else:
        return(True)
```

`addEntry` creates a new entry from a post dictionary that `praw` will give to us later. `isNew` will check if a given post dictionary is already in the database based on a Reddit post's id.


## More API setup {#more-api-setup}

Next, let's create a praw instance to pull posts from Reddit.

```python
reddit = praw.Reddit(client_id='your_id',
                     client_secret='your_secret_key',
                     user_agent='your_user_agent',
                     username='your_username',
                     password='your_password')
```

Great. One more custom function will let us use the [PushBullet](https://www.pushbullet.com/) service to push notifications to our devices. You'll need a PushBullet token to use the service, so if you don't already have one, go ahead and sign up for one. You can read more in the [PushBullet Docs](https://docs.pushbullet.com/).

```python
def pushbullet_link(title, body, url):
    msg = {"type": "note", "title": title, "body": body, "url": url }
    TOKEN = 'o.rNItGF4sNrVuKxqpPXcGs9f0G1Ro3j7r'
    resp = requests.post('https://api.pushbullet.com/v2/pushes',
                         data=json.dumps(msg),
                         headers={'Authorization': 'Bearer ' + TOKEN,
                                  'Content-Type': 'application/json'})
    if resp.status_code != 200:
        raise Exception('Error',resp.status_code)
    else:
        print('Message sent')
```


## The search function {#the-search-function}

With that setup out of the way, we just need to create a function to search reddit for the stuff we want to buy! The following function takes the following arguments:

-   Subreddit: The subreddit you want to search
-   mandatorySearchTerms: Exclude all results that lack these terms in the title
-   optionalSearchTerms: If a result contains _any_ of these terms, notify the user.

<!--listend-->

```python
def searchReddit(subreddit, mandatorySearchTerms, optionalSearchTerms):
    newPosts = []
    subreddit_instance = reddit.subreddit(subreddit)

    for submission in subreddit_instance.new(limit=100):
        flag = False
        for term in optionalSearchTerms:
            if term.lower() in submission.title.lower():
                flag = True
        for term in mandatorySearchTerms:
            if term.lower() not in submission.title.lower():
                flag = False

        if flag == True:
            newPosts.append({
                "id": submission.id,
                "subreddit": subreddit,
                "title": submission.title,
                "timestamp": dt.datetime.fromtimestamp(submission.created).strftime("%b %d %H:%M:%S"),
                "url": submission.url
            })

    for post in newPosts:
        if isNew(post):
            addEntry(post)
            pushbullet_link("New post found!", post['title'], post['url'])
```

The function is relatively simple:

1.  It asks the subreddit for the 100 most recent posts
2.  For each recent post, it checks to see if it includes all `mandatorySearchTerms` and at least one of the `optionalSearchTerms`. If it does, it sets a flag to mark the post.
3.  Flagged posts are converted to dictionaries containing key information including id, subreddit, title, timestamp, and url. A list of these dictionaries is generated as the program scans all returned posts.
4.  The list of flagged posts is cross-referenced with the database. For each post, if it is not in the database, it is added and the user is notified via PushBullet.


## A few sample searches {#a-few-sample-searches}

All that remains is using the search function to run a few searches. The following searches the BuildAPcSales subreddit for SSD hard drives:

```python
searchReddit(subreddit='buildapcsales',
             mandatorySearchTerms=['[SSD]'],
             optionalSearchTerms=['ssd'])
```

And this one searches the Pen\_Swap subreddit for my favorite kind of fountain pen, the Vanishing Point:

```python
searchReddit(subreddit='pen_swap',
             mandatorySearchTerms=['[WTS]'],
             optionalSearchTerms=['vp', 'capless', 'vanishing'])
```


## Setting up a Cron job {#setting-up-a-cron-job}

You can manually run the script to see if your searches are working, but someday you will want the script to run automatically. I have a spare Raspberry Pi running my script every two minutes using `cron`, a tool that lets you execute scripts on unix machines. To set this up, you have to first make a bash scrip that points to your Pyhon script, like this:

```bash
#!/bin/bash
source /home/pi/python/venv_python/bin/activate
python /home/pi/python/reddit_scraper.py
echo $(date) "Ran Cron job" >> /home/pi/logs/reddit_scraper.log
```

The script should activate your virtual environment, then run the python script. This script also generates a simple log file so I can make sure it's running according to schedule.

Put the script somewhere your bash environment can see it. Follow [these instructions](https://mycyberuniverse.com/create-personal-bin-directory-run-scripts-without-specifying-full-path.html) if you need help setting up a personal `/home/usr/bin` directory for the script. Make sure the script is executable e.g. `chmod +x path/to/your/bash_script`.

Next, just set up a cron job using the command `crontab -e`. If you need help with setting it up, check out this link: <https://linuxize.com/post/scheduling-cron-jobs-with-crontab/>.

All that's left to do is let it run for a while, check your logs, and jump on those sweet deals! Enjoy!
